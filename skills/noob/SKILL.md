# NoobAI-XL 提示词 SKILL v1

> 适用：NoobAI-XL 1.0 / 1.1（eps 预测）与 NoobAI-XL V-Pred 1.0。
> 关系：NoobAI 是从 Illustrious-XL 续训的 SDXL 模型，标签体系与光辉系同源，但**质量标签的含义、日期标签、e621 兽人标签、v-pred 参数**都与光辉不同，不能直接套 illustrious 的参数。
> 负面已过滤 NSFW 项。

---

## 执行卡（先读这段，读完即可产出）

**任务**：把用户需求转成 NoobAI-XL 的提示词。**先判断版本**：用户没说是 v-pred 就按 1.1（eps）写。

**硬规则**
1. 只输出两个代码块加一行参数；代码块内只放英文。
2. 正向必须以官方起手串开头：`masterpiece, best quality, newest, absurdres, highres, safe`。
3. 全小写，逗号后留空格；角色名与画师名括号转义 `\(...\)`。
4. 画师用纯名或 `artist:名字`，权重只用 `(名字:1.2)`；禁止 `{}`、`N::tag::`。
5. 日期标签与用户想要的年代一致：90 年代写 `old` 或 `early`，不要写 `newest`。
6. 画人类时负面带上兽人排除层；画兽人时删掉这层。
7. 用户没给的信息用默认值补齐，不反问。
9. **必须输出全部块**：正向、负向、参数行，缺一块即失败。
8. 骨架里的方括号是占位符，**必须换成具体值**；用户没指定画师时用 `ciloranko`，不要留方括号、不要照抄 `artist name` 这种占位字。

**露骨度**：露骨程度跟随用户。用户要性行为，就写具体器官、接触方式、体液与生理反应（cock / precum / cum / glans / soles / toes），不要用「隔着布料」「暗示」「留白」替代，也不要因为示例是通用版就把它当尺度上限。

**输出格式**

```text
[正向：起手串, 日期, 人数, 角色, 外观, 服装, 动作, 表情, 镜头, 场景与光, 画师, 风格]
```

```text
[负向：worst quality, old, early, low quality, lowres, signature, username, logo, bad hands, mutated hands, bad anatomy, extra fingers, watermark, text, multiple views, mammal, anthro, furry, ambiguous form, feral, semi-anthro]
```

参数：eps 1.1 / Euler a / 27 步 / CFG 5.5 / 832×1216
参数：V-Pred 1.0 / Euler / 32 步 / CFG 4.5 / 832×1216 / v_prediction + zsnr

**填空骨架**

```text
masterpiece, best quality, newest, absurdres, highres, safe, [日期], [人数], [角色名 \(作品名\)], [外观], [服装], [动作], [表情], [景别], [视角], [场景], [光], ([画师]:1.0), [风格词]
```

**以下输出一定是错的**
- v-pred 用 Euler a 或 CFG 5.5 以上；
- 起手串缺项或顺序打乱；
- 画古早风却写 `newest`；
- 画兽人还留着 `anthro, furry` 负面；
- 用 NAI 的 `{}` / `N::`。

---

## 一、版本与参数（三个版本，三套参数）

| 项 | 1.0 / 1.1（eps） | V-Pred 1.0 |
| --- | --- | --- |
| 预测类型 | eps | v-prediction（必须开 `v_prediction` + `rescale_betas_zero_snr`/zsnr） |
| 采样器 | Euler a（DPM++ 2M 也可） | **Euler，官方明说其他采样器不正常工作** |
| 步数 | 25–30 | 28–35 |
| CFG | 5–6 | 4–5 |
| 分辨率 | 总像素约 1024²：768×1344、**832×1216**、896×1152、1024×1024、1152×896、1216×832、1344×768 | 同左 |
| 前端 | A1111 / Forge / ComfyUI | reForge / ComfyUI / WebUI dev 分支 / Diffusers |
| 模型卡示例 | — | Diffusers 示例用 `num_inference_steps=28, guidance_scale=5` |

**v-pred 的坑**：把 v-pred 当 eps 跑（CFG 5.5 + Euler a 默认设置）会直接出糊图或灰图。换底模先确认预测类型。

---

## 二、输出契约

| 字段 | 内容 |
| --- | --- |
| `finish.positive` | Danbooru 标签，逗号分隔，全小写 |
| `finish.negative` | 负向标签，按第八节三层取用 |
| `finish.texts` | 画面文字（SDXL 文字能力弱，默认留空） |

