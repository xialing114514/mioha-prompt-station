# 光辉系 · Illustrious XL 提示词 SKILL v5

> 适用：Illustrious XL / 其衍生 merge（NoobAI 等按各自模型卡微调参数）。面向 SDXL 架构、Danbooru2023 语料训练的标签型模型。
> 本文件由 v4 `Illustrious.md` 升级而来，新增画师串管理模块、权重语法边界、负面三层结构。负面已过滤 NSFW 项。
> **权重语法只有一套：括号族 `(tag:1.2)`。`{}`、`[]` 之外的 NAI 写法（`N::tag::`、`artist:`）在这里全部无效。**

## v5.1 修正（2026-11-12）

v5 初版第六节把 NAI 的 `1.2::artist:name::` 列为「跨前端可选写法」，这是错的：**Illustrious 本体不支持 `N::tag::`，也不支持 `{}`**。群聊里那些带 `1.45::artist:x::` 的串全是 NAI 语境，搬到光辉里只会被当文字编码。本次已改写 §6.2、§6.3、§十、§八，并补「不要照抄 NAI 画师串」的改写示例。

---

## 一、模型参数默认值

| 参数 | 默认 | 区间 | 说明 |
| --- | --- | --- | --- |
| 采样器 | Euler a | Euler a / DPM++ 2M Karras | 官方推荐 Euler a；要更平滑换 DPM++ 2M |
| 步数 | 24 | 20–28（1536px 以上 30+） | 超过 30 收益很小，只会拖时间 |
| CFG | 5.5 | 5.0–7.5 | <5 发灰，>8 过饱和发脆 |
| 调度器 | normal | normal / karras（1536px+） | karras 在高分辨率下更稳 |
| 分辨率 | 832×1216 / 1216×832 | 1024²、1344×768、1536×1024 | SDXL 原生 1024² 基准 |
| Hires.fix | 关 | 需要时 denoise 0.35–0.5 | 二次元细节多时 0.4 起步 |
| CLIP skip | 2 | 2 | Illustrious 系惯用 |

**衍生 merge 不吃这一套默认值**，按各自模型卡来：

| 衍生 | 官方 / 社区给的值 |
| --- | --- |
| Illustrious Realism | DPM++ 2M Karras 或 DPM++ SDE Karras，CFG 5–6.5，25–35 步，clip skip 1 |
| NoobAI v-pred 测试 merge | Euler a + SGM Uniform，CFG 1.0–1.5，8 步起（4–6 步也能用） |
| Illustrious Gehenna | Euler a，20–30 步，CFG 5–7 |

把 v0.1 的 CFG 5.5 套到 v-pred merge 上，画面会直接糊。换底模先看模型卡。

---

## 二、输出契约

| 字段 | 内容 |
| --- | --- |
| `finish.positive` | Danbooru 标签数组，逗号分隔，全小写 |
| `finish.negative` | 负向标签数组，按「负面基线」三层取用 |
| `finish.texts` | 画面文字（SDXL 文字渲染弱，默认留空） |

交付形态：一个 `text` 代码块装正向，一个装负向，块外不写字。要给参数就另起一行写「参数：」。

---

## 三、正向槽位模板

按顺序铺，权重天然从前到后递减。**粗体为必填**。

| # | 槽位 | 内容 | 可省略 |
| --- | --- | --- | --- |
| 1 | **质量** | `best quality, masterpiece`（或走「质量预设」三档之一） | 否 |
| 2 | **人数** | `1girl, solo` / `2girls` / `no humans` | 否 |
| 3 | 身份 | `角色名 (作品名)`；原创写 `英文名-original` | 是（无角色时） |
| 4 | 外观 | 发长、发色、瞳色、体型、特征（角/耳/尾/疤） | 否 |
| 5 | 服装 | 款式 + 颜色 + 材质 + 穿着状态 | 否 |
| 6 | 动作 | 姿势 + 接触点 + 道具 | 否 |
| 7 | 表情 / 视线 | 情绪 + 眼 + 嘴 + `looking at viewer` | 否 |
| 8 | 镜头 / 构图 | 景别 + 视角 + 透视（各只留一个） | 否 |
| 9 | 场景 / 光 | 地点 + 时间 + 光源方向 + 色调 | 推荐 |
| 10 | 风格 / 画师串 | 画师块 + 媒介词 + 质量补充 | 推荐 |
| 11 | 画面文字 | `text` / `signature` 相关，或留空 | 是 |

一句话原则：**先定这是什么图，再定谁在画面里，最后定这一刻的光。**

---

## 四、Danbooru 标签结构

**四类标签，作用不同，不要混写：**

