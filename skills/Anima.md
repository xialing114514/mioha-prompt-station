# Anima · Base 提示词规范（2B DiT）

## 模型定位

- CircleStone Labs × Comfy Org：2B DiT + Qwen3-0.6B 文本编码器（Qwen-Image VAE）。
- 训练数据：数百万动漫图 + 约 80 万非动漫艺术图，知识截止 2025-09。
- 强项：Danbooru 标签、自然语言、两者混排；角色还原与画风跟随。
- 弱项：写实（设计上就避开）、长文本渲染（能出单词，短语容易崩）。

| 版本 | 质量词 | score 系列 | 参数 |
| --- | --- | --- | --- |
| **Base**（当前） | `masterpiece, best quality` | `score_7` 起，可到 `score_9` | 30–50 步，CFG 4–5 |
| Aesthetic | 可省 | 正负都不要 | 同上 |
| Turbo | 不需要 | 不需要 | CFG 1，8–12 步 |

## 七层总结构

按顺序铺，权重从前到后递减：

| 层 | 内容 | 必填 |
| --- | --- | --- |
| 1 质量 | `masterpiece, best quality, score_7` | 是 |
| 2 元信息 / 时间 | `highres, absurdres, anime key visual` + `year 2025, newest` | 推荐 |
| 3 安全 | `safe` / `sensitive` / `nsfw` / `explicit` | 是 |
| 4 主体 | `1girl` `2girls` `solo` `no humans` | 是 |
| 5 身份 | 角色名 + 作品名（同人）或 `英文名-original`（原创） | 有角色时 |
| 6 细节 | 外观、服装、动作、表情、镜头、构图 | 是 |
| 7 剧情 | 一句完整英文，写动作、道具、冲突、观众关系 | 是 |

一句话原则：先说明这是什么图，再说明谁在画面里，最后说明这一刻发生了什么。

## 各层写法

**质量层**：`masterpiece, best quality, score_7`。要更强完成度加 `score_8` / `score_9`，要更自然只留前两个。

**元信息层**：类型词（`highres, official art, anime screenshot, absurdres, anime illustration, anime cover art, anime key visual`）+ 年份（`year 2025`）+ 时代（`newest, recent, mid, early, old`）。

**安全层**：正常封面写 `safe`；含性器官或性行为写 `nsfw` / `explicit`；边界不清按 `safe` 处理。**安全词原样写，不查词。**

**主体层**：人数必须明说。`solo` 与 `1boy` 同时出现算矛盾。

**身份层**：同人写 `fujiwara chika, kaguya-sama wa kokurasetai`。作品名的圆括号要转义成 `\(...\)`，否则会被当成权重语法吃掉。原创角色写 `英文名-original`。每个角色都要补关键外观，只列名字会让模型串人。

**细节层**：发长、发色、瞳色、罩杯是核心四件；再补服装、动作、表情、镜头、构图。

**剧情层（必填）**：一句完整英文，把「做什么、对谁做、什么道具、什么冲突、观众关系」说清。

## 剧情感的两种写法

**普通场景**，一句话交代关系：

```
masterpiece, best quality, score_7, highres, safe, 1girl, long hair, direct eye contact, close-up, upper body, anime illustration, A young woman extends a sealed letter toward the viewer while rain blows through the ruined station behind her, her gaze fixed on the camera as if the final decision belongs to the person watching.
```

**主视觉 / 封面向**，五步搭：

1. 身份：逃婚新娘、冰封公主、赌场诈欺师、时间旅者
2. 当下动作：递出戒指盒、按下时间装置、递出判决书
3. 冲突背景：教堂外暴雨、冰宫封印破裂、法庭血月
4. 观众关系：像在邀请一起逃、像在审判观众、像把希望交出去
5. 风格词：`anime key visual, story opening, explosive composition, ultra detailed`

```
..., A runaway bride holds a ring box toward the viewer in the foreground while rain hammers the cathedral doors behind her, her body leaning forward and her direct gaze making it feel as if she is asking the person watching to run with her.
```

避免 `beautiful, dreamy, cinematic` 这类空形容词。写清动作、道具、冲突、关系。

## 纯自然语言模式

用户要整段描述、明确不要标签时，按三段长句输出，句号分隔：

```
[角色身份与外貌长句]. [动作与服饰长句]. [环境、光影与氛围长句].
```

例：

```
A pale girl with glossy cream blonde low twintails, blunt bangs and a single ahoge, dark cyan eyes and round cheeks. She sits on the shrine steps with her knees together, one hand holding a candy apple, wearing a white sailor top and a dark red pleated skirt that catches the evening light. Red torii and a donation box stand behind her, sunset rim light edging her hair while long shadows climb the stone steps.
```

三段分工：第一段交代是谁、长什么样；第二段交代在做什么、穿什么；第三段交代在哪、什么光、什么气氛。要标签+句子混排时用前面的混写原则，走七层结构。