---

## 三、正向槽位模板

官方给的起手串是固定的，照抄即可：

```text
masterpiece, best quality, newest, absurdres, highres, safe,
```

| # | 槽位 | 内容 | 可省略 |
| --- | --- | --- | --- |
| 1 | **前缀** | `masterpiece, best quality, newest, absurdres, highres, safe` | 否 |
| 2 | **日期** | `newest` / `recent` / `mid` / `early` / `old`（见第四节） | 否（`newest` 已在起手串里） |
| 3 | **人数** | `1girl, solo` / `2girls` / `no humans` | 否 |
| 4 | 角色 | `角色名 \(作品名\)` | 是 |
| 5 | 画师 | 纯名或 `artist:名字`，见第六节 | 推荐 |
| 6 | 外观 | 发长、发色、瞳色、体型、非人特征（含兽人特征） | 否 |
| 7 | 服装 | 款式 + 颜色 + 材质 + 状态 | 否 |
| 8 | 动作 / 表情 | 姿势 + 接触点 + 表情 + 视线 | 否 |
| 9 | 镜头 / 构图 | 景别 + 视角 + 透视 | 否 |
| 10 | 场景 / 光 | 地点 + 时段 + 光源 | 推荐 |
| 11 | 质量补充 | `highres`、`absurdres` 已在起手串，别重复 | 是 |

---

## 四、质量标签与日期标签（NoobAI 独有）

NoobAI 的质量词是按**数据集热度百分位**打的，不是抽象审美：

| 百分位 | 质量标签 |
| --- | --- |
| 前 5%（>95th） | `masterpiece` |
| 30th–95th 之间 | `normal quality` |
| 后 30%（≤30th） | `worst quality` |

模型卡只给了这三档的百分位定义；`best quality` / `good quality` / `bad quality` 在社区里也在用，但模型卡没有给百分位，属于经验写法。官方示例的起手串用的是 `masterpiece, best quality`，照抄即可。

**含义**：`masterpiece` ≈「这张图在 Danbooru 上属于前 5% 的热门图」。想要冷门、古早、小众的质感，写 `masterpiece` 反而会把它拉回热门风格。

日期标签同样有明确定义：

| 年份区间 | 标签 |
| --- | --- |
| 2005–2010 | `old` |
| 2011–2014 | `early` |
| 2014–2017 | `mid` |
| 2018–2020 | `recent` |
| 2021–2024 | `newest` |

画 90 年代赛璐璐要写 `old` 或 `early`，不要写 `newest`；写 `newest` 会拿到 2021 之后的现代画风。

---

## 五、标签结构

- 全小写，下划线换空格。
- **圆括号必须转义**：`frieren \(sousou no frieren\)`、`ask \(askzy\)`。不转义会被 A1111 族当成权重语法吃掉，角色名变成 `sousou no frieren` 的加权词。
- 数量 25–60 个；同义不并列。逗号后留一个空格（CLIP 分词器对 `", "` 有独立 token，紧贴写法切词不同）。
- e621 标签与 Danbooru 标签同库训练：兽人题材写 `anthro`、`feral`、`semi-anthro`、`mammal` 一类；**不画兽人时这组词要进负面**（模型卡默认就这么做）。
- 构图标签互斥同光辉系：景别 / 视角 / 透视各留一个。

---

## 六、画师写法

| 写法 | 例 | 说明 |
| --- | --- | --- |
| 纯名（社区主流） | `wlop, ask \(askzy\)` | 最稳 |
| `artist:` 前缀（模型卡示例） | `artist:john_kafka, artist:nixeu` | 模型卡的 Diffusers 示例这么写，能出图；与纯名差别不大 |
| 权重 | `(wlop:1.2)`、`(ask \(askzy\):0.8)` | 只认括号族 |

**不要写**：`1.2::artist:x::`、`{artist:x}`、`[artist:x]`——那是 NAI 语法，NoobAI 前端不解析，只会变成一串字符。

画师数量 3–6 位；换底模（尤其是不同 merge）时画师串要重调，因为不同 merge 对同一串的响应差别很大。

---

## 七、多人

结构：人数 → 全局镜头/场景 → 各角色（角色名 + 外观 + 服装）→ 动作归属。