| 类 | 例 | 用法 |
| --- | --- | --- |
| general | `long hair`、`pleated skirt`、`from below` | 描述画面，占总量 80% |
| character | `frieren \(sousou no frieren\)` | 只写角色名 + 作品名，外观另写；括号要转义 |
| copyright | `sousou no frieren` | 与 character 成对出现（作者名里的括号同样转义） |
| artist | `wlop`、`ask \(askzy\)` | 纯名字，括号转义，不加 `@`、不加 `artist:` |

**写法纪律：**

- 全小写，下划线换空格（`long hair` 优于 `long_hair`）。各家前端对下划线的处理不一致，直接写空格最稳。
- 作品名、画师名里的圆括号**必须转义**：`frieren \(sousou no frieren\)`、`ask \(askzy\)`。不转义的话，A1111 族会把 `(sousou no frieren)` 当成权重语法，角色名被吃掉一半。
- 数量 25–60 个。低于 15 个画面空，高于 70 个开始互相稀释。
- 逗号后留一个空格：CLIP 的分词器有 `", "` 这类合并 token，`a,b` 与 `a, b` 切出来的序列不同，后者更贴训练分布。
- 同义不并列：`beautiful` + `pretty` + `gorgeous` 只留一个。
- 描述「关系」用短语而非句子；Illustrious 的自然语言能力弱于同代 SDXL 变体，长句会丢后半段。

**构图标签互斥表（官方明确警告）：**

| 类别 | 只能选一个 |
| --- | --- |
| 景别 | `portrait` / `upper body` / `cowboy shot` / `full body` |
| 视角 | `from above` / `from below` / `from behind` / `eye-level` |
| 透视 | `dutch angle` / `fisheye` / `vanishing point` / `wide-angle` |

选一个主构图，其余靠补充描述；一次出现两个景别，模型会自己裁，画面不可控。

**安全层**：只写评级标签（如 `rating:general`），写在正向、不写进负向；不要用裸 `safe` 查词（Danbooru 有 `safe (container)` 保险箱）。

---

## 五、质量标签预设

官方支持这一套：`worst quality, bad quality, average quality, good quality, best quality, masterpiece`。

| 档 | 正向 | 适用 |
| --- | --- | --- |
| 轻 | `good quality, best quality` | 要自然、不要塑料感 |
| 标准 | `best quality, masterpiece` | 默认 |
| 高完成度 | `best quality, masterpiece, very detailed` | 封面 / 主视觉 |

三档选一。堆到 5 个以上质量词，只会互相稀释并让画面发腻。`masterpiece (quality)` 与 `masterpiece` 等价，二选一。

---

## 六、画师串管理模块（本 skill 核心）

### 6.1 四槽位拆法

把画师串当配方管理，不要一锅乱炖：

| 槽 | 作用 | 数量 | 权重带 |
| --- | --- | --- | --- |
| A 主风格 | 定整张图的笔触与色彩底子 | 1 | 1.0–1.4 |
| B 副风格 | 补主风格缺的那一半（主风格偏冷就补暖） | 1–2 | 0.6–0.9 |
| C 质感 | 线条、厚涂、水彩、网点等材质倾向 | 1–2 | 0.4–0.7 |
| D 排除 | 负权重踢掉不想要的笔触 | 0–2 | −1 ~ −3 |

合计 3–6 位是稳定区。超过 8 位互相打架，画面会变成拼贴。

### 6.2 权重写法：光辉只认一套

**Illustrious 本体（A1111 / Forge / ComfyUI / PixAI）只认括号语法。** 下面这张表是全部可用的写法，其余一律无效：

| 写法 | 语义 | 生效 |
| --- | --- | --- |
| `(artist name:1.2)` | 精确权重，1.2 倍 | 是 |
| `(artist name)` | 默认 ×1.1 | 是 |
| `((artist name))` | 嵌套 ×1.21（每层 ×1.1） | 是 |
| `[artist name]` | ÷1.1，约 0.91 倍 | 是 |
| `(artist name:0.8)` | 降到 0.8 倍（推荐用这个，比 `[]` 好控制） | 是 |

**以下写法在光辉里一律无效，写了等于没写：**

| 写法 | 实际结果 |
| --- | --- |
| `{artist name}` / `{{...}}` | 大括号不是语法，被当普通文字编码，画风不动 |
| `1.2::artist:name::` | 整段被当文字吃掉，`1.2`、`::`、`artist:` 全进编码器，只添噪声 |
| `artist:名字` | `artist:` 不是前缀，写进去只会稀释权重（见 6.3） |
| `[[artist name]]` | 反向叠括号，效果与单个 `[]` 相同，白写 |

