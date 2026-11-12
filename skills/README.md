# 模型 Skill 规范 · v7

八个模型或通道使用独立文件夹，软件切换模型时加载对应的默认规则。安装包已内置，也可下载独立的 mioha-model-skills-v7.zip。

| 模型／通道 | 规则 |
| --- | --- |
| 光辉 Illustrious XL | [SKILL.md](illustrious/SKILL.md) |
| NoobAI-XL | [SKILL.md](noob/SKILL.md) |
| NovelAI V5 | [SKILL.md](nai/SKILL.md) |
| Anima Base / Hybrid | [SKILL.md](anima/SKILL.md) |
| MOSS-Image 二次元 | [SKILL.md](moss-anime/SKILL.md) |
| MOSS-Image 真人摄影 | [SKILL.md](moss-real/SKILL.md) |
| Krea 2 二次元 | [SKILL.md](krea-anime/SKILL.md) |
| Krea 2 真人摄影 | [SKILL.md](krea-real/SKILL.md) |

同一模型的二次元、真人通道分别选择，默认不会混用。自定义 Skill 继续在自己的模型目录管理。切换提示词规范不改变生图后端、已选权重或 LoRA。

文件校验见 [manifest.json](manifest.json)。规则文本来自维护者提供的 v7 目录，保留原文件内容与哈希；模型权重与外部服务按各自说明配置。