## 三段式 TAG 构成

完整 prompt 按 Scene → Char[X] → Background 三段排。

**Scene（场景层）**

| 要素 | 写法 |
| --- | --- |
| 通用 | 画师 `@名字`，放本段靠前 |
| 分级 | `sfw` / `nsfw`，按画面可见内容判定 |
| 场景类型 | `exhibitionism` 等，可选 |
| 角色关系 | `hetero`、`duo` / `trio` / `group` |
| 相对位置 | `in center`、`left side`、`side by side`、`surrounding` |
| 视角构图 | 视角 / 区域 / 远近 / 透视 / 焦点 / 角度 |

**Char[X]（角色层）**

- 主角详述，配角简化。
- 位置用逗号直接连：`in center, 1girl`。每个角色给 1 个中心坐标，拥抱、接吻这类可以重叠。
- 配角 ≤2 人各自独立描述；超过 2 人合并：`around, 4boys`。
- 种族判定按人形度：≥60% 用 `girl` / `boy`（人类、亚人、精灵、兽耳、天使、魅魔、人鱼）；50–60% 看脸，人脸保留用 `girl` / `boy`，兽化脸、多肢、异形躯干用 `no humans`；<50% 用 `no humans`（纯兽、虫、龙、触手、机械）。
- 无角色时道具或建筑占一个 Char 槽。

外貌与服装字段：

| 组 | 内容 |
| --- | --- |
| 外貌核心 | 发长、发色、瞳色、罩杯 |
| 外貌修饰 | 性格、彩妆、瑕疵、纹身、晒痕、瞳孔、非人特征 |
| 服装主要 | 款式 + 颜色 + 细节（材质、形状、图案、开口） |
| 服装次要 | 款式 + 颜色 |
| 穿着状态 | 穿（跳过）、脱、敞开、半褪、掀起 |
| 衣物损耗 | 破损、撕裂、磨损、沾污、湿透 |
| 裸露梯度 | 掀起 / 半脱 → 无上装、拉上衣 → 无下装、仅丝袜 → 全裸 → `visible through`、`see-through`、`covered` |
| 装饰 | 项链、吊袜带、珠宝、乳贴 |
| 露肤点 | 肩、脐、背、腿、乳沟、侧乳、下乳 |
| 非衣当衣 | 丝带、绷带、创口贴 |
| 少女风 | 雪纺、薄纱、蕾丝、过膝袜、泡泡袖、褶裥 |
| 熟女风 | 深 V、开衩、镂空、紧身、乳胶 |

动作字段：

| 类型 | 结构 | 例 |
| --- | --- | --- |
| 基础 | 姿势 + 空间位置 + 肢体细节 | `sitting, on stone steps, leaning back` |
| 无对象 | 肢体 + 动作 + 细节 | `one hand, arm up, peace hand` |
| 有对象（肢体） | 肢体 + 动作 + 落点 | `one hand, covering chest, hand on own chest` |
| 有对象（物品） | 肢体 + 动作 + 物品 + 物品状态 + 位置 | `one hand, holding a staff, hand on staff, black magic staff, red gem, glowing gem` |
| 视线 | `looking at [x]`、`facing [x]` | `looking at another` |

表情字段：情绪（`happy`）、神态（笑 / 怒 / 哀 / 惊 / 慌 / 羞 / 累 / 反感）、眼（`tears`、`wide-eyed`）、嘴型（`smile`、`open mouth`）。

状态字段（按剧情主动加）：体表附着（`sweat`、`mud`、`stain`、`cum`、`saliva`）、身体损伤（`bruise`、`wound`、`bleeding`）、生理（`exhausted`、`weak`、`hungry`）。

剧情句放 Char 块末尾。

**Background（背景层）**

| 组 | 写法 |
| --- | --- |
| 空间 | 室内 / 室外 + 地点 + 周边事物 |
| 氛围 | 时间、天气、季节、节日、时代 |
| 光源 | `sun`、`ceiling light`、`warm lighting`（光源本身不在画面里） |
| 逆光 | `backlighting`、`rim lighting` |
| 侧光 | `sidelighting`、`dramatic shadows` |
| 顶光 / 顺光 | `toplighting`、`cast shadows` |

## 视角与可见性

**视角判定**

- 第一人称：`pov`（相机即观众视角，角色默认不出镜）
- 第三人称：`third-person view`（单方出镜、对视、无交互、镜面、自拍、对话），有互动时补 `duo` / `trio` / `group`
- 判定不清默认第三人称

**构图限制**

| 构图 | 禁写 |
| --- | --- |
| `upper body` | 膝、脚、腿以下 |
| `lower body` | 脸、瞳色、表情 |
| `from behind` / `back view` | 正面部位、脸、瞳色、表情（回头除外） |
| 遮挡（门后、遮挡物） | 被挡住的身体部位、服装、配饰 |
| 蒙眼 / 闭眼 | 瞳色、瞳孔 |
| 穿戴整齐 | 内衣、被覆盖的部位 |

