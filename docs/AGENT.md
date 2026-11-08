# 外部 AI Agent 接入

[手册目录](README.md) · [内置 AI](BUILT_IN_AI.md)

## 职责与标准流程

外部 Agent 负责创意设计，软件内置 AI 负责按当前绘图模型和 Skill 转换表达。原始方案、版本和转换结果分开保存。外部 Agent 不需要维护所有绘图模型的专属语法，也不应另外建立重复推理服务。

流程：查询能力 → 创建会话 → 提交原始设计 → 请求转换 → 查询／订阅任务 → 回答追问或提出反馈 → 获取结果 → 申请应用到编辑器 → 用户确认 → 按权限申请生图。

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
| resources | 查询资源与画师信息 |
| skills / models | 请求管理对应资源 |
| generate | 请求生图 |
| delete | 请求删除资源 |

资源删除、覆盖提示词和最终生图按业务要求由用户确认。外部 Agent 没有任意文件、系统命令或数据库直连入口，也不能解除全局内容模式。

## 任务状态

| 状态 | 含义 |
| --- | --- |
| queued | 已保存并排队 |
| running | 已领取／执行；应用任务还可能在等编辑器回执 |
| awaiting_input | 需要回答补充问题，可沿用原任务恢复 |
| awaiting_confirmation | 等待用户审阅 |
| success | 当前操作已完成；应用提示词以编辑器回执为准 |
| error / cancelled | 失败或已取消，查看错误与已有结果后再决定是否重试 |

设计、转换等支持 `requestId` 去重。同一次重试保持参数与 requestId 不变；改变方案或参数时使用新的 requestId。

## REST、分页与 SSE

认证使用 `Authorization: Bearer YOUR_AGENT_TOKEN`，基础地址为软件实际端口下的 `/api/agent`。先读 `/capabilities` 获取当前协议能力、完整路由和队列状态。

列表支持 `limit`／`offset`，默认100、最大200，总数在 `X-Total-Count`。SSE `/events` 是有界流，支持 `Last-Event-ID` 或 `after` 续传，默认90秒，客户端需按游标重连。流断开不等于任务取消。

转换任务优先于尚未执行的生图，但不会抢占已经运行的任务。会话上下文与任务按 Agent 隔离。


## 工具与接口索引

### MCP 工具（27项）

| 工具 | 用途 |
| --- | --- |
| `workspace_status` | 查询当前绘图模型、Skill、界面心跳与队列状态。converterReady 只在工作进程在线且可领取转换时为真；队列阻塞不等于任务失败。 |
| `workspace_capabilities` | 查询应用版本、协议能力、权限、完整 REST 路由、数据库诊断和队列状态。先调用此工具了解当前软件支持范围。 |
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
| `get_job` | 查询本 Agent 单条任务、错误、追问、排队位置和转换结果 ID。生图或应用提示词处于 awaiting_confirmation 时须等待用户在软件确认。 |
| `get_design` | 读取本 Agent 设计原文和版本记录；不将转换提示词当作原始设计。 |
| `get_prompt` | 读取本 Agent 转换后的正负提示词、目标模型、Skill 快照和审阅信息。 |
| `apply_prompt` | 请求把转换结果应用到编辑器，会覆盖现有正负提示词并可能切换模型。仅创建待确认任务，用户批准前不会修改编辑框。 |
| `request_generation` | 提交本会话已转换 Prompt ID 的生图请求，返回待用户批准任务。不能绕过确认，也不能直接提交任意系统命令或文件路径。 |
| `cancel_job` | 取消本 Agent 尚未运行、待确认或待补充的任务；正在执行的上游任务不保证能中断。 |
| `generation_history` | 分页查看本 Agent 的生成结果与实际请求记录，可按会话恢复历史。 |
| `save_model` | 请求创建或编辑模型定义，value 为现有业务模型对象。需要 models 权限并由用户审阅批准后执行。 |
| `save_skill` | 请求创建或编辑模型 Skill，value 含 modelId、名称与规则内容。需要 skills 权限，用户确认后才保存。 |
| `save_artist` | 请求创建或编辑画师资源，value 含 modelId 与画师串。需要 resources 权限，用户确认后保存。 |
| `delete_model` | 提交模型删除请求，需 models/delete 权限；用户批准前不删除任何记录或文件。 |
| `delete_skill` | 提交 Skill 删除请求，需 skills/delete 权限；遵守引用和内置资源保护。 |
| `delete_artist` | 提交画师删除请求，需 resources/delete 权限，仍须用户确认。 |
| `delete_resource` | 提交资源记录删除请求，需 resources/delete 权限，不提供任意文件删除入口。 |

### REST 路由

| 方法 | 相对路径 | 用途 |
| --- | --- | --- |
| GET | `/capabilities` | 能力、版本、就绪与队列状态 |
| GET | `/state` | 当前模型、Skill 与队列状态 |
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
| GET | `/prompts/{id}` | 转换结果 |
| POST | `/prompts/{id}/apply` | 请求覆盖编辑器，须用户确认 |
| POST | `/generations` | 请求生图，须用户确认 |
| GET | `/generation-history` | 本 Agent 生成历史 |
| GET | `/events?sessionId=&after=&timeout=90` | SSE；支持 Last-Event-ID 与 retry，按游标续传 |
| POST | `/models ／ /skills ／ /artists` | 管理操作请求，须用户确认 |
| DELETE | `/models/{id} ／ /skills/{id} ／ /artists/{id} ／ /resources/{id}` | 删除请求，须用户确认 |
