# 小梧桐提示词站

**Mioha Prompt Station · 把创意、提示词、绘图资源与生图流程放进同一个工作台。**

[![最新版本](https://img.shields.io/github/v/release/xialing114514/mioha-prompt-station?label=Release&color=456b89)](https://github.com/xialing114514/mioha-prompt-station/releases/latest)
![系统](https://img.shields.io/badge/Windows-10%20%2F%2011%20x64-0078D4)
[![使用指南](https://img.shields.io/badge/Docs-简体中文-607d69)](docs/README.md)

[**下载安装包**](https://github.com/xialing114514/mioha-prompt-station/releases/latest) · [快速上手](docs/QUICK_START.md) · [WD14 模型安装](docs/WD14.md) · [更新记录](CHANGELOG.md) · [问题反馈](https://github.com/xialing114514/mioha-prompt-station/issues/new/choose)

![提示词工作区与可视化权重编辑](assets/screenshots/prompt-workspace.png)

用中文描述画面，让**内置 AI**结合本地标签、角色配方与绘图模型 Skill 整理提示词；也可以自己选词、调整权重、管理 LoRA，再连接本机 NeoForge、ComfyUI 或已配置的 NovelAI 服务生图。

当前版本 **0.2.2**。软件名称统一为「小梧桐提示词站」，版本号在设置中查看。本仓库提供安装包、使用文档、模型 Skill 和发布记录。

## 一套完整的创作流程

| 从哪里开始 | 在软件中完成什么 |
| --- | --- |
| 已有画面想法 | 内置 AI 选词、读取本地角色、按模型 Skill 转换提示词 |
| 想自己控制细节 | 检索中英标签、正负词分开编辑、拖动排序、逐词调整权重 |
| 需要角色和风格素材 | 角色配方、画师串、模型与 LoRA 管理、资料索引与来源检索 |
| 想保留方案再换模型 | 原始设计独立保存，选择 Anima／MOSS／光辉／NAI 重新转换 |
| 已准备好提示词 | 连接绘图引擎，加载底模与 LoRA、联动触发词、查看任务与历史 |
| 只有参考图片 | 优先读取原始元数据；没有参数时使用本地 WD14 或自行配置的视觉接口 |
| 使用外部 AI Agent | 通过 MCP／REST／SSE 交接设计、调用内置 AI、申请生图和资源操作 |

## 内置 AI：从描述到模型适用的提示词

![内置 AI 选词与模型 Skill 选择](assets/screenshots/built-in-ai.png)

内置 AI 是工作台的一部分，提供可直接操作的选词、检索和转换流程。它调用在线服务或用户配置的大语言模型，**安装包不包含本地大语言模型权重**。

- **自然语言选词**：中文描述画面，生成标签或完整英文描述，保留正向与负向结果。
- **读取真实本地配方**：查询内置角色与用户新建、编辑的角色，读取外观、服装和角色标签；已删除项不会继续作为可选配方。
- **按绘图模型适配**：Anima、MOSS、光辉 Illustrious、NovelAI 各自加载 Skill，支持手动选择、兼容组合及自定义模型。
- **本地资料检索**：查询已导入的角色、画师、标签、素材与模型知识，保留可用来源信息。
- **可选联网查证**：开启后按工具能力查询角色资料；配置支持看图的接口时可辅助识别参考图。关闭后使用快速、本地资料优先的流程。
- **继续修改**：可以多轮补充要求；结果可以复制、追加或替换到编辑器，也可以发送到生图界面。
- **免费与自定义接口**：内置免费通道默认选择 **NVIDIA: Nemotron 3 Ultra (free)**；支持使用自己的 OpenAI 兼容接口及模型。

免费模型的额度、速度与可用性由服务提供方决定。程序保留用户手动选择，不会在 Ultra 不可用时静默换成 Super VL；可用「恢复默认模型」切回 Ultra。

[阅读内置 AI 使用指南](docs/BUILT_IN_AI.md)

## 提示词、角色与资料管理

| 功能 | 能力 |
| --- | --- |
| 提示词编辑 | 完整词库约 5.4 万条；中英文检索、中文释义、正负词框、拖动顺序、逐词权重、语法适配 |
| 角色配方 | 内置与自建角色、外观编辑、中文优先显示、分类管理；删除分类时可保留或同时移除角色 |
| 预设与存档 | 保存常用组合与创作状态，支持数据备份与恢复 |
| 随机组合 | 按主题和画面类型组合，支持种子与锁定项，适合建立创作起点 |
| 绘图资料中心 | 角色图鉴、画师图鉴、标签、提示词素材、模型知识与图片资源；本地索引、来源管理和按需缓存 |
| 画师与风格 | 按绘图模型管理画师串与风格内容，再按对应语法插入提示词 |
| 辅助工具 | 提示词识别、中英翻译、自定义标签、图片参数读取 |

![本地角色配方与分类管理](assets/screenshots/character-recipes.png)

资料能否在线获取、是否需要用户自己的 Token，以及可用于哪些模型，以来源状态和资料兼容性为准。第三方资料的权利仍归原作者与维护者。

## 模型中心、LoRA 与 Skill

![各模型独立的 Skill 规则与编辑空间](assets/screenshots/skill-center.png)

**绘图模型分类**与**生图引擎**分别管理。Anima／MOSS／光辉／NAI 是模型规范；NeoForge／ComfyUI／NovelAI API 是执行方式。选择某个分类，不代表对应权重已经安装。

- NeoForge 与 ComfyUI 分开读取资源，显示引擎归属、底模兼容性和本地绑定状态。
- 模型及 LoRA 支持信息获取、预览、介绍、触发词、收藏、筛选、批量任务与组合配方。
- 本地资源通过已有元信息或哈希获取 Civitai 资料；支持 Civitai 与 civitai.red 链接。预览受内容模式及来源分级影响。
- LoRA 加载与触发词联动，支持逐项开关；保留两个引擎各自的参数和工作流要求。
- 四种模型拥有独立 Skill 空间，另附 Anima Aesthetic 差异规范；自定义模型也有自己的目录。

[模型与生图指南](docs/MODELS_AND_GENERATION.md) · [模型 Skill 文件](skills/README.md)

## 原始设计、内置 AI 与外部 Agent

![设计方案与提示词转换工作区](assets/screenshots/design-workspace.png)

外部 Agent 或用户完成创意设计，内置 AI 负责转换为目标绘图模型的表达。原始设计与转换结果分别保存，可查看版本、补充反馈和重新转换。

![本机 AI Bridge 与独立 Agent 管理](assets/screenshots/agent-bridge.png)

AI Bridge 提供 **27 项 MCP 工具**及统一 REST 接口、SSE 事件流：独立 Token、权限范围、会话隔离、任务查询、追问恢复、编辑器应用确认，以及生图请求确认。安装目录附带独立 CLI／MCP 客户端，无需另装 Python。

[Agent 接入文档与配置示例](docs/AGENT.md)

## 连接生图与图片反推

![连接生图界面：引擎、提示词与参数配置](assets/screenshots/generation-center.png)

支持本机 NeoForge、ComfyUI 与已配置的 NovelAI API，提供提示词同步、模型与 LoRA 选择、任务状态、生成历史及程序内预览。NeoForge 可查看本次任务进度与实时预览。MOSS 等特殊架构需要匹配的模型组件和工作流。

反推按以下顺序处理：

1. **图片含生成元数据**：读取文件里的原始提示词、底模／LoRA 及可用参数。
2. **图片没有元数据**：使用已安装的本地 WD14 模型；也可手动选择已配置的视觉接口。

> **WD14 必须准备配套文件。** 只有一个 ONNX 模型文件不够：同目录还需要同一模型版本的 `selected_tags.csv`。安装包只提供插件，不附带模型，也不自动下载。

[**查看 WD14 下载与放置教程**](docs/WD14.md)

## 内容显示与安全模式

`0.2.2` 默认使用安全模式，隐藏标记为 NSFW 的标签、资料和未确认安全的预览。

- 首次可在左侧点「解除 NSFW 限制」，阅读说明并选择取消或明确确认。
- 确认后，侧栏不再保留该入口；之后通过「设置 → 内容显示」切换。
- 「解除 AI 限制」按钮须全局解锁后才可用；它控制软件自己的 AI 内容偏好，不改变服务提供方的规则。
- 安全模式不会删除原来的资源和创作数据。分级与标签筛选不等于图像内容识别，仍需自行核对导入和生成的素材。

## 开始使用

1. 从 [最新 Release](https://github.com/xialing114514/mioha-prompt-station/releases/latest) 下载 Windows 安装包；校验值在同页 `SHA256.txt`。
2. 自选安装目录，可创建桌面快捷方式。需要 **Windows 10/11 x64** 与 **Microsoft Edge WebView2 Runtime**。
3. 打开软件即可使用本地选词、角色配方等功能。使用 AI 时选择免费通道或配置自己的接口。
4. 需要生图时，再启动对应引擎或配置自己的 NovelAI 凭据；需要 WD14 时按教程安装配套模型文件。

[完整快速上手](docs/QUICK_START.md) · [常见问题](docs/FAQ.md) · [更多截图](docs/SCREENSHOTS.md)

## 文档与反馈

| 文档 | 内容 |
| --- | --- |
| [使用手册](docs/README.md) | 所有指南与阅读顺序 |
| [内置 AI](docs/BUILT_IN_AI.md) | 免费模型、自定义接口、本地角色检索、转换与查证 |
| [模型与生图](docs/MODELS_AND_GENERATION.md) | 引擎分工、资源、LoRA、工作流与历史 |
| [WD14 安装](docs/WD14.md) | 文件下载、目录示例、扫描与故障排查 |
| [Agent 接入](docs/AGENT.md) | MCP、CLI、REST、Token、权限和任务状态 |
| [数据与隐私](docs/DATA_AND_PRIVACY.md) | 本地数据、云同步范围、在线调用与备份 |
| [反馈与参与](CONTRIBUTING.md) | 提交问题、建议、文档和 Skill 修订 |

发现问题请使用 [反馈表单](https://github.com/xialing114514/mioha-prompt-station/issues/new/choose)，附软件版本、引擎、复现步骤和脱敏截图。不要上传 Key、Token 或个人数据库。

## 致谢、资源与支持开发

感谢 **柒月FIGO** 的设计启发，感谢 **znso4-pa**、**四路至强超上天** 与 **百濑美月** 的 AI 技术支持。

[完整致谢与第三方资源声明](CREDITS.md) · [使用资源与数据来源](THIRD_PARTY_RESOURCES.md) · [更新记录](CHANGELOG.md)

**感谢每一份支持 ♡** 如果这个工具帮助到了你，欢迎自愿赞助，支持后续功能开发、维护与更新。赞助完全自愿，不影响任何软件功能及使用权限。[查看支持开发页面](SUPPORT.md)

---

本仓库的公开发布范围为安装包、文档、模型 Skill 与相关展示资源，不代表软件源码或第三方内容以统一开源许可证发布。组件和模型按各自许可使用；外部服务的可用性与使用限制以提供方条款为准。
