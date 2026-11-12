# NovelAI V5 提示词 SKILL v5

> 适用：NovelAI Diffusion V5 / V4.5 Full。V4 Full 与 V3 的差异见「版本对照」，参数栏给 V5 默认。
> 本文件由 v4 `NAI.md` 升级而来，新增多角色站位语法、画师串混用规则、漫画分镜逻辑、三栏交付契约。负面已过滤 NSFW 项。

---

## 一、模型参数默认值

| 参数 | 默认 | 区间 | 说明 |
| --- | --- | --- | --- |
| 采样器 | Euler Ancestral | Euler Ancestral / DPM++ 2M | Ancestral 系配 SMEA 更稳 |
| 步数 | 28 | 28（Opus 档 ≤28 不耗 Anlas） | 再多收益很小 |
| Prompt Guidance | 7.0 | 5.0–8.0 | 低更绘画化，高更锐利；>8 边缘过饱和 |
| Noise Schedule | karras | native / karras / exponential | karras 通用 |
| SMEA | 开 | 开 / 关 | 细线多、画面碎时关掉试试 |
| 分辨率 | 832×1216 / 1216×832 | 1024²、1536×1024 | 超范围耗点数 |
| 质量词开关 | 开 | 开 / 关 | 开着就别再手写质量词；图里要画字时关掉 |

社区实测常用串：`步数28, Euler Ancestral, Prompt_Guidance = 7.0, Noise_Schedule = karras`。

---

## 二、三栏交付契约

NAI 的输入是**三个独立字段**，混着写是最常见的错误来源。

```text
Base Prompt:
<人数 / 风格块 / 镜头 / 场景 / 光线 / 关系句 / 质量词>

Character 1:
<性别, 发型·瞳色·体征, 服装·配饰, 本角色的动作与表情（可选）>

Undesired Content:
<负面串>
```

| 规则 | 说明 |
| --- | --- |
| 外貌只写一遍 | 写进 Character 栏，Base Prompt 里不再重复 |
| 动作/镜头/场景只写 Base | 不要漏进 Character 栏 |
| 人数只写一次 | Base 开头写 `2girls`，Character 栏只写 `girl` |
| 质量词放最后 | 所有描述句之后 |
| 栏名不进代码块 | 代码块里只放要粘贴的串 |

**版权角色极简原则**：NAI 认识的版权角色，只写 `角色名 (作品名)`，不要再补发色瞳色胸围；只有换装时才写服装。

---

## 三、正向槽位模板

| # | 槽位 | 内容 | 可省略 |
| --- | --- | --- | --- |
| 1 | **人数** | `1girl, solo` / `2girls` / `no humans` | 否 |
| 2 | **身份** | `角色名 (作品名)` 或 OC 外貌锚点 | 否 |
| 3 | 画师串 | 3–6 位，见第五节 | 推荐 |
| 4 | 年份 / 时代 | `year 2025`、`newest` | 是 |
| 5 | 外观 | 发/瞳/体型/非人特征 | 否 |
| 6 | 服装 | 款式 + 状态（穿/脱/半褪） | 否 |
| 7 | 动作 / 表情 / 视线 | 姿势 + 接触点 + 表情 | 否 |
| 8 | 镜头 / 构图 | 景别 + 机位 + 透视 | 否 |
| 9 | 场景 / 光 | 地点 + 时段 + 光源方向 | 否 |
| 10 | 关系句 | 空间、占比、进行中的动作、光影叙事（1–3 句） | 推荐 |
| 11 | 复杂度 / 质量 | `high complexity` / `very aesthetic, masterpiece, no text` | 是（质量词开关开着就省） |

**重要内容放前半段**：NAI 对前段权重更高，这是它与 SDXL 差异最大的一点。

---

## 四、多角色站位语法（本 skill 核心）

### 4.1 官方字段法（最稳，优先用）

