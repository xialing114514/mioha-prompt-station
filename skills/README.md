# 模型 Skill 规范 · v4

这些 Markdown 文件供软件内置 AI 转换提示词时阅读，不作为程序执行。四类模型的语法独立，切换模型时加载对应规范。

| 模型 | 规则文件 |
| --- | --- |
| Anima Base | [Anima.md](Anima.md) |
| Anima Aesthetic | [AnimaAesthetic.md](AnimaAesthetic.md)，与 Base 一起选用 |
| MOSS-Image | [MOSS.md](MOSS.md) |
| 光辉 Illustrious XL | [Illustrious.md](Illustrious.md) |
| NovelAI | [NAI.md](NAI.md) |

新安装包已内置这些规则。升级时替换旧内置条目，原始设计和已有转换结果保留；自定义模型的专属目录独立。

手动使用：打开软件「Skill 中心」，选择对应模型，导入 Markdown，检查兼容模型并保存，再在 AI 选词区选择需要启用的 Skill。

规则中的上游来源见各文件末尾；具体衍生底模、模型版本和在线服务设置仍以所用模型的实际说明为准。Anima 示例使用明确成年原创角色。文件校验信息见 manifest.json。
