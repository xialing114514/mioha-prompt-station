# Krea 2 · 二次元 SKILL v1（krea-anime）

> 适用：Krea 2 Medium / Turbo，以及以 Krea 2 为底座的动漫微调（Krea 2 Anime、JANK2 等）。
> 核心认知：**Krea 2 没有画师标签体系，二次元画风靠「风格 LoRA + 风格参考 + 描述」三层补，不靠 `artist:`。**
> 与 `krea-real` 是两份独立 skill。Krea 2 没有负面输入框，本文第五节给的是正向替代句。

---

## 执行卡（先读这段，读完即可产出）

**任务**：把用户需求转成 Krea 2 的二次元提示词。**正向是一段连续英文散文；不要输出负面块。**

**硬规则**
1. 只输出一个代码块（正向散文）加一行参数。不要输出负面、不要输出标签表。
2. 散文按顺序覆盖：主体动作 → 外观服装 → 构图机位 → 场景 → 光色 → 画面语言（线条/上色/眼睛）。
3. 画面语言必须写：至少一项线条词、一项上色词、一项眼睛词（`clean lineart` / `cel shading` / `detailed eyes`）。
4. 不写画师名、不写 `artist:`、不写 `@名字`、不写质量词、不写权重、不写 `no ...`。
5. LoRA 触发词整段照抄，放在散文最后一句。
6. 用户没给的信息用默认值补齐，不反问。
7. 骨架里的方括号是占位符，**必须换成具体值**；用户没指定画师时用 `ciloranko`，不要留方括号、不要照抄占位字。

**露骨度**：露骨程度跟随用户。用户要性行为，就写具体器官、接触方式、体液与生理反应（cock / precum / cum / glans / soles / toes），不要用「隔着布料」「暗示」「留白」替代，也不要因为示例是通用版就把它当尺度上限。

**输出格式**

```text
[一段英文散文，末句放 LoRA 触发词（若有）]
```

参数：Krea 2 Medium / creativity medium / 1024×1024 / LoRA: retroanime（权重 0.8，在节点里设）

**填空骨架**

```text
A [主体] [动作], [外观与服装], [景别] from [机位], [场景], [光]. [线条词], [上色词], [眼睛词]. [LoRA 触发词]
```

**以下输出一定是错的**
- 输出负面块（Krea 2 没有这个字段）；
- 写 `artist:wlop`、`@名字`、`masterpiece`；
- 把 LoRA 权重写成 `(x:0.8)` 塞进正文；
- 漏掉画面语言段，出来是「插画风的动漫角色」而不是动画感。

---

## 一、模型与参数

| 项 | 值 | 说明 |
| --- | --- | --- |
| 首选变体 | Krea 2 Medium | 官方定位就是「插画、动漫、绘画、表现性风格」 |
| 快速迭代 | Krea 2 Turbo | 8 步、CFG 0、`mu 1.15`，1K–2K |
| 动漫微调 | Krea 2 Anime、JANK2 等 | 社区把 Turbo 继续微调成默认动漫输出，写不写 `anime` 都能出动画风 |
| 分辨率 | 1:1 1024²、3:2 1248×832、2:3 832×1248、16:9 1376×768、9:16 768×1376 | 托管版统一 1K 档 |
| creativity | `low` / `medium`（默认）/ `raw` | 定死细节用 raw；要它补氛围用 medium |
| LoRA | Krea 风格（Krea Styles）或社区 LoRA | 官方 LoRA 以 Turbo 为底，权重从 0.6 起试 |

---

## 二、画风控制的四级优先

按可靠性排序，从上到下依次降级：

| 级 | 手段 | 用法 | 可靠性 |
| --- | --- | --- | --- |
| 1 | **风格 LoRA** | 载入官方或社区 LoRA，触发词照抄，权重 0.6–0.9 | 高 |
| 2 | **风格参考图（Srefs）** | 最多 4 张参考图，各自有强度滑杆；只取风格，不取构图 | 高 |
| 3 | **情绪板** | 一组同方向的图，定配色、质感、氛围 | 中高 |
| 4 | 提示词描述 | 写媒介、线条、上色方式、年代 | 中 |