```text
Base Prompt:
2girls, from side, medium shot, upper body, Character 1 and Character 2 mutual#embracing,
arms around each other, indoor room, warm lamp light from the right, very aesthetic, masterpiece, no text,

Character 1: girl, long black hair, red eyes, white dress, source#hug, soft smile, eyes closed,
Character 2: girl, short blonde hair, blue eyes, black coat, target#hug, blush, eyes closed,
```

- `Character N` 编号从**看画的人这边**从左往右数，最左边是 1。
- 主串里用同一个编号分配动作与道具，不要写 A/B。
- 站位靠 `Character 1 on the left, Character 2 on the right` 或界面里的 Custom Position（Y 轴下=近、上=远）。

### 4.2 花括号分组法（`{人物[tags]人物}`）

把一个角色的整段特征用 `{}` 包起来，让权重作用于整组，这是 NAI 社区最常用的紧凑写法：

```text
2girls, {girl, long black hair, red eyes, white dress, source#hug, soft smile},
{girl, short blonde hair, blue eyes, black coat, target#hug, blush},
indoor room, warm lamp light, very aesthetic, masterpiece, no text
```

`{...}` 每层 ×1.05，`{{...}}` ×1.1；组内可以再嵌 `[tag]` 做局部降权。

**取舍**：角色数 ≥3、或要做精确站位时走 4.1 的字段法；2 人轻量场景用 4.2 更省 token。两种不要混着写同一张图。

### 4.3 互动前缀

| 前缀 | 谁写 | 例 |
| --- | --- | --- |
| `source#{动作}` | 动作发起方 | `source#poking cheek` |
| `target#{动作}` | 动作承受方 | `target#poking cheek` |
| `mutual#{动作}` | 双向对称动作 | `mutual#holding hands`、`mutual#eye contact` |

前缀只绑定施受方向，表情与气场仍由常规 tag 控制。主串里也可以直接写 `Character 1 and Character 2 mutual#kiss`。

### 4.4 站位硬规则

- 先写人数与站位，再写各自外观：顺序反了必然串脸。
- 每人都要有 1 个中心锚点（`on the left` / `in center` / `on the right`）。
- 两人拥抱、接吻这类接触，允许位置重叠，但**接触点必须写清**（`arms around each other`、`hand on the other's cheek`）。
- 画面里出现第三个人却没写人数，模型会自己加人。

---

## 五、画师串混用规则

| 规则 | 内容 |
| --- | --- |
| 格式 | `artist:画师名`，纯逗号并列；名字里的括号原样保留 |
| 数量 | 2–4 位常用，5–6 位上限；超过 6 位互相打架 |
| 位置 | 紧跟角色之后、年份之前；太靠后会被稀释 |
| 权重 | `1.4::artist:x::` 抬、`0.5::artist:x::` 压、`-2::artist:x::` 踢出去 |
| 防拼贴 | 必须配 `-2::artist collaboration::`（群里几乎人手一条） |
| 官方风格 | `official art` 与画师串同时出现时，`official art` 权重给低（0.3–0.6），否则会盖掉画师 |
| 查不到 | 照写 `artist:原名`。本地词典是抽样收录，查空不代表不存在 |

三种可复用的串型：

```text
# 稳定型：主 + 副 + 质感
1.3::artist:ciloranko::, 0.8::artist:yoneyama mai::, 0.6::artist:quasarcake::

# 厚涂型：主 + 厚涂 + 排除平涂
1.2::artist:name::, 0.7::artist:name::, 1.5::thick paint::, -1::flat color::

# 双人同框型：角色在前，画师在后，强力防拼贴
2girls, A (series), B (series), kiss, 1.2::artist:name::, 0.8::artist:name::,
-3::artist collaboration::, -2::group::
```

---

## 六、权重语法

**括号法（V2 起，全局可用）**：`{tag}` ×1.05、`{{tag}}` ×1.1025、`[tag]` ÷1.05、`[[tag]]` 继续除；嵌套继续乘除。

**数值法（V4 起，推荐）**：`1.5::tag::`、`0.5::tag::`、`-2::tag::`；末尾一个 `::` 自动闭合所有段。负值适合点杀（`-1::flat color::`、`-3::multiple views::`、`-1::nostril::`），不适合替代整段 UC。