```text
2girls, upper body, from side, classroom,
girl A on the left, long black hair, glasses, cardigan, holding a book,
girl B on the right, short blonde hair, hoodie, hand on chin,
talking, looking at each other, afternoon window light,
```

- 人数只写一次。
- 每人都要外貌锚点与位置；画面出现第三人却没写人数，模型会自己加人。
- 3 人以上不要 `close-up`；4 人以上不要 `cowboy shot`。

---

## 八、负面基线（已过滤 NSFW）

执行时默认输出「必需层 + 按需层 + 兽人排除层（画人类时）」；画兽人时删掉兽人排除层。

**必需层（模型卡原串去掉首项）**

```text
worst quality, old, early, low quality, lowres, signature, username, logo,
bad hands, mutated hands
```

**按需层**

```text
bad anatomy, bad proportions, extra fingers, fewer fingers, fused fingers, deformed,
cropped, blurry, out of focus, jpeg artifacts, watermark, text, multiple views
```

**兽人排除层（只在画人类 / 动漫角色时加）**

```text
mammal, anthro, furry, ambiguous form, feral, semi-anthro
```

画兽人题材时把这六个词全部删掉——它们会把你要的主体一起压掉。

**风格封印（按需）**：`3d, realistic, photo`（要纯二次元时）；`monochrome, greyscale`（要彩色时）。

**已过滤 NSFW 项**：模型卡负面首项 `nsfw` 已移除；语料里的 `loli`、`child`、`cleavage` 等词条替换为 `bad proportions`、`distorted face`、`unnatural body`。本 skill 负面基线不含任何 NSFW 词条。

---

## 九、自检清单（交付前逐条过）

1. 版本确认：eps 还是 v-pred？v-pred 是否开了 v_prediction + zsnr、是否用了 Euler？
2. 起手串完整：`masterpiece, best quality, newest, absurdres, highres, safe`。
3. 圆括号是否转义成 `\(...\)`（角色名、画师名里的括号）。
4. 日期标签与想要的年代一致（`newest` ≠ 90 年代）。
5. 人数只写一次，每人有外貌锚点。
6. 画师只用了括号权重或纯名，没有 `{}` / `N::` / `artist:` 之外的 NAI 语法。
7. 画人类时负面里有兽人排除层；画兽人时已删掉。
8. 负面无 NSFW 词条。

---

## 十、完整示例

### 示例 1｜单人 · eps 1.1

```text
masterpiece, best quality, newest, absurdres, highres, safe,
1girl, solo, frieren \(sousou no frieren\), elf ears, long white hair, twin braids, green eyes,
white robe, black cape, gold trim, holding staff,
standing in a snowy forest, looking at viewer, calm expression,
upper body, three-quarter view, from slightly below,
cold blue light from behind the trees, falling snow, rim light on hair,
(wlop:0.8), (ask \(askzy\):1.1), detailed background
```

负向：

```text
worst quality, old, early, low quality, lowres, signature, username, logo, bad hands,
mutated hands, bad anatomy, extra fingers, watermark, text, multiple views,
mammal, anthro, furry, ambiguous form, feral, semi-anthro
```

参数：eps 1.1 / Euler a / 27 步 / CFG 5.5 / 832×1216

---

## 十一、常见错误

| 错误 | 后果 |
| --- | --- |
| v-pred 用默认 eps 参数 | 糊图 / 灰图 |
| v-pred 用 Euler a 以外的采样器 | 画面崩坏（官方明说） |
| 角色名括号不转义 | 被当权重吃掉，角色变味 |
| 写 `masterpiece` 想要古早画风 | 被拉回热门风格 |
| 画兽人还留着 `anthro, furry` 负面 | 主体被压掉 |
| 照抄 NAI 的 `{}` / `N::tag::` | 语法不存在，白写 |
| 画师串从光辉直接搬过来不重调 | 不同 merge 响应差别大，画风跑偏 |

---

## 十二、来源

- 1.0 模型卡：https://huggingface.co/Laxhar/noobai-XL-1.0
- 1.1 模型卡：https://huggingface.co/Laxhar/noobai-XL-1.1
- V-Pred 1.0 模型卡：https://huggingface.co/Laxhar/noobai-XL-Vpred-1.0
- 官方 ControlNet：https://civitai.com/models/929685
- 相关 skill：`skills/illustrious/SKILL.md`（同源标签体系，参数不同）
