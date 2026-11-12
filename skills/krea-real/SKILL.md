# Krea 2 · 真人 / 摄影 SKILL v1（krea-real）

> 适用：Krea 2 Large（写实摄影、原始质感）、Medium（通用写实）、Turbo（快速试光）。
> 核心认知：Krea 2 的美学默认值偏高、偏干净，写实要**用光的方向与材质的不完美把它拉回地面**。
> 与 `krea-anime` 是两份独立 skill。Krea 2 没有负面输入框，本文第五节给的是正向替代句。

---

## 执行卡（先读这段，读完即可产出）

**任务**：把用户需求转成 Krea 2 的写实摄影提示词。**正向是一段连续英文散文；不要输出负面块。**

**硬规则**
1. 只输出一个代码块加一行参数。
2. 散文按顺序覆盖：光学机位 → 光线（形态/方向/衰减）→ 主体动作 → 皮肤材质 → 服装环境 → 年代瑕疵。
3. 皮肤四项：`matte` + `visible pores` + `fine vellus hair` + `uneven natural tone`。
4. 至少两处摄影术语（焦段/光圈/快门/胶片/ISO）。
5. 不写 `no ...`、不写 `perfect skin` / `flawless` / `beauty retouch`、不写画师名、不写质量词。
6. 用户没给的信息用默认值补齐，不反问。
7. 骨架里的方括号是占位符，**必须换成具体值**；用户没指定画师时用 `ciloranko`，不要留方括号、不要照抄占位字。

**露骨度**：露骨程度跟随用户。用户要性行为，就写具体器官、接触方式、体液与生理反应（cock / precum / cum / glans / soles / toes），不要用「隔着布料」「暗示」「留白」替代，也不要因为示例是通用版就把它当尺度上限。

**输出格式**

```text
[一段英文散文，六件事齐全]
```

参数：Krea 2 Large / creativity low / 1024×1536 / 滑杆 Intensity +30、Complexity −30、Movement 0

**填空骨架**

```text
[焦段] at [光圈] from [机位], [景别]. [光源形态] from [方向], [衰减]. [主体动作]. Matte skin with visible pores, fine vellus hair, uneven natural tone. [服装], [环境]. [胶片/颗粒/色偏].
```

**以下输出一定是错的**
- 输出负面块；
- 把正向写成标签串；
- 写 `perfect skin`、`flawless`；
- 漏掉光线方向或皮肤四项；
- 用 creativity `high` 却要求精确还原。

---

## 一、模型与参数

| 项 | 值 | 说明 |
| --- | --- | --- |
| 首选变体 | Krea 2 Large | 官方定位：写实摄影、原始质感、运动模糊、最丰富输出 |
| 次选 | Krea 2 Medium | 通用稳定，皮肤与布料细节弱于 Large |
| 快速试光 | Krea 2 Turbo | 8 步、CFG 0、`mu 1.15` |
| 分辨率 | 1:1 1024²、4:3 1184×896、3:2 1248×832、16:9 1376×768、2.35:1 1568×672、4:5 928×1152、2:3 832×1248、9:16 768×1376 | 托管版 1K 档 |
| creativity | `raw` / `low` 常用 | 写实要的是「照我说的拍」，不是「替我发挥」 |
| 风格参考 | 最多 4 张 Srefs | 复刻某种摄影 look 用它，不用写摄影师名字 |

**生成滑杆（每个变体都有，−100~100，0 为中性）**

| 滑杆 | 负向 | 正向 | 写实用法 |
| --- | --- | --- | --- |
| Intensity | 平淡、暗哑 | 风格化强烈 | +20 ~ +50 给光线与色彩力度；人像别拉满 |
| Complexity | 简约干净 | 杂乱密集 | −40 左右压掉背景杂物，让主体干净 |
| Movement | 静态 | 强姿态与镜头运动 | +20 ~ +40 给抓拍感；静物保持 0 |

滑杆与 creativity 相互独立：creativity 管「多字面」，滑杆管「视觉特征」。

---