**中文注释段**：`0.75::artist:name::0::油画厚涂感::` —— 值 0 的中文段不参与采样，只作人类可读备注；给协作者或自己复盘时很好用。

**不认的语法**：`(tag:1.2)`、`((tag))`、`[[tag]]`。这些是 SD / Pony / 光辉系的括号族语法，NAI 会把它们当普通文字吃掉；从那边迁移过来的串必须整体改写，否则画风会完全变掉（群里实测：同一位画师的 `(画师:0.8)` 无效、改成 `0.8::artist:画师::` 才生效）。

**这几条也只属于 NAI**：`{}`、`[]`、`N::tag::`、`artist:`、`source#/target#/mutual#`。搬到光辉、Anima、MOSS 里全部无效，反向也不成立——两套语法硬互斥，不存在通吃的写法。

---

## 七、漫画分镜逻辑

V5 可以直接出一整页多格漫画，不必切图拼接。

**四步搭法：**

1. **首句声明版式**：`A four-panel manga page in a 2x2 grid, white panel borders, read left-to-right, top-to-bottom, no text outside the panels.`
2. **逐格锚定**：`Top-left:`、`Top-right:`、`Bottom-left:`、`Bottom-right:` 或 `Panel 1:`。每格独立写景别、人物、动作、底色，**格子之间不共享句子**。
3. **角色跨格复用**：外观全写进 Character 栏，格子内只写动作与表情，避免每格重画一遍脸。
4. **台词**：`text"要出现的台词"`，中文可直接写；气泡归属写清是谁在说，否则会贴错人。

**互斥原则**：一格里只留一个景别、一个视角；跨格才换机位。分格多、内容密时精简画风词，否则整页会糊。

### 7.1 `Text:` 与台词（按 UI 实际行为写）

社区有人读了 85 张 V5 成图的元数据（其中 63 张漫画），把 UI 真正发出去的东西还原了出来，结论比猜测可靠：

| 事实 | 做法 |
| --- | --- |
| 引号里的文字会被 UI 自动收进一个 `teXt:` 块，追加在 Base Prompt 末尾；顺序按角色框排列 | 想让台词落在指定格，就在那一格写引号，别指望位置靠语义 |
| 手动 `Text:` 必须是该 prompt 的**最后一段**，其后所有内容都按字面渲染 | 先写完画面，再写 `Text:`；`Text:` 之后不要再跟任何标签 |
| `Text:` 前必须有空格、逗号或换行 | 写成 `high complexityText:` 会让质量词插进字里 |
| 每次生成字数上限：Curated 374 字符、Full 750 字符 | 长台词拆到多格，别硬塞一格 |
| 质量词开关会在 `Text:` 之前插入 `very aesthetic, masterpiece, no text` | 短字不出来时先关开关 |
| 一个 `Text:` 里换行 = 多个气泡；被插件改成逗号会把两行并成一个气泡 | 多气泡用换行，不用逗号 |
| `||A|B|C||` 随机分支里的引号会被全部收集 | 随机台词写成 `||" hey". Text: hey|" yo". Text: yo||` |

### 7.2 角色框与坐标

- V5 最多 **22 个角色框**，每框有自己的 UC。
- 位置是 **0–1 的自由浮点**（`x` 左→右、`y` 上→下），不是旧版 5×5 网格；不开 AI's Choice 时才生效。`y` 越小越靠上、人物越小，越大越近。

### 7.3 漫画页实际参数（社区成图统计）

| 项 | 实际取值 |
| --- | --- |
| 尺寸 | 832×1216 最多，其次 1024×1536、1024×1024（2×2）、1216×832；竖条用 896×2304 / 1088×1920 |
| 步数 | 23 或 28（少量 18–19） |
| Guidance | 5–7.5 |
| CFG rescale | 0–0.4 |
| 采样器 | `k_euler_ancestral` 为主，其次 `k_dpmpp_2m_sde`、`k_dpmpp_2m` |
| 噪声 | karras |

