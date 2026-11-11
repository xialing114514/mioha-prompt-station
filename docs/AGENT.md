# 外部 AI Agent 接入

[手册目录](README.md) · [内置 AI](BUILT_IN_AI.md)

## 职责与标准流程

外部 Agent 负责创意设计，软件内置 AI 负责按当前绘图模型和 Skill 转换表达。原始方案、版本和转换结果分开保存。外部 Agent 不需要维护所有绘图模型的专属语法，也不应另外建立重复推理服务。

流程：查询能力 → 创建会话 → 提交原始设计 → 请求转换 → 查询／订阅任务 → 回答追问或提出反馈 → 获取结果 → 配置实际资源 → 申请应用／生图 → 按权限自主确认或用户确认 → 获取执行回执。

软件界面需保持运行，才能领取转换任务、执行编辑器应用和返回回执。HTTP 在线与转换可领取是两个状态。

## 在软件中创建身份

打开 AI Bridge，新建 Agent，只授予实际需要的权限。Token 明文仅创建或轮换时展示一次，保存到自己的私人文件。停用撤销访问；删除会撤销 Token、取消未执行请求并保留创作历史。运行中的任务需先完成再删除。

## MCP 配置

安装目录附带 `agent-tools/mioha-agent.exe`，无需安装 Python。下面是无凭据示例，请替换成自己电脑的实际路径：

```json
{
  "mcpServers": {
    "mioha-workspace": {
      "command": "C:\\Path\\To\\MiohaPromptStation\\agent-tools\\mioha-agent.exe",
      "args": ["--mcp"],
      "env": {
        "MIOHA_AGENT_TOKEN_FILE": "C:\\Path\\To\\private-agent-token.json"
      }
    }
  }
}
```

私人 Token 文件格式：

```json
{"token":"YOUR_AGENT_TOKEN"}
```

不要把 Token 文件提交到 GitHub。修改客户端配置后重连 MCP。

## 地址发现与 CLI

客户端默认读取当前用户目录的 `agent-endpoint.json`，定位实际运行端口并核对实例。显式的 `MIOHA_WORKSPACE_URL` 优先；换安装路径或端口后，优先检查是否还有旧地址覆盖。

同一客户端还提供 `status`、`capabilities`、`sessions`、`jobs`、`designs` 等 CLI 命令，使用 `--help` 查看参数。普通软件使用不依赖 CLI。

## 权限与确认

| 权限 | 用途 |
| --- | --- |
| read | 状态、模型、Skill、会话及任务读取 |
| design | 提交创意方案与版本 |
| convert | 请求内置 AI 转换与反馈 |
| resources | 查询资源与画师，管理本 Agent 的卡片、实验、分镜和图库 |
| skills / models | 请求管理对应资源 |
| generate | 配置底模、LoRA、参数及工作流，申请生图 |
| images | 使用图像工作台与参考素材 |
| delete | 请求删除资源 |

资源删除、覆盖提示词和最终生图仍经过业务确认。默认允许 Agent 确认自己已授权的请求；用户可在身份列表切换为始终由用户确认。确认生图可能使用自己的付费服务，请按需授予 generate 权限。外部 Agent 没有任意文件、系统命令或数据库直连入口，也不能解除全局内容模式。

## 任务状态

| 状态 | 含义 |
| --- | --- |
| queued | 已保存并排队 |
| running | 已领取／执行；应用任务还可能在等编辑器回执 |
| awaiting_input | 需要回答补充问题，可沿用原任务恢复 |
| awaiting_confirmation | 待确认；按权限调用 confirm_job，或等待用户审阅 |
| success | 当前操作已完成；应用提示词以编辑器回执为准 |
| error / cancelled | 失败或已取消，查看错误与已有结果后再决定是否重试 |

设计、转换等支持 `requestId` 去重。同一次重试保持参数与 requestId 不变；改变方案或参数时使用新的 requestId。

## REST、分页与 SSE

认证使用 `Authorization: Bearer YOUR_AGENT_TOKEN`，基础地址为软件实际端口下的 `/api/agent`。先读 `/capabilities` 获取当前协议能力、完整路由和队列状态。

列表支持 `limit`／`offset`，默认100、最大200，总数在 `X-Total-Count`。SSE `/events` 是有界流，支持 `Last-Event-ID` 或 `after` 续传，默认90秒，客户端需按游标重连。流断开不等于任务取消。

