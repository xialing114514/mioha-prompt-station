# 使用资源与数据来源声明



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

用途：NAI 在线法典、画师索引、画风组与提示词素材参考。原站在线读取；用户主动选择的条目可保存在本地。
[原始地址：NovelAI QuickTagCloud](https://novelai.quicktagcloud.com/?c=artist_nai5_personal&p=1h3dewu)

- **SAkizuki / DanbooruSearch**

用途：Danbooru 标签查询与图片预览。
[原始地址：SAkizuki / DanbooruSearch](https://huggingface.co/spaces/SAkizuki/DanbooruSearch)

本项目对上述来源的资料进行整理、索引与检索，以提供统一的使用体验。各来源网站及其内容仍归原作者、维护者或相应权利人所有。

第三方图片资源优先保留原始来源及链接。图片引用、缓存和展示均应遵循对应来源的使用规则与授权要求。



完整功能参考、依赖与版权声明见 [致谢与第三方资源声明](CREDITS.md)。

## 4.0 远程运行组件

远程数据通道使用 aiortc、aioice、PyAV、pylibsrtp 等组件。Windows 的 PyAV 运行依赖包含动态 FFmpeg 及其他原生库，各自保持原许可证；发行包提供原始许可证和来源清单。[同版本源码与构建资料](https://github.com/xialing114514/mioha-prompt-station/releases/download/v4.0.0/Mioha-4.0.0-FFmpeg-ThirdParty-Sources.zip) 与安装包一同提供。上游：[aiortc](https://github.com/aiortc/aiortc)、[PyAV](https://github.com/PyAV-Org/PyAV)、[FFmpeg](https://ffmpeg.org/)。