```text
A four-panel manga page in a 2x2 grid, white panel borders, read left-to-right, top-to-bottom,
2girls, school rooftop, sunset,
Panel 1: wide shot, both girls standing apart, wind, silent,
Panel 2: close-up on girl A, surprised expression, speech bubble reading "你刚才说什么？",
Panel 3: close-up on girl B, looking away, blushing, speech bubble reading "没什么。",
Panel 4: full shot, back view, both walking away in opposite directions, long shadows,
very aesthetic, masterpiece,
Text: 你刚才说什么？

没什么。
```

---

## 八、负面基线（已过滤 NSFW）

**必需层（V5 Heavy 预设，原样用）**

```text
lowres, artistic error, film grain, scan artifacts, worst quality, bad quality, jpeg artifacts,
very displeasing, chromatic aberration, dithering, halftone, screentone, multiple views, logo,
too many watermarks, negative space, blank page
```

**按需层（人像 / 多手 / 单人图）**

```text
bad anatomy, bad hands, extra fingers, fewer fingers, fused fingers, missing fingers,
bad proportions, extra limbs, deformed, cropped, blurry, out of focus, signature, text
```

**可选层（按画面选）**

| 加什么 | 什么时候 | 什么时候别加 |
| --- | --- | --- |
| `monochrome, greyscale` | 要彩色 | 要黑白稿 |
| `halftone, screentone` | 要干净画面 | 要漫画网点 |
| `multiple views` | 要单视图 | 要分格/多视图（官方预设里带着，得换成 Light 或 None） |
| `logo` | 画面无字 | 要画招牌/标题 |

**点杀优先用负权重，不要堆 UC**：`-2::extra characters::`、`-3::multiple views::`、`-1::flat color::`、`-2::text, watermark::`。

**已过滤 NSFW 项**：原始语料的 UC 与群友负面里出现过 `nsfw`、`loli`、`child`、`cleavage`、`nipples`、`nude` 一类词条，已全部移除，替换为 `bad proportions`、`distorted face`、`unnatural body` 等解剖/质量项。本 skill 的负面基线不含任何 NSFW 词条；分级由正向的评级标签表达，不靠负面。

**图里要画字时**：关掉质量词开关（它带的 `no text` 会让字出不来），并把 UC 预设从 Heavy/Human Focus 换成 Light 或 None（前者带 `logo`、`multiple views`）。

---

## 九、自检清单（交付前逐条过）

1. 版本确认过，质量词与开关状态对得上（开关开着就不再手写）。
2. 三栏分工正确：外貌只在 Character 栏，动作/镜头/场景只在 Base。
3. 人数只写一次，Character 栏不带数字。
4. 画师全带 `artist:`，数量 2–6，无 `@`，无 `(tag:1.2)` 之类 SD 语法。
5. 多角色：每人有站位锚点与外貌锚点，互动带 `source#` / `target#` / `mutual#`。
6. 权重括号闭合，或用 `::` 收尾；没有悬空闭合符。
7. 负面与画面需求不冲突（黑白稿别否 `monochrome`，要画字别留 `logo`/`no text`）。
8. 负面里没有任何 NSFW 词条。
9. 画了字：`Text:` 是整个 prompt 的最后一段，它后面没有标签，且字数没超过 Curated 374 / Full 750。

---

## 十、完整示例

### 示例 1｜单人 · 窗边逆光

```text
Base Prompt:
1girl, solo, 1.2::artist:ciloranko::, 0.8::artist:yoneyama mai::, year 2025,
long black hair, low ponytail, hair over shoulder, amber eyes, pale skin,
white camisole, thin strap slipped, oversized cardigan, bare feet,
sitting on the windowsill, knee up, one hand on the frame, looking outside,
upper body, three-quarter view, from side,
traditional room, paper window, afternoon light, strong backlight, rim light on hair,
dust in the air, warm and quiet,
1.3::cinematic lighting, volumetric light::, very aesthetic, masterpiece, no text

Character 1:
girl, long black hair, amber eyes, white camisole, oversized cardigan, bare feet, calm expression

Undesired Content:
lowres, artistic error, film grain, scan artifacts, worst quality, bad quality, jpeg artifacts,
very displeasing, chromatic aberration, dithering, halftone, screentone, multiple views, logo,
too many watermarks, negative space, blank page, bad anatomy, bad hands, extra fingers, (artist collaboration:-2)
```