**人数与景别**

| 人数 | 可用景别 | 禁用 |
| --- | --- | --- |
| 1–2 | 任意 | — |
| 3 | `cowboy shot`、`threesome` | `close-up` |
| 4+ | `full body`、`wide shot`、`group picture` | `close-up`、`cowboy shot` |

连续生图要主动换视角：POV / 第三人称、全身 / 半身、俯 / 仰 / 侧、远近、光线方向。

## 多角色

结构：人数 → 角色标识 → 每人关键外观 → 站位关系 → 动作剧情。

```
masterpiece, best quality, score_7, highres, safe, 2girls, black-haired woman with amber eyes, silver-haired woman with blue eyes, standing side by side, one reaches toward the viewer while the other watches in silence, dramatic tension, anime key visual
```

每个角色都要绑定外貌与位置。画面里出现第三个人却没有人数标签，模型会自己加人。

## 混写原则

前半段用标签定风格、角色、镜头、构图边界；后半段用一句自然语言把动作、道具、冲突、观众关系写活。标签打底，句子补剧情。

自然语言至少两句。太短的 prompt 容易出怪图。

完整拼法示例（明确成年原创角色）：

```text
masterpiece, best quality, score_7, highres, safe, 1girl, solo, adult woman,
Lena-original, pink hair, long hair, amber eyes, navy coat, white scarf,
holding umbrella, standing, eye-level, anime illustration,
Lena, a 25-year-old original character, pauses beneath a streetlight while rain
gathers on her umbrella. She holds a sealed letter toward the viewer, with the
quiet station behind her suggesting the beginning of a journey.
```

注意排布：安全层在质量层之后、人数之前；裸露梯度紧跟服装；剧情句收尾。不要把这套顺序打乱。

## 权重

```
(sweat:1.3)      强调
(trash can:0.6)  减弱
(chibi:2)        强推
```

- 常规强调 1.3–1.6，减弱 0.6–0.8；要压过同段其它标签，数值得到 2 附近才明显。
- 只认 `(tag:数字)`。不要写 `::`、`BREAK`、`(tag:数字)` 以外的任何权重语法。
- 作品名里的圆括号要转义，避免和权重语法打架。

## 负面词

```
worst quality, low quality, score_1, score_2, score_3, artist name, blurry, jpeg artifacts, chromatic aberration
```

按场景追加：`bad anatomy, bad hands, missing fingers, extra digits, cropped, watermark, text, signature, multiple views`。

负向只和用户的否定项一一对应，不做联想扩展。用户说「不要复古」，写 `retro artstyle, old, vintage` 就够，不要顺手加 `vintage microphone`。

## 生成参数

| 参数 | 值 |
| --- | --- |
| 分辨率 | 512² – 1536²；常用 1024×1024、832×1216、1216×832 |
| 步数 | 30–50 |
| CFG | 4–5 |
| 采样器 | `er_sde`（默认）、`euler_a`（细线柔）、`dpmpp_2m_sde_gpu`（多变）、`euler` |
| 调度器 | 要油画质感可试 `beta57`（需 RES4LYF 节点） |

## 查词工具的铁律

`search_tags` 只用来确认标签在不在库里，不决定写什么。

1. **`safe` 是安全标签，不是物体。** Danbooru 另有 `safe (container)`（保险箱）。查 `safe` 会命中它，照抄就在正向里多一个保险箱，安全层反而丢了。安全词原样写，不查、不加括注、不替换。
2. **画师查不到也要写。** 文本编码器吃任意字符串，标签库收录不全。用户点名的画师查空也照写 `@原名`，保留括注（`@ask (askzy)`）。查空只代表没收录，不代表不存在。
3. **负向不做二次联想。** 见上节。

## 自检

1. 七层齐：质量、元信息、安全、主体、身份、细节、剧情。
2. 安全词干净，没有 `safe (container)`。
3. 人数标签与实际人物数一致，每人有外貌锚点。
4. 构图与可见性不冲突：`upper body` 没有膝脚，`from behind` 没有正脸。
5. 3 人以上没有 `close-up`，4 人以上没有 `cowboy shot`。
6. 剧情句写清了动作、道具、冲突、观众关系。
7. 权重只用了 `(tag:数字)`，数值在 1.3–2.0 之间。
8. 画师带 `@`，数量 ≤2。
9. 负向没有高分 score 标签，也没有联想出来的具体物件。

## 来源

- https://huggingface.co/circlestone-labs/Anima
- 项目七层提示词模板（维护者提供）
- 社区补充：anima-prompt-writing、anima-prompt-crafter-skill、ComfyUI-EasyUseAnima
