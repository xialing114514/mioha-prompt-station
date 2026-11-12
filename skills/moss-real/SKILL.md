# MOSS-Image · 真人 / 摄影 SKILL v5（moss-real）

> 适用：MOSS-Image Base / SFT 的写实人像、纪实、产品、场景摄影。
> 与 `moss-anime` 是两份独立 skill，**负面基线完全不同**，不要同时启用；本文件的负面里含反二次元 / 反 CG 项，粘到二次元需求上会把画面推成照片。
> 执行只需读「执行卡」；下面是依据、细则与示例。负面已过滤 NSFW 项。

---

## 执行卡（先读这段，读完即可产出）

**任务**：把用户需求转成 MOSS-Image 写实摄影提示词。**正向是一段连续散文，不是标签串。**

**硬规则**
1. 只输出两个代码块加一行参数。代码块内只放英文。
2. 正向一段话，按顺序覆盖六件事：光学与机位 → 光线（形态/方向/衰减）→ 主体动作 → 皮肤与材质 → 服装环境 → 年代与瑕疵。
3. 皮肤必须写满四项：`matte` + `visible pores` + `fine vellus hair` + `uneven natural tone`。
4. 至少两处摄影术语：焦段、光圈、快门、胶片、ISO 任选。
5. 至少一处不完美：疤、痣、磨损、灰尘、水痕。
6. 不写 `no ...`、`without ...`；不写 `perfect skin`、`flawless`、`beauty retouch`。
7. 负面原样输出本文件的必需层（含 `cartoon, anime, illustration, painting, 3d render, glossy skin, plastic skin`），不要删这几项。
8. 用户没给的信息用默认值补齐，不反问。
9. **必须输出全部三块**：正向、负向、参数行。缺任何一块都算失败，哪怕用户只给了一句场景。
10. **正向是一整段**，段内不空行；皮肤四项逐字出现：`matte` / `visible pores` / `fine vellus hair` / `uneven natural tone`。
11. 骨架里的方括号是占位符，**必须换成具体值**；不要留方括号、不要照抄占位字。

**露骨度**：露骨程度跟随用户。用户要性行为，就写具体器官、接触方式、体液与生理反应（cock / precum / cum / glans / soles / toes），不要用「隔着布料」「暗示」「留白」替代，也不要因为示例是通用版就把它当尺度上限。

**输出格式**

```text
[一段英文散文，六件事齐全]
```

```text
worst quality, bad anatomy, watermark, jpeg artifact, cartoon, anime, illustration, painting, 3d render, cgi, glossy skin, plastic skin, blurry, out of focus, lowres, bad hands, extra fingers, fused fingers, deformed hands, distorted face
```

参数：euler + normal / 50 步 / CFG 3.0 / Shift 6.0 / SLG 3（skip 26）/ 1024×1536

**填空骨架**

```text
[焦段] at [光圈] from [机位], [景别]. [光源形态] from [方向], [衰减]. [主体动作]. Matte skin with visible pores, fine vellus hair, uneven natural tone, [一处瑕疵]. [服装], [环境]. [胶片/颗粒/色偏].
```

**以下输出一定是错的**
- 把正向写成标签串；
- 写 `no blur`、`no people`；
- 漏掉皮肤四项中的任何一项；
- 负面删掉 `cartoon / anime / 3d render / glossy skin / plastic skin`；
- 把「已过滤 NSFW 项」写进提示词。

---

## 一、模型参数默认值

| 参数 | 默认 | 区间 | 说明 |
| --- | --- | --- | --- |
| 采样器 | `euler` + `normal` | `euler` + `sgm_uniform` | — |
| 步数 | 50 | ≥50 | 官方下限 50；写实降步数会先崩皮肤与手指 |
| CFG | 3.0 | 3.0–4.0 | 写实不要拉高；CFG 越高负面权重越大，容易过度平滑 |
| Shift | 6.0 | 5.0–7.0 | 写实常用 6.0 |
| SLG | 开，scale 3，skip layer 26 | 22 / 24 / 26 | **26 常用于写实**；关 SLG 更「真」但细节会塌 |
| 分辨率 | 1.5M 总像素 | 1M / 1.5M / 2M | 宽高必须是 16 的正整数倍 |
| 版式 | 1024×1536 / 1536×1024 / 1456×1456 | — | 人像多用 1024×1536 |

档位取舍：

| | 档 Q（交付） | 档 R（快迭代） |
| --- | --- | --- |
| 分辨率 | 1.5M | 0.6M |
| 步数 | 50 | 16–30 |
| SLG | 开 26 | 关 |
| 特征 | 皮肤纹理、睫毛、织物纤维收敛得好 | 真实感更强，手指与眼睛易崩 |
| 补偿 | 不需要堆防崩词 | 必须补「手、眼、布料」的正面细节描述 |

---

## 二、正向槽位：摄影五段（自然语言主导）

**整段散文，不写标签串。** 一段一件事，按下面的顺序铺（这五段就是本 skill 的槽位模板）：

