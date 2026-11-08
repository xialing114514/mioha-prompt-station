# 小梧桐提示词站 · Mioha Prompt Station

面向 AI 绘画的 Windows 创作工作台：整理提示词与角色资料、管理绘图资源，让创意方案经模型专属 Skill 转换后进入生图流程。

**当前版本：0.2.1。** 本仓库发布安装包、使用说明、模型 Skill 与更新记录；程序源码保留在本地。

[下载安装包](https://github.com/xialing114514/mioha-prompt-station/releases) · [更新记录](CHANGELOG.md) · [模型 Skill](skills/README.md) · [反馈问题](https://github.com/xialing114514/mioha-prompt-station/issues)

## 能做什么

- **提示词工作区**：正负提示词、中文释义、词条拖动排序与权重、角色配方、预设和存档。内置 AI 读取同一本地角色配方，包含自建及编辑结果；分类可删除并选择保留或移除分类内角色。
- **模型 Skill**：Anima、MOSS、光辉 Illustrious、NovelAI 分别管理规范，支持 Aesthetic 差异规则与自定义模型。
- **模型与 LoRA 资源**：NeoForge 与 ComfyUI 分开读取，管理预览、介绍、触发词和适用模型；连接生图时联动触发词。
- **连接生图**：本机 NeoForge、ComfyUI 和配置后的 NovelAI 服务；保留引擎各自参数，支持任务状态、历史和程序内图片预览。
- **图片反推**：先读图像元数据；没有元数据时可使用独立 WD14 插件，不需要启动 NeoForge。模型由用户自行提供。
- **绘图资料中心**：统一检索角色、画师和标签资料，中文名称优先，保留来源信息。
- **AI 协作**：外部 Agent 提交创意设计，内置 AI 按当前 Skill 转换。REST／SSE／MCP 接入支持独立 Token、任务与会话隔离、用户确认。
- **更新检查**：设置页可跳转 GitHub、手动检查版本；启动时可每天检查一次公开 Release，包含内测版本。应用不会自动覆盖安装文件。

## 安装与首次使用

1. 从 [Releases](https://github.com/xialing114514/mioha-prompt-station/releases) 下载 Setup 安装包，可对照同页 SHA256.txt。
2. 选择安装目录，可创建桌面快捷方式。需要 Windows 10/11 x64 和 Microsoft Edge WebView2 Runtime。
3. 打开程序，在「模型中心／资源」设置本机 NeoForge 或 ComfyUI 目录；需要生图时先启动相应引擎服务。
4. 在「AI 选词」选择绘图模型与 Skill。默认免费内置模型为 **NVIDIA: Nemotron 3 Ultra (free)**。会保留已保存的用户选择，可点「恢复默认模型」切回 Ultra；不可用时提示，不静默改成 Super VL。免费通道的可用性和额度由提供方决定，也可配置自己的接口。
5. NovelAI、自建云同步和其他需认证的服务，应由每位用户配置自己的凭据。发行包不包含维护者的私人 Key 或 Token。

个人创作数据保存在 `%USERPROFILE%/.mioha-prompt-forge`。更换安装目录不会直接迁走该数据，卸载不会主动删除它。

## WD14 模型目录

安装包只提供插件，不附带模型，也不自动下载模型。将有权使用的 `model.onnx` 与 `selected_tags.csv` 放在 `models/wd14/自定义模型名/`，再在软件中扫描启用。

## Agent 接入

安装目录 `agent-tools/mioha-agent.exe --mcp` 提供 MCP，无需另装 Python；同一文件也提供 CLI。先在软件 AI Bridge 中创建自己的 Agent，复制只展示一次的 Token 到私人文件，用 `MIOHA_AGENT_TOKEN_FILE` 指向它。

地址默认从当前用户的 `agent-endpoint.json` 发现；手动指定 `MIOHA_WORKSPACE_URL` 可覆盖。不要公开 Token 文件。停用或删除 Agent 会使其 Token 失效；删除后保留创作历史。删除、覆盖提示词及最终生图按照业务权限要求由用户审阅确认。

## 致谢、资源与支持

- [完整致谢与第三方资源声明](CREDITS.md)，包括柒月FIGO、znso4-pa、四路至强超上天与百濑美月。
- [使用资源与数据来源](THIRD_PARTY_RESOURCES.md)。第三方网站、模型、组件和内容的权利归各自权利人。
- [支持开发](SUPPORT.md)：感谢每一份支持 ♡ 赞助完全自愿，不影响任何软件功能及使用权限。

## 使用说明

安装包提供基础代码保护与当前 Windows 用户级凭据加密，未进行发布者数字签名。不同绘图引擎、底模和 LoRA 的效果应在自己的配置下验证。程序标题与桌面快捷方式统一使用「小梧桐提示词站」，版本号显示在设置中。反馈时请提供软件版本、引擎及操作步骤，遮盖 Key、Token 与私人资料。