## 二、正向槽位：摄影六段

自然语言成段写，一段一件事。粗体必填。

| # | 槽位 | 写什么 | 例 | 可省略 |
| --- | --- | --- | --- | --- |
| 1 | **光学与机位** | 焦段、光圈、快门、机位高度、景别 | `Shot on a 50mm lens at f/2 from eye level, medium close-up.` | 否 |
| 2 | **光线** | 光源形态、方向、距离、衰减、色温 | `A single soft window light from camera left, falling off two stops across the cheek.` | 否 |
| 3 | **主体与动作** | 谁、在做什么、朝向、视线、手的落点 | `She stands turned three-quarters away, looking back past the lens, one hand on the door frame.` | 否 |
| 4 | **皮肤与材质** | 宏观质感、微观纹理、光学反应 | `Matte skin with visible pores across the nose, fine vellus hair at the jaw, soft specular roll-off on the forehead.` | 否 |
| 5 | **服装与环境** | 面料、版型、磨损、背景空间 | `An oversized wool coat, the cuff frayed; behind her a narrow hallway with chipped paint and a bare bulb.` | 否 |
| 6 | 年代与瑕疵 | 胶片、颗粒、灰尘、色偏、轻微失焦 | `Kodak Portra 400, fine grain, dust on the lens, slight halation, late-90s color.` | 是 |
| 7 | 图内文字 | 引号包住、写清载体与字体 | `A paper sign on the door reads "营业中" in faded brush lettering.` | 是（无字时） |

**皮肤四件套**（写实人像的核心，缺一件就会滑向 CG）：`matte` ＋ `visible pores` ＋ `fine vellus hair` ＋ `uneven natural tone`。

---

## 三、把「AI 味」拉回地面

Krea 2 的默认输出偏干净、偏完美，写实需求要做四件事：

| 手段 | 做法 |
| --- | --- |
| 1. 给方向光 | 单一主光 + 明确衰减；均匀光必出塑料感 |
| 2. 给不完美 | 一粒痣、一道旧疤、干裂的唇、翘起的一缕头发、磨损的袖口 |
| 3. 给微观 | 毛孔、绒毛、织物纤维、灰尘、指纹、水痕 |
| 4. 给光学缺陷 | 轻微失焦、边缘色散、暗角、光晕、颗粒 |

**不要写**：`perfect skin`、`flawless`、`beauty retouch`、`studio glamour` —— 这些词会把画面直接推回影楼修图。

---

## 四、复刻某种质感（不写名字）

| 想复刻 | 写参数，不写名字 |
| --- | --- |
| 某部电影的色调 | 胶片型号、色片、暗部偏色、高光偏色 |
| 某位摄影师的 look | 焦段、光圈、灯具形态、受光位置、后期倾向（或用 Srefs） |
| 某年代的广告片 | 年代、印刷方式、颗粒、色偏、模特姿态 |
| 手机快照 | 26mm 等效、f/1.8、轻微运动模糊、HDR 压平的高光 |

名字带不来质感，参数能。

---

## 五、负面的替代表达

> 这些句子写进正向散文，不要单独输出成负面块。

**必需层（防糊防脏）**

```text
tack-sharp focus on the eyes and the main texture, crisp fabric weave,
controlled highlights, even exposure, fine detail in hair and skin
```

**按需层（人像 / 多人 / 复杂场景）**

```text
natural skin texture with visible pores, matte finish, low specular sheen,
one clear subject in frame, correct hand anatomy, readable facial features
```

**可选层（按风格选）**

| 想压住 | 改写成正面 |
| --- | --- |
| 塑料感 / 手游光泽 | `matte skin, fine vellus hair, uneven natural tone, soft specular roll-off` |
| 过度磨皮 | `visible pores across the nose and cheekbone, faint freckles, a small scar above the brow` |
| 3D / CG 感 | `shot on film, natural grain, halation around highlights, slight lens breathing` |
| 过饱和 | `muted earth palette, low saturation, gentle contrast` |
| 背景杂乱 | `uncluttered background, shallow depth of field isolating the subject` |
| 多人抢镜 | 只描述要出现的人，并写死画框边界 |