### 示例 2｜双人 · 互动前缀 + 站位

```text
Base Prompt:
2girls, from side, medium shot, upper body, Character 1 and Character 2 mutual#holding hands,
snowy street at dusk, tram tracks, warm shop light from the left, deep blue twilight,
1.2::artist:name::, 0.8::artist:name::, -3::artist collaboration::, very aesthetic, masterpiece, no text

Character 1: girl, tall, black wool coat, tilted umbrella, looking down, soft smile
Character 2: girl, petite, oversized white scarf, looking up, flustered blush

Undesired Content:
lowres, artistic error, scan artifacts, worst quality, bad quality, jpeg artifacts, very displeasing,
multiple views, logo, bad anatomy, bad hands, extra fingers, extra characters
```

### 示例 3｜四格漫画

```text
Base Prompt:
A four-panel manga page in a 2x2 grid, white panel borders, read left-to-right, top-to-bottom,
2girls, classroom, afternoon,
Panel 1: wide shot, girl A at the window, girl B at the desk, empty classroom,
Panel 2: close-up, girl A turning around, surprised, text"还没走？",
Panel 3: close-up, girl B looking down, small smile, text"等你。",
Panel 4: full shot, two silhouettes walking out together, long shadows on the floor,
1.1::artist:name::, high complexity, very aesthetic, masterpiece

Character 1: girl, long black hair, school uniform, ribbon tie
Character 2: girl, short brown hair, school uniform, cardigan

Undesired Content:
lowres, worst quality, bad quality, jpeg artifacts, very displeasing, bad anatomy, bad hands,
extra fingers, deformed, dithering, too many watermarks
```

---

## 十一、常见错误

| 错误 | 后果 |
| --- | --- |
| 用 SD 的 `(tag:1.2)` | NAI 不认，原样当文字吃掉 |
| 开质量词开关又手写质量词 | 双份质量词，画面发糊发腻 |
| 画师不带 `artist:` | 被当普通标签，风格不生效 |
| 外貌在 Base 与 Character 栏各写一遍 | 权重被重复叠加，脸会变形 |
| 多人不写站位 | 串脸、换装、多出人头 |
| 互动不写 `source#`/`target#` | 动作方向反了 |
| 分格之间共享句子 | 内容串格，一格糊两格 |
| 要画字却留着 `no text` 与 Heavy UC | 字出不来 |
| `Text:` 后面又跟标签 | 标签被当文字印在画面上 |
| 一个 `Text:` 里用逗号分行 | 多句并成一个气泡 |
| 把 `{}` / `N::` / `artist:` 搬到光辉或 MOSS | 语法不存在，整段变死字 |

---

## 十二、来源

- 官方文档：https://docs.novelai.net/en/image/basics/
- 权重：https://docs.novelai.net/en/image/strengthening-weakening/
- UC：https://docs.novelai.net/en/image/undesiredcontent/
- 质量标签：https://docs.novelai.net/en/image/qualitytags/
- 步数与引导：https://docs.novelai.net/en/image/stepsguidance/
- 多角色与互动前缀：https://github.com/moudanji/NAI5-SKILL
- 11 个拆分技能（写法 / 分镜 / 反推）：https://github.com/Miint-Sunny/nai5-prompting
- 画师串与质量词条：https://nai4.top/ArtistStyle/
- V5 能力与 Bot 参数：https://nai4.top/nai3bot指南/绘画nai5模型/
- 群聊语料拆解：`资料文件夹/群聊语料拆解报告.md`