**知道局限**：Krea 2 不认识画师名，也不认识大部分版权角色。写 `artist:wlop`、`@yoneyama_mai`、角色名都当普通文字。要某个画风，只能走上面四级；要某个角色，用角色 LoRA 或把外观写全。

---

## 三、正向结构（八段）

| # | 槽位 | 写什么 | 例 | 可省略 |
| --- | --- | --- | --- | --- |
| 1 | **主体与动作** | 谁、几个、在做什么、朝向、视线、表情 | `A girl in a sailor uniform leans over the railing and looks back over her shoulder.` | 否 |
| 2 | **外观与服装** | 发型发色、瞳色、服装款式与配色、配饰 | `Short black bob with blunt bangs, amber eyes, navy sailor collar, red neckerchief, pleated skirt.` | 否 |
| 3 | 道具 | 手里拿的、身边的、承载叙事的物件 | `A paper cup of shaved ice in one hand, melting at the rim.` | 是 |
| 4 | **构图与机位** | 景别、角度、透视、景深 | `Cowboy shot, low angle, slight dutch tilt, shallow depth of field.` | 否 |
| 5 | **场景** | 地点、时间、天气、背景元素 | `School rooftop at dusk, water tank, chain-link fence, city below.` | 否 |
| 6 | **光与色** | 光源方向、色温、主色、对比 | `Warm sunset backlight, rim light on hair, orange and teal palette.` | 否 |
| 7 | **画面语言（anime 专属）** | 线条、上色、完成度、年代 | `Clean lineart, crisp cel shading with hard-edged shadows, vibrant flat colors, detailed eyes.` | 否 |
| 8 | **LoRA 触发词** | 照抄官方/社区给的触发串，放最后 | `Purple retro anime style` | 是（无 LoRA 时） |

第 7 段是 Krea 2 出不出「动画感」的关键：不写它，Medium 会给你一张「有动漫角色的插画」，而不是动画截图的质感。

---

## 四、LoRA 使用规则

| 规则 | 内容 |
| --- | --- |
| 触发词 | 整段照抄，不改写、不拆词、不翻译；多数触发词放在提示词末尾 |
| 权重 | 0.6 起试；风格不明显加到 0.8–0.9；超过 1.0 容易过曝、线条发硬 |
| 叠加 | 最多 2 个（1 风格 + 1 角色）；叠加时风格降 0.1–0.2，角色保持 |
| 训练底模 | 官方 LoRA 以 Turbo 为底；社区 LoRA 用 RAW 训、Turbo 跑 |
| 滑块型 LoRA | 双向：正权重推向写实，负权重推向动漫；同一张图只用一次，不要和其他风格 LoRA 叠 |
| 失效排查 | 画面不动 → 权重太低或底模不匹配；画面糊 → 权重过高；风格串味 → 两个 LoRA 打架，删一个 |

**双向滑块 LoRA 的用法**（社区常见的一支）：

| 想要 | 做法 |
| --- | --- |
| 动漫图更「实」一点 | LoRA 权重设为 **+0.35**（正数推向写实） |
| 写实底图更「动画」一点 | LoRA 权重设为 **−0.5**（负数推向动漫） |

权重是在 LoRA 节点或 Civitai 的滑块里设的，**不是写进提示词的**——Krea 2 不解析 `(x:0.35)` 这种写法，写进提示词只会变成字面文本。这类 LoRA 只调「真实感 ↔ 动漫」一个轴，不换画风；换画风仍要风格 LoRA。

---

## 五、负面的替代表达

Krea 2 无负面通道，按三层写正面：

> 这些句子是**写进正向散文的素材**，不是单独的负面块；Krea 2 没有负面字段。

**必需层（防糊防脏）**

```text
tack-sharp lineart, clean edges, crisp cel shading, even flat color areas,
detailed eyes with clean highlights
```

**按需层（人像 / 多人 / 复杂背景）**

```text
readable facial features, consistent character design across the frame,
one clear subject, correct hand anatomy, uncluttered background
```

**可选层（按风格选）**