| # | 槽位 | 写什么 | 例 | 可省略 |
| --- | --- | --- | --- | --- |
| 1 | **光学与显影** | 胶片/机身、镜头焦段、光圈、快门、滤镜 | `shot on a 35mm film body with a 50mm lens at f/1.8` | 否 |
| 2 | **机位与遮挡** | 平视/俯拍/仰拍、景别、前景遮挡 | `eye-level medium close-up, a blurred doorframe cutting the left edge` | 否 |
| 3 | **光线与受光** | 灯具形态、角度、距离、衰减、色温 | `a single softbox at 45 degrees from camera left, hard falloff on the cheek` | 否 |
| 4 | 年代与调色 | 时代感、色片、暗部偏色、高光偏色 | `late-90s Hong Kong color, teal shadows, warm highlights` | 可省（要中性色时） |
| 5 | **质感与瑕疵** | 皮肤质感、衣料、颗粒、灰尘、划痕 | `visible pores, fine vellus hair, subtle grain, dust on the lens` | 否 |
| 6 | 主体与动作 | 谁、在做什么、穿什么、接触什么 | `she turns three-quarters away and looks back toward the lens` | 可省（纯场景图时） |
| 7 | 图内文字 | 招牌、标签、包装上的字（引号包裹） | `"营业中" printed on frosted glass` | 可省（无字时） |

1、2、3、5 四段是硬槽位，缺任何一段都会让画面往「手游 CG」方向滑；4 段决定年代感，6 段决定叙事，7 段只在要画字时出现。

**铁律：描述成品画面，不写下指令。** 写「她站在雨里，伞面挂着水珠」，不写「请画一个下雨的场景」。要排除什么，写进负面提示词，不用句子否定，也不要写 NAI 的负权重。

---

## 三、正向必须包含的三类词

**1. 光线（至少三项：形态 / 方向 / 衰减）**

```text
soft window light from camera left, 45 degrees, gentle falloff into the shadow side
one hard key from above-right, deep shadow under the jaw
practical lamp behind her, warm rim on the shoulder
```

**2. 皮肤质感（三层：宏观 / 微观 / 光学）**

```text
macro: matte, natural tone, slight redness on the cheek and nose
micro: visible pores across the nose and cheekbone, fine vellus hair along the jaw
optical: soft specular roll-off on the forehead, subsurface warmth in the ear
```

**3. 摄影术语（至少两项）**

```text
50mm at f/1.8, 1/125s, ISO 400, Kodak Portra 400, shallow depth of field,
slight halation, natural vignetting, handheld micro-shake
```

这三类词缺一类，出来的图就会往「手游 CG」方向滑。

---

## 四、抵消「手游光泽感」（本 skill 核心）

MOSS 在写实人物上有一个稳定倾向：皮肤被处理成均匀发光、高光瓷化、暗部被提亮，观感接近手游立绘的 CG 皮肤。社区把这种质感叫「油腻」「塑料感」，silver-moss 的写实通道把 `plastic skin`、`smooth waxy skin`、`airbrushed skin` 列为必封项。

**四步抵消法（按顺序叠加）：**

| 步 | 手段 | 具体写法 |
| --- | --- | --- |
| 1 | 正面写「不发光」的皮肤 | `matte skin, natural uneven tone, visible pores, fine vellus hair, subtle redness` |
| 2 | 正面写「有方向的光」 | `single directional key from camera left, hard falloff, deep shadow side`——均匀光必然出瓷面 |
| 3 | 正面写「不完美」 | `a small scar on the brow, faint freckles, slightly dry lips, one loose strand` |
| 4 | 负面封杀 | `glossy skin, plastic skin, airbrushed skin, smooth waxy skin, cgi, 3d render, doll, digital art` |

补充两条：

- **CFG 不要拉高**。CFG 越高负面权重越大，容易过度平滑；写实档保持 3.0。
- **SLG 开 26 比关掉更适合交付**。关掉更「真」，但皮肤细节会先塌，塌完就只剩一层光滑的壳。

---

## 五、负向基线（已过滤 NSFW）

执行时默认输出整个必需层（含反二次元 / 反 CG 组）+ 按需层，不要删 `cartoon / anime / illustration / painting / 3d render / glossy skin / plastic skin`。

**必需层（含反二次元 / 反 CG）**

```text
worst quality, bad anatomy, watermark, jpeg artifact,
cartoon, anime, illustration, painting, 3d render, cgi,
glossy skin, plastic skin
```

**按需层（人像必加）**

```text
blurry, out of focus, lowres, bad hands, extra fingers, fused fingers, missing fingers,
deformed hands, distorted face, asymmetrical eyes, extra limbs, fused bodies, low quality
```

**可选层（按场景选）**

| 加什么 | 什么时候 |
| --- | --- |
| `airbrushed skin, smooth waxy skin, doll, digital art` | 皮肤仍然发亮时 |
| `cosplay, wig, synthetic hair` | 头发像假发片时 |
| `dof, bokeh`（反向） | 不需要虚化时 |
| `visible eyes, open eyes` | 闭眼/遮眼场景要防止眼睛被画出来 |
| `feet on floor` | 悬空姿态、高脚凳 |
| `multiple heads, extra head` | 分镜 inset 多长出人头 |