**不要照抄 NAI 的画师串。** 群里流传的 `1.45::artist:ciloranko::`、`[artist:sho_(sho_lwlw)]`、`{artist:hokori sakuni}` 全是 NAI 语法，搬到光辉里一个都不生效——那些括号会被当文字编码，画风与不写画师时几乎一样。要在光辉里复现同一个画风，必须改写成 `(ciloranko:1.45)`、`(sho (sho lwlw):0.9)`、`(hokori sakuni:1.1)`。

反过来同理：NAI 里写 `(artist:x:1.2)` 也会失效（NAI 认 `1.2::artist:x::` 与 `{x}`）。**两边语法是硬互斥的，不存在一套通吃的画师串。**

### 6.3 排列顺序

- 画师块**整块连续**，不要被角色词或场景词切断。
- 位置：角色与外观之后、质量词之前（群里样本最多的位置）。
- 探索期可以前置（先定画风再想内容），定稿后挪回角色之后，便于复现。
- 画师名写**纯名**，括号转义：`(ciloranko:1.2)`、`(ask \(askzy\):1.0)`。不加 `@`，不加 `artist:`。

### 6.4 四个可直接用的稳定块

```text
# 稳定块 1：清透水感（不挑题材）
(artist name:1.3), (artist name:0.9), (artist name:0.8), (artist name:0.6)

# 稳定块 2：厚涂高细节（主视觉）
(artist name:1.2), (artist name:0.7), (thick paint:0.6), (visible brushstrokes:0.5)

# 稳定块 3：压低副画师（防拼贴）
(artist name:1.1), (artist name:0.5), (artist name:0.3)
# 配套负面：artist collaboration, multiple views

# 稳定块 4：平涂被带厚涂时（压不想要的方向，靠负面，不靠负权重）
(artist name:1.0), (clean lineart:0.8)
# 配套负面：flat color, cel shading
```

**注意**：正向里没有负权重。`(tag:-1.5)` 是 NAI 的写法，光辉系前端不解析，写了只是一串字符。要压某个方向，把它写进负向（`flat color, cel shading`），或在正向里用低权重形容词替换（`clean lineart:0.8`）。

风格描述词作为画师名的替代/补充（模型没收录的画师，用这些顶上）：

| 想要 | 写 |
| --- | --- |
| 水彩 | `watercolor (medium), wet on wet, paper texture` |
| 厚涂 | `impasto, thick paint, visible brushstrokes` |
| 赛璐璐 | `cel shading, flat color, clean lineart` |
| 复古 | `retro artstyle, 1990s (style), grain` |
| 素描 | `sketch, graphite (medium), monochrome` |
| 官方图 | `official art, promotional art, key visual` |

### 6.5 画师串失效排查

| 现象 | 先查 |
| --- | --- |
| 画风完全没变 | 语法是否被前端吃掉（6.2） |
| 画面变成拼贴 | 负面里是否漏了 `artist collaboration`，或画师超过 8 位 |
| 手崩、脸崩 | 是否用了笔触极端细碎的画师（把它的权重压到 0.4–0.6，或删掉） |
| 光位不听话 | 画师串自带强光位，先删串再调光照词 |
| 颜色发脏 | 副风格与主风格色系冲突，砍掉 B 槽 |

---

## 七、负面基线（已过滤 NSFW）

三层取用，不要全塞。

**必需层（永远写）**

```text
worst quality, low quality, lowres, bad anatomy, bad hands, extra digits, fewer digits, jpeg artifacts, displeasing, very displeasing
```

**按需层（人像/多手/多视图时加）**

```text
bad proportions, poorly drawn hands, fused fingers, missing fingers, malformed limbs, extra limbs, deformed, cropped, out of focus, scan artifacts, signature, username, watermark, logo, text, multiple views
```

**可选层（按画面选，选错会反噬）**

| 加什么 | 什么时候 | 什么时候别加 |
| --- | --- | --- |
| `monochrome, greyscale` | 要彩色 | 要黑白稿 |
| `comic, 2koma, 4koma` | 要单幅 | 要漫画分格 |
| `3d, realistic, photo` | 要纯二次元 | 要半写实 |
| `sketch, lineart` | 要成品图 | 要线稿 |
| `artist collaboration` | 要统一画风 | 本来就要多人画风混合 |

**已过滤 NSFW 项**：原始语料里的 `nsfw`、`loli`、`child`、`cleavage`、`nipples`、`nude` 等条目，全部替换为等价的解剖/质量项（`bad proportions`、`distorted face`、`unnatural body`），本 skill 的负面基线不含任何 NSFW 词条。

负面只排除「缺陷」与「用户明确不要的东西」，不要否定用户点名的画风；写 80 个以上的负面词会互相抵消。

---

## 八、自检清单（交付前逐条过）

