# MOSS-Image · 二次元 SKILL v5（moss-anime）

> 适用：MOSS-Image Base / SFT 的插画、赛璐璐、厚涂、水彩、海报型二次元需求。
> 与 `moss-real` 是两份独立 skill，**共用负面会互相打架**，不要同时启用。
> 本 skill 由 v4 `MOSS.md` 拆出并重写：轻画师标签，重标签结构 + 自然语言氛围句。负面已过滤 NSFW 项。

---

## 一、模型参数默认值

| 参数 | 默认 | 区间 | 说明 |
| --- | --- | --- | --- |
| 采样器 | `euler` + `normal` | `euler` + `sgm_uniform` | MOSS 认 flow-matching Euler |
| 步数 | 50 | ≥50 | 官方下限就是 50；低于 40 细节塌 |
| CFG | 3.0 | 3.0–5.0 | 长描述 3.0；短标签串可升到 4–5 |
| Shift | 5.0 | 5.0–6.0 | 二次元用 5.0 |
| SLG | 开，scale 3，skip layer 22 | 22 / 24 / 26 | 开 SLG 保细节；关掉更「真」但会崩 |
| 分辨率 | 1M 总像素 | 1M / 1.5M / 2M | 宽高必须是 16 的正整数倍 |
| 版式 | 1024×1024 / 832×1248 / 1360×768 | — | 竖图优先 832×1248 |

尺寸速查：1:1 `1024×1024`；2:3 `832×1248`；3:2 `1248×832`；16:9 `1360×768`；9:16 `768×1360`。

---

## 二、正向结构：三段式

```text
[结构段] 媒介与画风 → 人数 → 角色 → 外观 → 服装 → 动作 → 镜头 → 场景
[氛围段] 2–4 句自然语言：光怎么落、空气里有什么、这一瞬在发生什么
[收尾段] 质量补充（≤2 项）
```

| 段 | 作用 | 长度 |
| --- | --- | --- |
| 结构段 | 决定「画成什么样、画的是谁、在哪」 | 25–60 个标签 |
| 氛围段 | 决定「这一刻的空气」 | 2–4 句，60–150 词 |
| 收尾段 | 材质与完成度补充 | 1–2 项 |

**为什么结构段要标签化**：MOSS 的 4,096 token 上下文很宽，但它对「离散事实」的解析在标签形态下最稳（人数、发色、机位、服装形制）。整段散文写服装，容易把上衣和裙子的颜色串掉。

**为什么氛围段要句子**：光的方向、空气的湿度、主体的情绪落点，只有句子能说清；标签写 `warm light` 与写 `Late sun comes through the blinds and lands across her collarbone` 是两个画面。

---

## 三、结构段槽位

| # | 槽位 | 内容 | 可省略 |
| --- | --- | --- | --- |
| 1 | **媒介 / 画风** | `painterly anime illustration`、`cel shading`、`watercolor`、`gouache texture`、`clean lineart` | 否 |
| 2 | **人数** | `1girl, solo` / `2girls` / `no humans` | 否 |
| 3 | 角色 | 角色名 + 作品名；原创写外观锚点 | 是 |
| 4 | 外观 | 发长、发色、瞳色、体型、非人特征 | 否 |
| 5 | 服装 | 款式 + 颜色 + 材质 + 穿着状态 | 否 |
| 6 | 动作 / 表情 | 姿势 + 接触点 + 表情 + 视线 | 否 |
| 7 | 镜头 / 构图 | 景别 + 机位 + 透视 | 否 |
| 8 | 场景 | 地点 + 时段 + 天气 | 否 |
| 9 | 光 / 色 | 光源方向 + 色温 + 主色 | 推荐 |
| 10 | 画师 | `@画师名`，≤2（见第五节） | 是 |
| 11 | 质量补充 | `detailed eyes`、`clean lineart`、`intricate details` | 是 |

---

## 四、氛围段写法

一句一件事，按「光 → 空气 → 主体反应 → 环境参与」推进：