**已过滤 NSFW 项**：官方基线原串首项 `nsfw` 已移除；语料中的 `loli`、`child`、`cleavage` 等词条替换为 `bad proportions`、`distorted face`、`unnatural body`。本 skill 负面基线不含任何 NSFW 词条。负面词只改变采样方向，不承担内容过滤职责；内容边界由用户与部署层决定。

---

## 六、皮肤质感词库（按需取）

| 维度 | 词 |
| --- | --- |
| 表面 | `matte skin`、`satin finish`、`natural sheen`、`slightly oily t-zone`（要真实油脂时用） |
| 微观 | `visible pores`、`fine vellus hair`、`skin texture`、`freckles`、`mole`、`scar` |
| 光学 | `soft specular roll-off`、`subsurface scattering`、`matte finish`、`uneven tone` |
| 状态 | `flushed cheeks`、`tired eyes`、`chapped lips`、`sweat on the temple` |

要「干净通透」就写 `clean matte skin with even tone`；要「真实粗糙」就写 `visible pores, uneven tone, tiny blemishes`。**两种都要有方向光**。

---

## 七、图内文字（写实场景常用）

- 招牌、包装、标签上的字全部进引号，一行一个引号，避免价格错配到别的菜名。
- 曲面上的字宜短，两行以内最好；长段落和生僻字最容易崩。
- 收尾加一句 `Keep all text sharp, fully visible, and free of glare.`
- 画面要出现文字时，把 `watermark` 从负面里去掉。

---

## 八、自检清单（交付前逐条过）

1. 正向是散文，不是标签串；五段顺序在，段与段不重复。
2. 光线写了形态 + 方向 + 衰减三项，不是一句 `good lighting`。
3. 皮肤质感写了宏观 + 微观 + 光学三层，且 `matte`、`visible pores`、`fine vellus hair`、`uneven natural tone` 四个短语逐字出现。
4. 摄影术语 ≥2 项（焦段 / 光圈 / 胶片 / 快门 / ISO）。
5. 没有写画外之物（谁按快门、相机在哪只手上）。
6. 负面含 `cartoon, anime, illustration, painting, 3d render, glossy skin, plastic skin`。
7. 负面没有 NSFW 词条；要画字时 `watermark` 已去掉。
8. 分辨率宽高是 16 的倍数，总像素 1M–2M。

---

## 九、完整示例

### 示例 1｜室内人像 · 窗光

```text
Shot on a 35mm film body with an 85mm lens at f/2, 1/125s, ISO 400, Kodak Portra 400, eye-level medium close-up with a blurred curtain edge cutting the right of the frame. A single large window at camera left is the only light, falling off about two stops across the cheek so the shadow side keeps its detail. She sits turned three-quarters away and looks back toward the lens, lips slightly parted, one hand resting on the windowsill. Matte skin with visible pores across the nose and cheekbone, fine vellus hair along the jaw, uneven natural tone with a faint redness at the cheek, soft specular roll-off on the forehead and a small dry patch beside the brow. An oversized knit sweater with visible fibers, one loose strand of hair over the shoulder. Late-90s interior color, warm highlights on the skin, cool grey shadow in the room behind, subtle grain, slight halation around the window, dust in the air.
```

负向：

```text
worst quality, bad anatomy, watermark, jpeg artifact, cartoon, anime, illustration, painting,
3d render, cgi, glossy skin, plastic skin, airbrushed skin, smooth waxy skin, doll, digital art,
blurry, out of focus, lowres, bad hands, extra fingers, fused fingers, deformed hands,
distorted face, asymmetrical eyes
```

参数：euler + normal / 50 步 / CFG 3.0 / Shift 6.0 / SLG 3（skip 26）/ 1024×1536

---

## 十、常见错误

| 错误 | 后果 |
| --- | --- |
| 正向写成标签串 | 皮肤与光线失控，直接滑向 CG |
| 只写「柔和光」 | 无方向光 → 瓷面高光 |
| CFG 拉到 5–7 | 过度平滑，皮肤像磨皮 |
| 关掉 SLG 又降步数 | 手指、睫毛、织物纤维全塌 |
| 负面混进二次元通道 | 画面被推成插画 |
| 负面里留着 `watermark` 又要画招牌 | 字整体消失 |
| 写「相机握在右手」 | 画面多出一台相机 |
| 复刻某部片时写片名/演员名 | 名字带不来质感，参数才能 |

---

## 十一、来源

- 官方提示词增强 skill（caption / layout 两种交付物）：https://github.com/OpenMOSS/MOSS-Image/blob/main/skills/moss-image-prompt-enhancer/SKILL.md
- 官方使用指南：https://github.com/OpenMOSS/MOSS-Image/blob/main/USERGUIDE.md
- 写实通道、分层负面、档 Q / 档 R：https://github.com/jialanhu0915/silver-moss-image-prompt-skill
- 群聊中「油腻 / 塑料感」的实测吐槽与参数经验：`资料文件夹/chats/全部消息_纯文本.txt`
- 群聊语料拆解：`资料文件夹/群聊语料拆解报告.md`
