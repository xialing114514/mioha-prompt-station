# 致谢与第三方资源声明

## 一、特别致谢

本项目在功能设计、交互布局及提示词生成流程方面，参考并借鉴了柒月FIGO大佬的提示词生成器。感谢原作者提供的优秀设计思路与创作启发。

同时，特别感谢 **znso4-pa**、**四路至强超上天** 与 **百濑美月** 在项目开发过程中提供的 AI 技术支持与帮助。

## 二、第三方数据来源

本项目整合了以下公开网站的部分数据与资料，用于构建角色索引、画师索引、标签查询及提示词素材库。

- **Luckyfish Illustrious**

用途：Illustrious 角色查询与索引。
[原始地址：Luckyfish Illustrious](https://zhuanqfish.github.io/Luckyfish-illustrious-character-select/)

- **AnimaDex**

用途：角色与画师索引。
[原始地址：AnimaDex](https://animadex.net/?mode=characters)

- **TAGs Gallery**

用途：NovelAI V4.5 / V5 标签查询与预览。
[原始地址：TAGs Gallery](https://tags.gallery/zh-cn/v4-5)

- **NovelAI QuickTagCloud**

用途：画师索引与提示词素材参考。
[原始地址：NovelAI QuickTagCloud](https://novelai.quicktagcloud.com/?c=artist_nai5_personal&p=1h3dewu)

- **SAkizuki / DanbooruSearch**

用途：Danbooru 标签查询与图片预览。
[原始地址：SAkizuki / DanbooruSearch](https://huggingface.co/spaces/SAkizuki/DanbooruSearch)

本项目对上述来源的资料进行整理、索引与检索，以提供统一的使用体验。各来源网站及其内容仍归原作者、维护者或相应权利人所有。

第三方图片资源优先保留原始来源及链接。图片引用、缓存和展示均应遵循对应来源的使用规则与授权要求。

## 三、功能与技术参考

本项目在部分功能设计及技术实现方面参考了以下开源项目：

- **ComfyUI LoRA Manager**：LoRA 资源管理、资源展示及管理流程方面的设计参考。原项目采用 GPL-3.0 许可证。
- **ComfyUI NewBie LLM Formatter**：不同绘图模型的提示词格式适配与转换思路参考。
- **柒月FIGO 提示词生成器**：提示词生成器的部分功能、交互设计与布局参考。
- **dsh-our-free-model**：模型接入及免费模型通道相关功能参考。原项目采用 MIT 许可证。

以上项目的代码、设计及相关成果均归各自原作者所有。涉及第三方代码的实际使用、修改或再分发时，将按照对应许可证要求处理。

## 四、开源技术与模型依赖

本项目使用或集成了以下技术与组件：

- Vue
- Vite
- Vue Router
- Python
- SQLite
- pywebview
- ONNX Runtime

图像标签识别功能使用 **SmilingWolf WD14 模型系列**。其他反推模型由用户自行提供，其使用权限及相关许可由对应模型作者决定。

第三方开源组件、模型及依赖的许可证，以实际发行版本附带的第三方许可证清单为准。

## 五、版权与免责声明

- 本项目为独立开发的软件项目，与上述第三方网站、工具及模型的原作者或运营方不存在默认的官方合作、隶属或背书关系。
- 第三方网站的名称、商标、图片、标签、角色资料及其他内容归其各自权利人所有。
- 本项目对第三方资料的整理与索引，不代表取得了相关内容的所有权或无限制再分发授权。
- 第三方开源组件按照其各自许可证使用，具体版权声明及许可证文本以软件发行包内的相关文件为准。
- 用户自行导入的模型、LoRA、反推模型及其他资源，其合法性与使用权限由用户自行确认。
- 本项目调用的第三方模型服务及在线 API 由各自提供方独立运营，其可用性、免费额度、数据处理方式及使用限制以服务提供方的最新条款为准。
- 如相关权利人认为本项目展示或引用的内容存在版权、授权或其他问题，可通过项目反馈渠道联系维护者。经核实后，我们将及时采取相应处理措施。

感谢所有开源项目作者、数据维护者及社区贡献者。

**感谢每一位为开源社区与 AI 绘画生态作出贡献的人。**

- **[DeepSeek Harness](https://github.com/deepseek-ai/deepseek-harness)**：对话循环和内部 Agent 组合机制参考，移植提示词投影、请求头规范化与工具中断恢复代码。Copyright (c) 2026 DeepSeek，MIT 许可证；发行包保留原许可。