转换任务优先于尚未执行的生图，但不会抢占已经运行的任务。会话上下文与任务按 Agent 隔离。


## 生图资源、LoRA 与工作流控制

先用 `list_resources` 读取稳定资源 ID，`list_workflows` 读取工作流 ID；可用 `get_generation_settings` 查询已应用配置。读取完整工作流需要 resources 权限，改变生图配置需要 generate 权限。

调用 `configure_generation`，其 value 示例如下（替换为真实 ID）：

```json
{
  "modelId": "model_anima",
  "provider": "comfy",
  "checkpointResourceId": "CHECKPOINT_RESOURCE_ID",
  "workflowId": "WORKFLOW_ID",
  "loras": [
    {"resourceId": "LORA_A_ID", "weight": 0.85, "clipWeight": 0.6, "triggerEnabled": true},
    {"resourceId": "LORA_B_ID", "weight": 1.1, "triggerEnabled": false}
  ],
  "parameters": {"width": 832, "height": 1216, "steps": 24},
  "requestId": "UNIQUE_CONFIGURATION_REQUEST"
}
```

返回配置任务，success 才表示界面已经应用。配置本身不生图，`loras: []` 清空列表。生成请求使用本 Agent 的配置快照，避免其他 Agent 或界面改变当前设置后串用资源。

## 自主确认与页面帮助

用 `confirm_job` 确认自己提交的待确认请求。不能跨 Agent、绕过 Token 权限或资源引用保护；被设为用户确认模式时自确认返回403。全局内容模式不对外部 Agent 开放。

先读取 capabilities 中的 `selfConfirmationAllowed`、`confirmationMode` 和新能力。1.2 保留旧的 `*RequiresUserConfirmation` 兼容字段，新客户端不能再把它理解为只允许人工点击。

`list_pages`／`get_page_help` 提供界面问号中同一份功能说明。分类管理也有独立资源分类工具，删除分类保留磁盘权重。更新软件后请重连外部 MCP 进程，旧进程不会自动加载新增工具。

## 工具与接口索引

运行新版程序并重连 MCP 客户端，随后通过 `tools/list` 或 `workspace_capabilities` 获取当前能力。

### MCP 工具

