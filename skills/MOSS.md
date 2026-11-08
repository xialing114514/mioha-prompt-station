# MOSS-Image 提示词规范（Base / SFT · 5B DiT）

## 模型规格

| 项 | 值 |
| --- | --- |
| 架构 | 5B 单流 DiT + Qwen3.5-9B 文本编码器 |
| 上下文 | 最长 4,096 tokens，长描述完全吃得下 |
| 版本 | `MOSS-Image-Base`（原始底座）、`MOSS-Image-SFT`（社区微调，文字与版式更稳） |
| 分辨率 | 总像素 1M / 1.5M / 2M 最稳，可外推 0.25M（512²）– 4M（2K） |
| 宽高 | 必须是 16 的正整数倍 |
| 强项 | 中英文字渲染、自动换行、版面控制、长自然语言、Layout 精确摆放 |
| 弱项 | 标签堆叠式短句、极小的字与生僻字 |

## 三条铁律

1. **要出现在画面上的字，全部放进引号。** 标点、价格、换行一并原样保留。模型按引号内容逐字渲染。
2. **描述成品画面，不写下指令。** 从上到下说清这张图长什么样：主体、布局、光线、质感。写「她站在雨里，伞面挂着水珠」这样的成片描述，不要写「请画一个下雨的场景」。
3. **细节用英文，文字用任何语言。** 英文训练数据最多，描述风格细节最稳；要渲染的文字可以是中文、英文或双语。

## 三档画面模式

| 模式 | 主体描述方式 | 停用条款 |
| --- | --- | --- |
| **写实 / 摄影** | 光学与显影参数打头：胶片型号、镜头焦段、色片、颗粒 | 用满全部五段 |
| **二次元 / 插画** | 媒介词打头：水彩、赛璐璐、厚涂、丝网印刷 | 不写毛孔、闪光灯、胶片颗粒 |
| **2D/3D 混合** | 按画面元素分别指定，不套单一基底 | 视需求取舍 |

复刻某部片的质感时，**不写片名和演员名**，改写光学参数：胶片型号、镜头、色片、灯具形态、受光位置。名字带不来质感，参数能。

## 描述顺序

**写实档五段**

1. 光学与显影：`shot on 35mm film, Kodak Portra 400, slight halation`
2. 机位与遮挡：`eye-level wide shot, foreground desk corner blurs into frame`
3. 灯具与受光：`single softbox from the left at 45°, hard falloff on the cheek`
4. 年代与调色：`late-90s Hong Kong color, teal shadows, magenta highlights`
5. 质感与瑕疵：`subtle grain, dust on the lens, a chipped mug on the counter`

**二次元档五段**

1. 媒介与画风：`painterly anime illustration, gouache texture, clean lineart`
2. 主体与外观：`a girl with glossy cream blonde low twintails`
3. 构图与镜头：`upper body, three-quarter view, shallow depth of field`
4. 光色：`warm desk lamp from the right, cool rain light outside`
5. 氛围与留白：`quiet evening, generous negative space above`

每段交代一件事，段与段不重复。极简需求保持极简，别为了「丰富」加不相干的物件。

## 图内文字工程

这是 MOSS 的主场，写法直接决定成败。

| 技巧 | 做法 |
| --- | --- |
| 引号 | 每段文字单独加引号，一行一个引号，防止价格错配到别的菜名 |
| 换行 | 在引号内直接换行或写 `\n`，断行位置由你决定 |
| 短行 | 一行别太长，长段落和生僻字最容易崩 |
| 位置 | 文字放角落、边缘，别压主体 |
| 分区 | 用「左侧 / 右侧 / 顶部 / 底部」切开版面，让模型分别填 |
| 列结构 | 先说「左时间、右议题」，行与行才会对齐 |
| 材质 | 写「深绿纹理卡纸上的烫金」比写「高级感」有效十倍 |
| 收尾 | 加一句 `Keep all text sharp, fully visible, and free of glare.` |

**引号里的内容不一定是要画的字。** 「想要时间慢下来的感觉」是情绪参考，不能当成标题渲染。只有明确要出现的文案才进引号。

**占位词禁止。** 不要写「此处放标题」，直接写标题本身。用户没给文案又要求代拟时，可以创作，但不得虚构事实信息（时间、价格、参数）。

## 常见排版类型

| 类型 | 组织方式 |
| --- | --- |
| 海报 | 主标题 + 副标题 + 视觉主体，信息分层，一行一件事 |
| 菜单 / 价目 | 每道菜单独占一行并加引号，价格跟在同行 |
| 招牌 / 门头 | 招牌是照片的一部分，用镜头语言控制写实度 |
| 书籍 / 专辑封面 | 只放一个中心元素，给标题与作者名留干净空间 |
| 板书 / 便签 | 指定笔色与手写体，字才像真写上去 |
| 产品包装 | 曲面上的字宜短，两行以内最好 |
| 信息图 | 写清「左图标、右说明」的固定结构，编号行样式统一 |

## 标签序列（可选写法）