| 想压住 | 改写成正面 |
| --- | --- |
| 照片感 / 3D 感 | `hand-drawn 2D lineart, flat cel shading with two-tone shadows` |
| 塑料皮肤 | `matte anime skin with flat shadow shapes, soft blush on the cheeks` |
| 上色脏 | `limited palette, clean color separation, two-tone shading` |
| 画面糊 | `crisp line weight, clear silhouette, sharp separation between hair and background` |
| 画风漂移 | 降 creativity 到 `raw`，并把画面语言写死 |

---


**与 NSFW 过滤的关系**：本 skill 没有负面串，因此不存在负面词条污染；分级由提示词内容与平台策略决定，不由本文件表达。
## 六、角色一致性

- **有角色 LoRA**：LoRA 触发词 + 关键外观锚点（发色、瞳色、标志配饰），其余交给 LoRA。
- **没有角色 LoRA**：外观写全（发长、发色、瞳色、刘海、服装、配饰），每张图重复一遍；Krea 2 不会因为写了名字就记住角色。
- 多图连续：身份细节每张重复，换姿势、换机位要明说。
- 多角色：每个角色一段，用 `GIRL1` / `GIRL2` 这类短标签，完整描述一次后只用标签，不用代词。

---

## 七、自检清单（交付前逐条过）

1. 变体选对：Medium / Turbo / 动漫微调，三者参数与表现不同。
2. 画风靠 LoRA 或参考图，没写画师名、没写 `artist:` / `@名字`。
3. LoRA 触发词整段照抄，放在末尾；权重 0.6–0.9。
4. 第 7 段（画面语言）写了：线条、上色、完成度至少各一项。
5. 没有质量词（`masterpiece` / `8k` / `best quality`）。
6. 没有负面句；要压的方向已改写成正面。
7. 角色一致性有锚点；多角色用短标签、不用代词。
8. creativity 与需求匹配：定死细节用 `raw`，要氛围用 `medium`。

---

## 八、完整示例

### 示例 1｜官方复古动画 LoRA

```text
A girl in a navy sailor uniform runs along a coastal road at sunset, one hand holding her hat, the other stretched back, hair and skirt pulled by the wind. Short black bob, blunt bangs, amber eyes, red neckerchief fluttering. Wide shot from a low angle, the road curving toward the sea on the right. Telephone poles and a low sea wall on the left, the ocean glowing under a heavy orange sky. Strong backlight from the horizon, long shadows on the asphalt, warm rim light on her hair and shoulders. Clean lineart, crisp cel shading with hard-edged shadows, limited warm palette, detailed eyes with bright highlights. Purple retro anime style
```

参数：Krea 2 Medium / creativity medium / 1024×1024 / LoRA: retroanime（权重 0.8，在节点里设）

---

## 九、常见错误

| 错误 | 后果 |
| --- | --- |
| 写画师名想复刻画风 | 当普通文字，画风不变 |
| 不写第 7 段画面语言 | 出「插画风的动漫角色」，不是动画感 |
| LoRA 触发词翻译或改写 | 触发失败 |
| 两个风格 LoRA 同权重叠加 | 画面糊、风格打架 |
| 用 `masterpiece` 补质量 | 无效，占预算 |
| 用反向 LoRA 又写正向风格词 | 两个方向互相抵消 |

---

## 十、来源

- 官方 API / 变体说明（Medium 擅长插画动漫）：https://www.krea.ai/docs/cn/developers/krea-2/overview
- 官方使用指南（变体、风格参考、情绪板、滑杆）：https://www.krea.ai/docs/cn/user-guide/features/krea-2
- 官方提示词指南：https://github.com/krea-ai/krea-2/blob/main/docs/prompting.md
- 官方复古动画 LoRA（触发词 `Purple retro anime style`）：https://huggingface.co/krea/Krea-2-LoRA-retroanime
- 社区 LoRA 生态（Krea 2 Anime、JANK2、双向风格滑块）：Civitai `baseModel=Krea 2` 条目，本地存档 `资料文件夹/community/krea/civitai_krea_lora.json`
- 群内实测（Krea 2 无画师串、动漫向靠 LoRA）：`资料文件夹/chats/全部消息_纯文本.txt`，2026-09-07 / 2026-09-12