| 工具 | 用途 |
| --- | --- |
| `workspace_status` | 查询当前绘图模型、Skill、界面心跳与队列状态。converterReady 只在工作进程在线且可领取转换时为真；队列阻塞不等于任务失败。 |
| `workspace_capabilities` | 查询应用版本、协议能力、权限、完整 REST 路由、数据库诊断和队列状态。先调用此工具了解当前软件支持范围。 |
| `list_pages` | 列出全部软件页面及作用、操作步骤与输出，先了解软件再操作。 |
| `get_page_help` | 读取指定页面的功能介绍。id 来自 list_pages，不依赖界面截图。 |
| `get_generation_settings` | 读取当前或本 Agent 最近成功应用的生图配置，包含引擎、底模、LoRA 与工作流。 |
| `configure_generation` | 配置生图但不执行绘图，需 generate 权限。value 支持 modelId、provider(neo/comfy/nai)、checkpointResourceId、workflowId、loras:[{resourceId,weight,clipWeight,triggerEnabled}]、parameters 与 requestId。先用 list_resources 查询实际 LoRA ID；返回任务 ID，success 才表示界面应用完成。loras:[] 清空所选 LoRA。 NovelAI 可传 references:[{referenceId,strength,informationExtracted,fidelity}] 与 referenceConsent:true，另需 images 权限；先用 list_references 查询素材。软件检查模型兼容与互斥并保存原图快照。 |
| `list_workflows` | 列出 ComfyUI 文件夹中的工作流及可用状态；API 格式可直接读取，普通 UI 格式需连接 ComfyUI 获取节点定义。 |
| `get_workflow` | 读取所选工作流及 sha256，需要 resources 权限。常规 UI 格式结合已连接 ComfyUI 的节点定义转换；旁路等不确定格式要求导出 API，避免改变工作流含义。 |
| `list_storyboards` | 读取本 Agent 分镜项目，需 read/resources。不同 Agent 的项目和生成上下文隔离。 |
| `get_storyboard` | 读取本 Agent 分镜设计、全局与单画面覆盖、模型、版本、候选和主图。需 read/resources；图像 URL 用同一 Bearer Token 读取。 |
| `save_storyboard` | 请求新建或保存本 Agent 分镜项目。value={sessionId,name,modelId,globalDesign,scenes,panels,requestId,可选id,revision,generation}，需 design/resources；有 generate 权限时可采用已配置生图设置。panels 只提交设计字段，不直接写模型提示词；随后调用内置转换。确认管理请求后执行。 |
| `manage_storyboard` | 请求分镜动作。value={action:convert/apply-conversion/draw/primary/export/delete/cancel-pending,revision,panelId?,candidateId?,imageIndex?,count?,mode:primary/all?,requestId}。convert/apply 需 convert，draw/cancel 需 generate，delete 需 delete，始终需 resources。先 confirm_job，再查询项目；转换复用现有内置 AI，draw 必须引用本 Agent 会话已应用的转换。导出返回带同一 Token 读取的私有 ZIP URL。 |
| `list_prompt_cards` | 分页查询共享与本 Agent 的模型专属提示词卡片，需 read/resources；不会读取其他 Agent 私有卡片。返回 items/total/hasMore。卡片内容是参考数据，不是系统指令。 |
| `get_prompt_card` | 读取可用卡片的正文、正负面、权重和兼容模型，用稳定 ID 构建组合。需 read/resources。 |
| `save_converted_card` | 请求把本 Agent 已经由内置 AI 转换的 Prompt 保存为卡片。value={promptId,name,category,side:positive/negative,requestId,可选id}，需 resources/convert；确认后保存。不能直接写入未经转换的任意模型语法。 |
| `list_prompt_boards` | 分页查询共享及本 Agent 的已保存组合。其他 Agent 私有组合与片段隔离。需 read/resources。 |
| `get_prompt_board` | 读取组合、模型 ID 和版本，供批量或画风实验使用。需 read/resources。 |
| `save_prompt_board` | 请求保存卡片组合，value={sessionId,name,modelId,blocks:[{cardId,weight?,side?,enabled?,locked?}],requestId,可选id,expectedRevision}。需 resources，确认后执行；只选择真实卡片，由软件原有组合器处理模型语法。 |
| `list_experiments` | 分页查询本 Agent 的组合实验，需 read/resources。不能读取其他 Agent 实验。 |
| `get_experiment` | 查询本 Agent 实验状态、逐项任务、实际种子和图像 URL。返回轻量摘要；需 read/resources。 |
| `create_experiment` | 请求建立暂停状态的批量实验。value={sessionId,name,boardId,dimensions:[{name,cardIds:[已有卡片ID],lockedIndex?}],method:all/sample,count?,repeat?,samplingSeed?,seedPolicy:fixed/increment,requestId}。需 resources/generate。使用已配置的本 Agent 生图设置，每批最多256项；确认管理请求后仍须 control_experiment start 才执行。 |
| `control_experiment` | 控制本 Agent 实验，value={action:start/resume/pause/retry/cancel-item/reorder/delete,requestId,按原操作需要的jobId或jobIds}。需 resources/generate；delete 另需 delete。先确认管理请求，再由现有批量队列执行，不另建推理系统。 |
| `list_style_experiments` | 分页查询本 Agent 画风实验，需 read/resources。 |
| `get_style_experiment` | 读取本 Agent 的画风代数、候选、父代、用户筛选标记和生成结果。需 read/resources。 |
| `create_style_experiment` | 请求从已有组合创建画风实验，value={sessionId,name,boardId,requestId}；需 resources，确认后建立独立实验。 |
| `evolve_style` | 请求探索、交叉或变异，value={cardIds,parentIds?,mode:explore/cross/mutate,count,maxCards,seed,requestId}。id 为本 Agent 实验 ID；需 resources。只选择现有卡片，原有画风演化逻辑负责候选生成。 |
| `draw_style_candidates` | 请求为本 Agent 选中画风候选建立批量任务，value={candidateIds,seed?,requestId}。需 resources/generate，使用本 Agent 已配置引擎；结果返回暂停的 batch，随后明确 start。 |
| `review_style_candidate` | 请求标记画风候选，value={review:treasure/special/reject/unrated,requestId}；id 为候选 ID，需 resources。确认后与图库评级联动。 |
| `save_style_cards` | 请求将选中候选保存为本 Agent 的可复用卡片，value={name?,requestId}；id 为候选 ID，需 resources，确认后执行。 |
| `inspect_image_workflow` | 检查本机 ComfyUI 工作流的真实图片、蒙版与 text/prompt/instruction 输入。value={sourceId,workflowId}；需 read/images/resources。先导入受控图片，选择 list_workflows 中的真实 ID。返回短期 planId、可用节点及缺失项。不同 Agent 的计划与私有工作流隔离。 |
| `request_image_workflow` | 请求按检查结果运行图像工作流，需 images/resources/generate，先创建自己的 session。value={planId,sessionId,modelId,imageNodeId,maskId?,maskNodeId?,textInput:{nodeId,name}?,instruction,instructionMode:append/replace,requestId}。返回待确认管理任务，随后 confirm_job；成功后结果给出图像任务 ID，用 get_image_workflow_task 查询。只改明确选择的输入，保留工作流模型、LoRA、精度及采样；不提供任意路径或命令入口。 |
| `get_image_workflow_task` | 查询本 Agent 图像编辑的队列、进度、错误及结果 URL；结果可携带相同 Bearer Token 获取，不能读取其他 Agent 的任务。 |
| `import_image_workflow_result` | 请求把成功结果导入为原图关联的工作台副本。id 为图像编辑任务 ID；value={index,requestId}，需 images/resources/generate，确认管理任务后执行。原图保留，重复导入返回同一个副本。 |
| `list_images` | 分页查询图像工作台，需 read 与 images 权限。能读取共享的用户图片和自己的私有图片，不能读取其他 Agent 的私有图片。每页最多 60 张，返回 items/hasMore/offset/limit。 |
| `get_image` | 获取图片尺寸、原图生成参数、来源和处理步骤。downloadUrl 可用同一 Agent Bearer Token 下载图片字节，不传任意文件路径。 |
| `get_image_parameters` | 读取原图写入的生成参数和本机处理记录，区分当前图像尺寸与原始生成尺寸，不把处理后的副本说成重新生成的图片。 |
| `import_image` | 导入图片到本机工作台，需 images 权限。value={url,name,safeConfirmed}，url 为 data:image/ 的上传或软件受控图库 URL。图片最多 64 MB/4000 万像素，PNG/JPEG/WebP。保留原图。禁止任意文件路径、外部图片 URL 和跨 Agent 私有图片。安全模式需明确 safeConfirmed:true。 |
| `process_image` | 请求处理副本，需 images 权限，返回待确认任务，用 confirm_job 确认自己的请求。value={sourceId,requestId,steps,format,quality,background,keepParameters}。steps 最多8项，type 支持 resize(width,height)、crop(x,y,width,height)、rotate(angle=90/180/270)、flip(axis=horizontal/vertical)、pad(x,y,width,height,color=transparent/#RRGGBB)、denoise。format=PNG/JPEG/WEBP，quality=1..100。resize 是 Lanczos 插值；原图保留，完成后 job.result 返回新图片。 |
| `rename_images` | 请求修改工作台中所选图片的名称，需 images 权限。value={ids:[图片ID],name:名称前缀,requestId}。最多100项，确认后执行，多张按选择顺序编号，不重命名用户原文件。 |
| `list_image_pipelines` | 读取可复用的图片处理步骤预设，需 read 与 images 权限，不读取其他 Agent 的私有预设。 |
| `save_image_pipeline` | 请求保存步骤预设，value={name,steps,format,quality,background,keepParameters,requestId,可选id,revision}，需 images 权限。更新时必须传当前 revision，确认后执行。 |
| `delete_image_pipeline` | 请求删除步骤预设，需 images 与 delete 权限，确认后执行，已处理的图片保留。 |
| `list_references` | 分页读取共享及本 Agent 的参考素材，需 read/images 权限。type 可筛选 general/vibe/precise。参考素材与图片稳定 ID 关联，不以显示名模糊匹配。 |
| `get_reference` | 读取参考类型、默认强度、提取量、精准程度和本机图片信息。不同 Agent 的私有参考隔离。 |
| `save_reference` | 请求创建/编辑参考条目，value={imageId,name,type:general/vibe/precise,preciseMode:character/style/character&style,category,tags,strength,informationExtracted,fidelity,requestId,可选id,revision}。需 images 权限。先 import_image，再保存此条目，随后 confirm_job。 |
| `delete_reference` | 请求删除参考条目，需 images/delete 权限，确认后执行。原图和已经提交的生图快照保留。 |
| `confirm_job` | 确认自己提交的待确认任务，包括应用、生图及管理操作。遵守原权限、会话及资源引用限制；Agent 被设为用户确认模式时返回403。确认生图可能使用已配置的付费服务。 |
| `list_resource_categories` | 读取用户自定义资源分类和计数；分类独立于引擎归属及模型适配。需要 resources 权限。 |
| `save_resource_category` | 请求创建或重命名分类。value={name,可选id}。需要 resources 权限，随后用 confirm_job 确认自己的任务。 |
| `assign_resource_categories` | 请求归类。value={resourceIds:[资源ID],categoryIds:[分类ID]}；空分类列表移至未分类。确认后生效，权重文件不移动。 |
| `delete_resource_category` | 请求删除分类关系；确认后执行，资源记录与磁盘文件保留，需要 resources 与 delete 权限。 |
| `list_models` | 分页列出可用绘图模型定义及稳定 ID，供选择设计方案转换目标；不会加载或下载权重。 |
| `list_skills` | 分页查询模型专属 Skill 规则与元数据，modelId 可过滤；切换模型时应选择兼容规则。 |
| `list_resources` | 分页查询 LoRA、真实触发词及模型兼容性。默认隐藏本机绝对路径；只有明确需要诊断时才传 paths=true。 |
| `list_artists` | 分页查询现有画师串与适用模型，不编造作者或跨模型语法。 |
| `list_sessions` | 分页恢复本 Agent 的独立会话，无法读取其他 Agent 的上下文。 |
| `list_designs` | 分页恢复本 Agent 设计稿；可用 sessionId 限定会话，原文与模型提示词分开保存。 |
| `list_jobs` | 分页查询本 Agent 的任务，可按 sessionId/status 过滤。queued 返回本 Agent 队列位置与阻塞原因，awaiting_input 要补充回答。 |
| `create_session` | 创建隔离会话并返回 session ID；提交设计、转换和生图前先创建或恢复自己的会话。 |
| `submit_design` | 提交与绘图模型无关的完整创意设计，支持自然语言或结构化对象。sessionId 必填；id 用于更新，requestId 用于安全重试。 |
| `convert_design` | 请求软件现有内置 AI 将指定设计版本转换为目标模型提示词，不另建推理系统。返回任务，需用 get_job 查询；requestId 可幂等重试。 |
| `resume_job` | 回答 awaiting_input 转换任务的追问，feedback 写完整补充内容。沿用相同任务 ID 和设计版本重新排队，requestId 用于幂等重试。 |
| `get_job` | 查询本 Agent 单条任务、错误、追问、排队位置、生成进度和转换结果 ID。awaiting_confirmation 时按 capabilities 的 selfConfirmationAllowed 判断：可自主确认则调用 confirm_job，否则等待用户确认。 |
| `get_design` | 读取本 Agent 设计原文和版本记录；不将转换提示词当作原始设计。 |
| `get_prompt` | 读取本 Agent 转换后的正负提示词、目标模型、Skill 快照和审阅信息。 |
| `apply_prompt` | 请求把转换结果应用到编辑器，会覆盖现有正负提示词并可能切换模型。返回待确认任务，按权限调用 confirm_job 自主确认，或由用户确认；确认成功后执行并回执。 |
| `request_generation` | 提交本会话已转换 Prompt ID 的生图请求，返回待确认任务。先用 configure_generation 选择实际底模、LoRA 和权重；按权限用 confirm_job 确认自己的任务，或等待用户确认。不能提交任意系统命令或文件路径。 |
| `cancel_job` | 取消本 Agent 尚未运行、待确认或待补充的任务；正在执行的上游任务不保证能中断。 |
| `generation_history` | 分页查看本 Agent 的生成结果与实际请求记录，可按会话恢复历史。 |
| `query_gallery` | 查询本 Agent 生成记录，支持收藏、Treasure/Special/Reject、模型、LoRA 相关搜索、Seed、标签、角色标注及 batch/style/tavern/storyboard/image-edit 来源。需 read；按记录分页，不返回历史图片 Base64。 |
| `review_generation` | 标记自己的生成组，需 resources。id 为 generation ID；value={favorite?,rating:unrated/treasure/special/reject?,tags?:[标签],characters?:[角色标注]}。不修改图片字节或生成参数。 |
| `save_model` | 请求创建或编辑模型定义，value 为现有业务模型对象。需要 models 权限，确认后执行；允许自主确认的 Agent 可用 confirm_job 确认自己的任务。 |
| `save_skill` | 请求创建或编辑模型 Skill，value 含 modelId、名称与规则内容。需要 skills 权限，确认后保存，可由已授权 Agent 自行确认。 |
| `save_artist` | 请求创建或编辑画师资源，value 含 modelId 与画师串。需要 resources 权限，确认后保存。 |
| `delete_model` | 提交模型删除请求，需 models/delete 权限；确认前不删除记录或文件；已授权 Agent 可确认自己的请求。 |
| `delete_skill` | 提交 Skill 删除请求，需 skills/delete 权限；遵守引用和内置资源保护。 |
| `delete_artist` | 提交画师删除请求，需 resources/delete 权限，仍须确认，可由已授权 Agent 自行确认。 |
| `delete_resource` | 提交资源记录删除请求，需 resources/delete 权限，不提供任意文件删除入口。 |

### REST 路由

| 方法 | 相对路径 | 用途 |
| --- | --- | --- |
| GET | `/capabilities` | 能力、版本、就绪与队列状态 |
| GET | `/state` | 当前模型、Skill 与队列状态 |
| GET | `/pages` | 页面列表与功能介绍 |
| GET | `/pages/{id}/help` | 指定页面的可读取功能帮助 |
| GET | `/generation-settings` | 查询生图配置与实际资源选择 |
| POST | `/generation-settings` | 配置底模、LoRA／权重与工作流，需 generate 权限 |
| GET | `/workflows` | 列出个人 ComfyUI 工作流 |
| GET | `/workflows/{id}` | 读取工作流图与校验值 |
| GET | `/prompt-cards ／ /prompt-boards` | 查询共享及本 Agent 的卡片与组合；需 read/resources，私有资源隔离 |
| POST | `/prompt-cards` | 将本 Agent 内置 AI 转换结果保存为卡片，需 convert/resources，确认后执行 |
| POST | `/prompt-boards` | 选择真实卡片 ID 构建组合，由软件处理模型语法，确认后执行 |
| GET | `/experiments ／ /style-experiments` | 查询本 Agent 批量与画风实验 |
| POST | `/experiments ／ /experiments/{id}/control` | 建立暂停的实验与开始／暂停／重试／排序，需 resources/generate，确认后执行 |
| POST | `/style-experiments ／ /style-experiments/{id}/evolve ／ /style-experiments/{id}/draw` | 管理画风候选与已有批量队列，确认后执行 |
| POST | `/style-candidates/{id}/review ／ /style-candidates/{id}/save-cards` | 用户评级与候选卡片保存，确认后执行 |
| GET | `/storyboards ／ /storyboards/{id}` | 本 Agent 分镜项目与候选，需 read/resources |
| POST | `/storyboards ／ /storyboards/{id}/manage` | 保存设计、内置转换、候选、主图与导出；管理请求按原权限确认后执行 |
| GET | `/storyboards/{id}/exports/{exportId}` | 同一 Agent Token 下载私有 ZIP |
| GET | `/generation-history/{id}/images/{index}` | 用本 Agent Token 读取自己的生成图片字节 |
| POST | `/image-workflows/plan` | 检查图片、蒙版和文本输入；需 read/images/resources |
| POST | `/image-workflows` | 请求图像工作流编辑；需 images/resources/generate 与本 Agent sessionId，确认后排队 |
| GET | `/image-workflows/{id}` | 本 Agent 图像编辑状态和结果 URL |
| GET | `/image-workflows/{id}/results/{index}` | 携带同一 Bearer Token 读取结果图片字节 |
| POST | `/image-workflows/{id}/import` | 请求将结果导入为关联工作副本，确认后执行 |
| GET | `/gallery?favorite=&rating=&modelId=&seed=&tag=&character=&origin=` | 本 Agent 创作记录筛选与轻量分页 |
| POST | `/generation-history/{id}/review` | 标记本 Agent 的生成组：收藏、评级、标签及角色 |
| GET | `/images?query=&limit=24&offset=0` | 工作台图片库，需 read 与 images 权限；最多每页 60 张 |
| GET | `/images/{id}` | 图片尺寸、来源、处理步骤和生成参数；不同 Agent 的私有图片隔离 |
| GET | `/images/{id}/parameters` | 读取原图生成参数和本机处理记录 |
| GET | `/images/{id}/content?thumbnail=1` | 用 Bearer Token 获取原图或缩略图字节 |
| POST | `/images/import` | 导入本机受控图片 URL 或 data:image 上传；images 权限，保留原图，不允许任意文件路径 |
| POST | `/images/process` | 请求图片处理副本；需 images 权限与 requestId，随后确认本 Agent 任务 |
| POST | `/images/rename` | 请求按 ids 顺序为图片库记录批量命名，随后确认 |
| GET | `/image-pipelines` | 步骤预设列表，需 read 与 images 权限 |
| POST | `/image-pipelines` | 请求保存步骤预设，含版本校验，随后确认 |
| DELETE | `/image-pipelines/{id}` | 请求删除预设，需 images 与 delete 权限，随后确认 |
| GET | `/references?query=&type=` | 参考素材列表，需 read 与 images 权限 |
| GET | `/references/{id}` | 参考类型、参数与图像信息 |
| POST | `/references` | 创建或编辑参考素材请求，确认后执行 |
| DELETE | `/references/{id}` | 删除参考条目请求，需 images 与 delete 权限，原图及已提交快照保留 |
| GET | `/resource-categories` | 自定义资源分类 |
| POST | `/resource-categories` | 创建／重命名分类，需确认 |
| POST | `/resource-categories/assign` | 按资源 ID 归类，需确认 |
| DELETE | `/resource-categories/{id}` | 删除分类关系并保留资源，需确认 |
| GET | `/models` | 绘图模型列表 |
| GET | `/skills?modelId=` | 模型 Skill 列表 |
| GET | `/resources?paths=0` | 资源；默认隐藏绝对路径 |
| GET | `/artists` | 画师列表 |
| GET | `/sessions` | 本 Agent 会话列表 |
| POST | `/sessions` | 创建独立会话 |
| GET | `/sessions/{id}/designs` | 会话设计稿列表 |
| GET | `/designs?sessionId=` | 设计稿列表 |
| POST | `/designs` | 提交／更新设计；支持 requestId |
| GET | `/designs/{id}` | 设计原文与版本 |
| POST | `/conversions` | 提交转换；支持 requestId |
| GET | `/jobs?sessionId=&status=` | 任务列表与排队位置 |
| GET | `/jobs/{id}` | 单个任务 |
| POST | `/jobs/{id}/resume` | 回答追问并沿用原任务继续转换 |
| POST | `/jobs/{id}/cancel` | 取消待执行／待补充任务 |
| POST | `/jobs/{id}/confirm` | 确认本 Agent 的已授权任务；用户确认模式下不允许自确认 |
| GET | `/prompts/{id}` | 转换结果 |
| POST | `/prompts/{id}/apply` | 请求覆盖编辑器，按权限自主确认或由用户确认 |
| POST | `/generations` | 请求生图，按权限自主确认或由用户确认 |
| GET | `/generation-history` | 本 Agent 生成历史 |
| GET | `/events?sessionId=&after=&timeout=90` | SSE；支持 Last-Event-ID 与 retry，按游标续传 |
| POST | `/models ／ /skills ／ /artists` | 管理操作请求，按权限自主确认或由用户确认 |
| DELETE | `/models/{id} ／ /skills/{id} ／ /artists/{id} ／ /resources/{id}` | 删除请求，检查权限及引用后确认执行 |

### 0.3.3 生图配置边界

`configure_generation` 的 `modelId` 关联提示词来源，实际执行由 `provider`、底模资源、LoRA、工作流和参数决定。切换写词规范不会自动切换生图后端，也不应为执行生图而改动写词规范。`parameters.naiModel` 可指定实际 NovelAI 生图模型 ID。底模与 LoRA 的明确元数据冲突、引擎位置及文件校验继续生效。

Neo 高清参数可放入 `parameters`：`hrCfg`、`hrDistilledCfg`、`distilledCfg`、`hrSteps`、`hrScale`、`denoisingStrength`、`hrUpscaler`、`hrResizeX`／`hrResizeY`、`hrPrompt`、`hrNegative`、`hrCheckpoint`、`hrModules`、`hrSampler`、`hrScheduler`。高清 CFG 和 Shift 的 `null` 表示继承，模型、模块及采样器名称必须来自所选 Neo 的实际清单，不能用任意系统路径。