Base 也吃 booru 标签，官方顺序：

```
Quality -> Score -> Meta -> Year -> Character -> Series -> Artist -> General
```

例：`best quality, highres, 1girl, solo, school uniform, holding umbrella, rain, puddle reflection`

画师前加 `@`。这套写法适合二次元角色图；摄影、海报、版式类需求仍走整段描述。

## 负面词

官方组合：

```
nsfw, worst quality, bad anatomy, watermark, jpeg artifact
```

按需追加：`ugly face, extra digits, fewer digits, text artifacts, blurry, lowres, multiple views, logo`。

摄影/写实档可加 `illustration, anime, cartoon, 3d render` 压住绘画感；二次元档反过来加 `photorealistic, photograph`。

## 生成参数

| 参数 | 值 |
| --- | --- |
| 步数 | ≥50 |
| CFG | 3.0 起（长描述够用）；短描述、标签式可升到 4–5 |
| Shift | 5.0（写实常用 6.0） |
| 采样器 | `euler`（配 `normal` 或 `sgm_uniform` 调度器） |
| SLG | 标准 SLG scale 3；两段式 skip layer 22 / 24 / 26，26 常用于写实 |
| 尺寸示例 | 1360×768、1456×1456、1536×1024、1680×944、1024×1024 |

关掉 SLG 会更写实，细节容易塌。要插画质感就留着。

## Layout 模式

只有需要精确摆放时用：指定文字位置、多格分镜、受控物体关系。细节多不等于要 Layout。

**草稿**（交网页编译）

```json
{
  "width": 1536,
  "height": 864,
  "bbox_coord_space": "abs",
  "bbox_format": "xyxy",
  "summary": {"caption": "整体英文描述", "canvas": "single"},
  "items": [
    {"label": "object", "caption": "透明玻璃杯，冰块与青柠片", "bbox": [160, 144, 704, 768]},
    {"label": "text", "caption": "深绿粗体中文标题，无底板", "text": "这一口，刚好入夏。", "bbox": [800, 192, 1472, 336]}
  ]
}
```

规则：

- `bbox` 左上角像素坐标 `[x_min, y_min, x_max, y_max]`，在 `0..width` / `0..height` 内。
- `label` 只取 `object` / `character` / `text`；`text` 单元必须有 `text`。
- 多格分镜里，整身格用 `character`，道具特写用 `object`——混用会让模型多长出人头。
- 每个元素一个框；共用、重叠都允许，框不是要画出来的边框。
- 换画布尺寸不会自动缩放框，改尺寸要重排坐标。
- 坐标换算由网页做，不手算 `abs_centralized`，不写归一化百分比。
- 编译产物是单行 JSON，`bbox_format: yxyx`、`bbox_coord_space: abs_centralized`，不带宽高；宽高在生成设置里另填。
- 气泡对白要写清说话人归属，否则模型会把台词贴到错的人头上。

## 中文输入怎么扩写

用户给的是中文短句时，按这八条落地：

| 输入 | 处理 |
| --- | --- |
| 短句 | 先定主体与动作，再补环境、光线、构图 |
| 情绪词（孤独、温暖） | 落实为视线、姿态、人物距离、留白、光色 |
| 风格词（高级、电影感） | 换成具体视觉选择，不把抽象词原样堆进 prompt |
| 关系描述 | 先写左右前后、接触、视线，再补服饰背景 |
| 中文文字 | 原文单独识别，英文描述里保留中文原文，不翻译 |
| 未指定的细节 | 只补与核心意图相容、好改的细节，不新增改变主题的设定 |
| 矛盾要求 | 能分层共存就说清；同一属性冲突就指出并让用户选 |
| 迭代（「再冷一点」） | 视为修改上一版，保留未要求改动的部分 |

极简需求就写极简，扩写不等于加物件。

## 自检

1. `prompt` 是一段完整成片描述，不是标签数组。
2. 画面上的字全在引号里，换行位置可追溯。
3. 抽象词已换成具体视觉（光位、焦段、材质、色温）。
4. 没有把操作说明、坐标解释、占位文案写进描述。
5. 写实档没有漏掉光学参数，二次元档没有混进胶片颗粒。
6. Layout：坐标在画布内、宽高是 16 倍数、`text` 有值、分镜用对了 `character` / `object`。
7. 负面词 5–15 个，和画面模式匹配。

## 来源

- 官方：https://github.com/OpenMOSS/MOSS-Image/blob/main/skills/moss-image-prompt-enhancer/SKILL.md
- 官方：https://github.com/OpenMOSS/MOSS-Image/blob/main/USERGUIDE.md
- 社区：https://moss-image.com/zh/moss-image-prompts（24 条官方示例与写法指南）
- 社区：https://github.com/Moeblack/moss-image-prompt-skill（三档模式、参数、闭合标签语法；CC BY-NC-ND 4.0，此处仅取事实性信息并署名）
- 社区：https://github.com/Work-Fisher/fisher-moss-prompt-expander（中文扩写方法与 Layout 校验规则，基于官方 skill 改编）