1. 人数标签与实际人物数一致；有角色名时带作品名。
2. 景别、视角、透视各只有一个，没有互相打架。
3. 画师是纯名，没有 `@`、没有 `artist:` 前缀。
4. 画师串整块连续，数量 3–6 位，全部是 `(名字:权重)` 括号语法，没有 `{}`、没有 `N::tag::`。
5. 质量词只有一档，没有堆 5 个以上。
6. 负面分层取用，没有混入 NSFW 词条，没有否定用户点名风格。
7. 画面文字需求为空时，`finish.texts` 留空，且没有把 `text` 写进正向。
8. 没有用 Pony 的 `score_9` 系列，也没有用 Anima 的 `score_N`。

---

## 九、完整示例

### 示例 1｜雨夜便利店（单人 · 半身）

正向：

```text
best quality, masterpiece, 1girl, solo, short black hair, blunt bangs, amber eyes,
convenience store uniform, striped shirt, name tag, apron, sleeves rolled up,
standing behind counter, one hand on the register, looking at viewer, tired smile,
upper body, eye-level, slightly off-center,
convenience store interior at night, fluorescent light from above, cold green-white cast,
rain streaks on the glass door behind her, blurred street lights outside,
(artist name:1.2), (artist name:0.7), (clean lineart:0.6), (cinematic lighting:0.5)
```

负向：

```text
worst quality, low quality, lowres, bad anatomy, bad hands, extra digits, fewer digits,
jpeg artifacts, displeasing, very displeasing, bad proportions, fused fingers, watermark,
signature, logo, multiple views, artist collaboration
```

### 示例 2｜天台逆光（单人 · 全身 · 强画风）

正向：

```text
masterpiece, best quality, 1girl, solo, long silver hair, hair blowing, red eyes,
white sailor dress, black ribbon, thighhighs, loose socks,
standing on rooftop, hand on railing, turning head, looking back, calm expression,
full body, from below, dutch angle,
rooftop at sunset, chain-link fence, water tank, city skyline, strong backlight, rim light,
orange haze, lens flare, film grain,
(artist name:1.3), (artist name:0.8), (artist name:0.6), (painterly:0.5)
```

负向：

```text
worst quality, low quality, lowres, bad anatomy, bad hands, extra digits, fewer digits,
jpeg artifacts, displeasing, very displeasing, bad proportions, cropped, out of focus,
watermark, signature, text, multiple views, monochrome, greyscale
```

### 示例 3｜双人对话（2 人 · 分区）

正向：

```text
best quality, masterpiece, 2girls, upper body, eye-level, from side,
girl A on the left, long black hair, glasses, cardigan, holding a book, talking,
girl B on the right, short blonde hair, hoodie, listening, hand on chin, smiling,
classroom after school, window light from the left, dust in the air, warm afternoon,
(artist name:1.1), (artist name:0.7), (soft shading:0.6)
```

负向：

```text
worst quality, low quality, lowres, bad anatomy, bad hands, extra digits, fewer digits,
jpeg artifacts, displeasing, very displeasing, bad proportions, fused fingers, extra limbs,
watermark, signature, logo, multiple views, artist collaboration
```

---

## 十、常见错误

| 错误 | 后果 |
| --- | --- |
| 不写画风/画师 | 素底图，用户以为模型坏了 |
| 画师加 `@` 或 `artist:` | 被当普通标签，风格不生效 |
| 照抄 NAI 的 `{}` / `N::tag::` 画师串 | 语法不存在，整段被当文字编码，画风不动 |
| 构图标签互相打架 | 画面随机裁切，人物缺手缺脚 |
| 用 Pony 评分词 | 无效，白占 token |
| 负面写 100 个词 | 关键词互相稀释 |
| 质量词堆 6 个 | 画面发腻、细节糊 |
| 把 `text` 写进正向又填文字 | 文字被压掉 |

---

## 十一、来源

- 官方模型卡：https://huggingface.co/OnomaAIResearch/Illustrious-xl-early-release-v0
- 技术报告：https://arxiv.org/abs/2409.19946
- 社区参数与工作流：https://github.com/regiellis/ComfyUI-EasyIllustrious
- 自然语言转正负词节点包：https://github.com/ComfyNodePRs/PR-deepseek-comfyui-Illustrious-prompt-plugin-ed25d440
- 画师串与质量词条：https://nai4.top/ArtistStyle/
- 语法归属旁证（把 Illustrious 归入括号族，与 NAI 分列）：https://github.com/JDJoe/Krea2PromptConverter
- 衍生 merge 参数：Civitai 模型卡（Illustrious Realism / NoobAI v-pred merge / Gehenna），本地存档 `资料文件夹/community/civitai/models_illustrious.json`
- 群聊语料拆解：`资料文件夹/群聊语料拆解报告.md`