```text
Warm afternoon light comes from the left and catches the dust above the desk.
A thin haze softens the far wall, and the curtain edge glows where the sun passes through.
She keeps her eyes down on the page, one hand resting flat on the paper.
```

三条纪律：

1. **不写负面句**。要「不要逆光」就写 `front light`，不要写 `no backlight`。
2. **不写画外之物**。谁在按快门、相机在哪只手上，画里没有这台设备，写了模型会画一台相机出来。
3. **不重复结构段**。氛围段里的外观词只留辨认必需的锚点，完整外观在结构段。

---

## 五、画师控制：已知局限与替代方案

**已知局限**：MOSS 对画师标签的控制明显弱于 NAI 与光辉系。写 `@画师名` 有时只带来极轻微的笔触偏移，有时完全无反应；把画师串当主风格开关用会失望。

**替代方案（按可靠性排序）：**

| 方案 | 写法 | 可靠性 |
| --- | --- | --- |
| 1. 风格描述词 | `painterly anime illustration, gouache texture, visible brushstrokes, paper grain` | 高 |
| 2. 媒介 + 工艺 | `screen printing, flat color separations, slight misregistration` | 高 |
| 3. 线条与上色方式 | `thin colored lineart, cel shading with soft gradients, rim light on hair` | 高 |
| 4. 时代与流派 | `1990s anime style, retro artstyle, high contrast` | 中 |
| 5. 画师标签 | `@画师名`，≤2，权重靠前置 | 低 |
| 6. 参考图 | 用同一画师的图做 Vibe / 图生图 | 取决于工具 |

**不认的语法**：`N::tag::`、`{}`、`source#` 这类前缀词一律不解析；`(tag:1.2)` 在 MOSS 前端也不保证生效，权重靠**词序前置**而不是括号。

**结论**：画师名当锦上添花，不当主开关；主风格一律用「媒介 + 工艺 + 线条」三件套描述。这是本 skill 与 NAI skill 最大的分歧点。

---

## 六、负面基线（已过滤 NSFW）

**必需层**

```text
worst quality, bad anatomy, watermark, jpeg artifact
```

**按需层**

```text
blurry, out of focus, lowres, bad hands, extra fingers, fused fingers, missing fingers,
deformed hands, distorted face, asymmetrical eyes, extra limbs, low quality
```

**可选层（二次元通道的风格封印，一套，不混用）**

```text
photorealistic, photograph, live action, 3d render, cgi, plastic skin,
sketch, thick black outlines, vector art
```

按画面取用：要水彩就别否 `watercolor` 相关；要线稿风就别否 `sketch`；要厚涂就别否 `painterly`。

**图里要画字时**：去掉 `watermark`（它会把正常文字一起压掉），文字照第三节的引号规则写进正向。

**已过滤 NSFW 项**：官方基线里的 `nsfw` 已移除；语料里的 `loli`、`child`、`cleavage` 等词条替换为 `bad proportions`、`distorted face`、`unnatural body`。本 skill 负面基线不含任何 NSFW 词条——负面词是采样方向，不是内容过滤器，这一点必须写清。

---

## 七、图内文字（二次元场景常用）

- 要出现在画面上的字，全部放进引号，标点与换行原样保留：`text "放学后见"`。
- 字放在角落、边缘、留白处，别压主体；气泡要写清归属。
- 长段落与生僻字最容易崩；小字目标高度 ≥24px。
- 收尾加一句 `Keep all text sharp, fully visible, and free of glare.`

---

## 八、自检清单（交付前逐条过）

1. 结构段槽位顺序对，人数只写一次，标签无重复。
2. 氛围段 2–4 句，每句一件事，没有负面句、没有画外之物。
3. 画师标签 ≤2，且主风格不是靠画师名撑的（有媒介 / 工艺 / 线条描述兜底）。
4. 景别、机位、透视各只有一个，没有互相打架。
5. 光有方向（`from the left` / `backlight`），不是一句 `good lighting`。
6. 负面分层取用，只开一套风格封印，无 NSFW 词条。
7. 要画字时，`watermark` 已去掉，字全在引号里。
8. 分辨率宽高是 16 的倍数，总像素 1M 附近。