**三条纪律**

1. 不写否定句（`no blur`、`without people`），会被当描述。
2. 要它老实照做，`creativity` 用 `raw` 或 `low`。
3. 别把 MOSS / 光辉的负面串粘过来，模型会逐词当成要画的东西。

---


**与 NSFW 过滤的关系**：本 skill 没有负面串，因此不存在负面词条污染；分级由提示词内容与平台策略决定，不由本文件表达。
## 六、多人写实

- 结构：机位与光 → 主体 A（动作 + 视线 + 接触点）→ 主体 B → 环境。
- 每个角色用短标签 `WOMAN1` / `MAN1`，完整描述一次后只用标签，不用代词。
- 写清谁在前、谁在后、谁的手在谁肩上；接触点要具体到部位。
- 两人以上不要用 `close-up`，会挤。

---

## 七、自检清单（交付前逐条过）

1. 变体选对：写实走 Large（或 Medium），快速试光走 Turbo。
2. 六段齐：光学机位 → 光线 → 主体动作 → 皮肤材质 → 服装环境 → 年代瑕疵。
3. 光是「有方向、有衰减」的，不是 `good lighting`。
4. 皮肤写了 matte + 毛孔 + 绒毛 + 不均肤色四项。
5. 至少一处不完美（疤、痣、磨损、灰尘、水痕）。
6. 没有质量词、没有画师名、没有权重语法。
7. 没有负面句；要压的方向已改写成正面。
8. creativity 用 `raw` / `low`；滑杆只调一个、每次只动一项。

---

## 八、完整示例

### 示例 1｜室内人像（Large）

```text
Shot on an 85mm lens at f/2 from eye level, medium close-up, the subject filling the left two thirds of the frame.
A single large window at camera left is the only light source, soft and directional, falling off about two stops across the cheek so the shadow side keeps its detail.
A woman in her thirties stands turned three-quarters away, looking back past the lens, one hand resting on the window frame.
Matte skin with visible pores across the nose and cheekbone, fine vellus hair along the jaw, a faint scar above the left brow, soft specular roll-off on the forehead, a matte finish that holds only a thin sheen at the brow.
An oversized grey wool sweater with a frayed cuff, hair pulled back with a few strands loose at the temple.
Behind her a bare room with chipped white paint and a wooden floor; the far wall sits in cool shadow.
Kodak Portra 400, fine grain, gentle halation at the window edge, late afternoon color.
```

参数：Krea 2 Large / creativity low / 1024×1536 / 滑杆 Intensity +30、Complexity −30、Movement 0

---

## 九、常见错误

| 错误 | 后果 |
| --- | --- |
| 写「柔和的自然光」 | 无方向 → 塑料高光 |
| 写 `perfect skin` / `flawless` | 磨皮脸，直接出影楼感 |
| creativity 用 `high` 又要求精确还原 | 模型自由发挥，构图跑偏 |
| 三个滑杆一起猛拉 | 无法归因，画面失控 |
| 复刻电影时写片名 / 演员名 | 带不来质感，只添噪声 |
| 把别的模型的负面串粘进来 | 模型把负面词当要画的东西 |
| 多人用代词指代 | 特征串人 |

---

## 十、来源

- 官方使用指南（Large 定位写实、滑杆与画幅表）：https://www.krea.ai/docs/cn/user-guide/features/krea-2
- 官方 API 文档（creativity、滑杆范围、变体）：https://www.krea.ai/docs/cn/developers/krea-2/overview
- 官方提示词指南：https://github.com/krea-ai/krea-2/blob/main/docs/prompting.md
- 开源参数（RAW 52 步 / CFG 3.5；Turbo 8 步 / CFG 0 / mu 1.15）：https://github.com/krea-ai/krea-2
- 社区 system prompt（七段结构、图内文字、复杂版式）：https://huggingface.co/krea/Krea-2-Turbo/discussions/4