---

## 九、完整示例

### 示例 1｜单人 · 教室傍晚

```text
painterly anime illustration, clean lineart, soft cel shading with gradients,
1girl, solo, long black hair, blunt bangs, amber eyes, school uniform, white shirt, navy sailor collar,
red ribbon tie, pleated skirt, sitting on a desk, legs crossed, holding a notebook, looking at viewer,
slight smile, upper body, three-quarter view, from slightly below,
classroom after school, rows of desks, chalk dust in the air, warm light from the windows on the left,
golden hour, long shadows on the floor,

Late afternoon light comes in low from the left windows and lays a warm band across the desks.
Chalk dust drifts in the beam, and the far wall falls into a soft grey-blue shadow.
She keeps the notebook against her chest and looks up, one corner of her mouth lifted.
The empty classroom stays quiet behind her, curtains barely moving.

detailed eyes, clean lineart
```

负向：

```text
worst quality, bad anatomy, jpeg artifact, blurry, out of focus, lowres, bad hands, extra fingers,
fused fingers, deformed hands, distorted face, asymmetrical eyes, photorealistic, photograph, 3d render, cgi
```

### 示例 2｜双人 · 雨夜天台

```text
anime key visual, thick paint, visible brushstrokes, high contrast,
2girls, one on the left, one on the right, facing each other, wet hair, school uniforms, soaked shirts,
hands clasped, looking at each other, tense expressions,
cowboy shot, from side, dutch angle, rooftop at night, chain-link fence, city lights below, heavy rain,
cyan and magenta neon reflections on the wet floor,

Rain falls in visible sheets between them and the city glows far below the fence line.
Neon from a sign behind the camera throws cyan on one face and magenta on the other.
Water runs off their sleeves and drips from their clasped hands onto the concrete.
Neither of them looks away.

intricate details, cinematic composition
```

负向：

```text
worst quality, bad anatomy, jpeg artifact, blurry, lowres, bad hands, extra fingers, deformed hands,
distorted face, extra limbs, photorealistic, photograph, live action, cgi, plastic skin
```

### 示例 3｜场景主导 · 无人神社

```text
anime scenery illustration, gouache texture, paper grain, muted palette,
no humans, wide shot, from below, stone torii gate, long stone steps, overgrown lanterns,
cedar forest, morning mist, low sun behind the gate, god rays between trunks, fallen leaves,

Morning mist sits low between the cedars and the sun breaks behind the torii in thin rays.
The stone steps climb into the fog and disappear, moss creeping over the lantern bases.
A few leaves drift down and settle on the wet stone without a sound.

detailed background, cinematic composition
```

负向：

```text
worst quality, bad anatomy, jpeg artifact, blurry, lowres, out of focus, photorealistic, photograph,
live action, 3d render, cgi, watermark
```

---

## 十、常见错误

| 错误 | 后果 |
| --- | --- |
| 把画师串当主风格 | 画风几乎不动，白白占 token |
| 结构段写成散文 | 服装颜色串位、人数漂移 |
| 氛围段写负面句 | 反向激活不想要的东西 |
| 写「相机在我的手里」 | 画面里多出一台相机 |
| CFG 拉到 7 | 边缘发硬、色彩过饱和 |
| 步数 30 | 细节塌，尤其是手与眼 |
| 宽高不是 16 的倍数 | 直接报错 |
| 要画字却留着 `watermark` | 字整体消失 |

---

## 十一、来源

- 官方提示词增强 skill：https://github.com/OpenMOSS/MOSS-Image/blob/main/skills/moss-image-prompt-enhancer/SKILL.md
- 官方使用指南：https://github.com/OpenMOSS/MOSS-Image/blob/main/USERGUIDE.md
- 二次元通道与分层负面：https://github.com/jialanhu0915/silver-moss-image-prompt-skill
- 图内文字与排版：同上的 `references/core/text-in-image.md`
- 群聊语料拆解：`资料文件夹/群聊语料拆解报告.md`
