# Anima · Base 提示词 SKILL v5（Hybrid 混合写法）

> 适用：Anima / Anima Base（2B DiT + Qwen3-0.6B 文本编码器）。Aesthetic 档的差异见文末。
> 本文件由 v4 `Anima.md` + `AnimaAesthetic.md` 升级而来，重写为 Hybrid 双段结构：**标签定结构，自然语言定氛围**。负面已过滤 NSFW 项。

---

## 一、模型参数默认值

| 参数 | 默认 | 区间 | 说明 |
| --- | --- | --- | --- |
| 采样器 | `er_sde` | `er_sde` / `euler_a` / `dpmpp_2m_sde_gpu` | er_sde 稳；euler_a 细线柔 |
| 步数 | 32 | 30–50 | 低于 28 细节不收敛 |
| CFG | 4.5 | 4–5（群内经验可到 6） | Anima 对 CFG 的容忍度比同代高 |
| 分辨率 | 1024² | 832×1216 / 1216×832 / 1536² | 总像素 1M 附近 |
| 调度器 | normal | normal / beta57（需 RES4LYF） | beta57 出油画质感 |
| 权重语法 | 以词序为准 | `(tag:1.3)` 视前端而定 | **社区分歧**：官方与多数个人 skill 主张权重基本无效、字段顺序即隐式权重；若你的前端确实解析权重，强调从 1.3 起、1.2 几乎无感 |

**token 预算**：普通分支 512 token（含 tag + NL + 标点 + 换行）。这是硬预算，不是建议值。

---

## 二、Hybrid 双段结构

```text
[标签段]  人数 → 角色 → 外观 → 服装 → 动作 → 镜头 → 场景 → 光
（空行）
[自然语言段] 一句一义，2–3 句，写标签写不出的关系、占比、接触、光向
```

| 段 | 负责什么 | 不该放什么 |
| --- | --- | --- |
| 标签段 | 离散事实：谁、几个人、什么颜色、什么机位、什么场景 | 长从句、因果关系、心理活动 |
| NL 段 | 关系与状态：谁在左、谁碰谁、占画幅多少、光从哪来、正在发生什么 | 重复标签、质量词、负面句 |

分工判据：**用词说得准的写标签，用句子才说得准的写句子。** 机位高低必须用标签（`from below`），句子推不动；「她侧过身，把脸埋进他肩窝」只能用句子。

---

## 三、标签段槽位

| # | 槽位 | 例 | 可省略 |
| --- | --- | --- | --- |
| 1 | **人数** | `1girl, solo` / `2girls` / `no humans` | 否 |
| 2 | 角色名 | `ui (blue archive)`、`name-original` | 是（无角色时） |
| 3 | 外观 | `short hair, blue hair, blue eyes, parted bangs, ahoge` | 否 |
| 4 | 服装 / 道具 | `raincoat, pleated skirt, hair flower, transparent umbrella` | 否 |
| 5 | 动作 / 表情 | `standing, outstretched arm, looking at viewer, light smile` | 否 |
| 6 | 镜头 / 构图 | `full body, from below, dutch angle` | 否 |
| 7 | 场景 / 天气 / 光 | `outdoors, day, cumulonimbus cloud, backlighting, lens flare` | 推荐 |
| 8 | 画师 | `@画师名`，≤2 位 | 推荐 |
| 9 | 质量 | `masterpiece, best quality`（Aesthetic 档删 `score_*`） | 是 |

**写法纪律**：全小写、**逗号后必须有一个空格**、下划线换空格。同一元素全串只出现一次。

逗号后的空格不是排版习惯：Anima 的文本编码器是 Qwen3-0.6B，词表里有 `",a"` 这种「逗号+字母」合并 token，`masterpiece,best quality` 与 `masterpiece, best quality` 会被切成不同的 token 序列，社区实测后者更稳（HuggingFace Anima #184）。

**括号要转义**：角色名、画师名里的括号写成 `ui \(blue archive\)`、`ask \(askzy\)`。ComfyUI / Forge / A1111 都把 `(…)` 当权重语法，不转义等于给名字加了 1.1 倍权重、还把括号吃掉；本项目自带的 tagger 管线可以直接写原括号，两者不冲突，**默认按转义写更安全**（Anima #171）。

**权重不是主要手段**：Anima 的注意力基本由词序决定，标签越靠前影响越大。想让某个元素压过别人，先把它前移、拆成更具体的标签，最后才考虑加权重。`(tag:1.2)` 在多数前端里只是多一点噪声；确实要写时从 1.3 起。想减弱，直接删词比降权可靠。

**不认的语法**：`{}`、`N::tag::`、`artist:`、`source#` / `target#` —— 这些分别属于 NAI 与别的体系，Anima 不解析，写了只会变成字面文本。上面那行 `(tag:1.3)` 是唯一可能生效的权重写法，且优先级排在词序之后。

**服装要写全**：角色的默认 / 官方服装不会因为写了角色名就稳定出现——标签给少了会掉到别的衣服，写全了才复现。群里和 HuggingFace 讨论区（Anima #206）都是同一个结论。

**权重与 `@` 的位置**：真要加权时，`@` 放在括号里，写成 `(@artist:1.1)`；写成 `@(\(artist:1.1\))` 会把 `@` 隔在权重外。自然语言段里的权重同样生效（`out of (focus food stalls:2) in the background`），但优先级仍低于词序（Anima #164）。

---

## 四、自然语言段：7 槽位 + P 规则

### 4.1 七槽位（按顺序，可跳）

| 槽 | 内容 | 例 |
| --- | --- | --- |
| S1 风格 / 触发词 | **仅当用户明确给出**才写 | （用户没给就跳过，直接从 S2 开始） |
| S2 主体 | 谁 / 什么，数量 | `A girl with silver hair sits by the window.` |
| S3 外观与着装 | 发 / 眼 / 表情 + 服装，写成关系句 | `She wears a white sundress with thin straps.` |
| S4 姿势与动作 | 含方向、朝向、接触关系 | `One hand rests on the sill, the other holds a cup.` |
| S5 场景与光影 | 环境、时间、光源方向、色调、情绪 | `Afternoon light comes from the left, warm and quiet.` |
| S6 构图与镜头 | 景别、角度、焦点 | `Bust shot, shallow depth of field.` |
| S7 质量补充 | 可选项 ≤2 | `Clean lineart, detailed eyes.` |

### 4.2 P 规则

- **P-1 重要在前**：主体永远第一句。
- **P-2 关系成句**：属性挂到主体上，`a girl with silver hair wearing a white dress`，不写散装罗列。
- **P-3 一句一义**：一个短句一个语义单元，不嵌套歧义从句。
- **P-4 总量**：1–3 句，或 10–25 个元素；>40 元素视为稀释。
- **P-5 禁同义堆砌**：`beautiful / gorgeous / stunning` 只留一个。
- **P-6 禁矛盾**：`indoor` + `outdoor`、`standing` + `sitting` 同时出现即事故。
- **P-7 质量咒语清零**：NL 段不写 `masterpiece / 8k / hdr / ultra detailed / perfect face`。
- **P-8 想要就写正面**：不靠负面塑造画面（粉象效应）。
- **P-9 空间显式**：`left / right`、`facing the viewer / turned away`、`holding / resting on`。
- **P-10 名词动词优先**，形容词节制。
- **P-11 权重只给标签**：NL 段里不写 `(word:1.2)`。

---

## 五、512 token 预算

**计数范围**：标签段 + 空行 + NL 段 + 全部标点。

**超预算三步精简（从上往下做，够了就停）：**

1. **去水词**：`Use very soft, gentle and warm sunlight beaming brightly from the upper left` → `Soft warm light from the upper left.`
2. **合并短句**：两句同主语 → 一句，用现在分词带过。
3. **砍画画废话**：`Keep the character sharp and clear against the background` 这类整句删掉。**必须保住两句**：① 编剧句（正在发生的高光动作）；② 监督句（人物占画幅比例与左右位置）。

标签层超限时的删减顺序：重复标签 → 上位词冗余 → 次要背景装饰 → 非用户指定的低置信项。用户点名的身份、人数、外观、服饰、动作锚点最后动；锚点本身就超限，反问用户保哪个，不要擅自删。

**扩写分支例外**：用户要求「加细节 / 拉满」时走三段式长稿（标签锚点 → 分层细节 → 精炼收尾），不受 512 限制，但也绝不为了省 token 砍句子。

---

## 六、互斥表（Q 规则）

| 互斥对 | 说明 |
| --- | --- |
| `facing the viewer` ↔ `turned away` | 朝向只能一个 |
| `from front` ↔ `from behind` | 机位只能一个 |
| `from above` ↔ `from below` | 俯仰只能一个 |
| `close-up` ↔ `full body` | 景别只能一个 |
| `standing` / `sitting` / `lying` / `running` | 同一主体同一时刻只能一个 |
| `closed eyes` ↔ `looking at the viewer` | 闭眼时不能要求对视 |
| `barefoot` ↔ `wearing shoes` | 脚部状态只能一个 |
| 同部位状态 | `toes curled` ↔ `toes spread` 只留一个；手足细节合计 ≤2 项 |
| 物理兼容 | `underwater` 不配 `smoking`；`rain` 不配 `dry ground` |

**视线默认**：单人图除非明确要背影 / 侧脸 / `from behind`，必须写 `looking at the viewer`。

---

## 七、多人分段（多人时必读）

标签段改为四段式：

```text
[人数] → [全局镜头·场景·光] → [角色 A 段] → [角色 B 段] → [NL：位置与动作归属]
```

- 各角色段按「角色名（可选）→ 外观 → 服装 → 表情姿势」组织，段间空一行。
- 人数只写一次；角色段只写 `girl` / `boy`，不带数字。
- NL 段逐人写位置（`A stands on the left, B behind her`），再写互动句。
- 3 人以上不要 `close-up`；4 人以上不要 `cowboy shot`。

---

## 八、画师控制

- 格式 `@画师名`，带括号原样保留：`@ask (askzy)`。
- 数量 ≤2；超过 2 位会互相打架。
- 画师名查不到也照写——文本编码器吃任意字符串，标签库收录不全。
- 不加画师 = 素底图，这是 base 的既定行为。
- **堆叠不等于融合**：直接并列多个 `@画师`，编码器会各算各的，出来的往往是「平均脸」而不是混合画风；标签超过 10 个时更平（Anima #167 / #161 的实测）。要混合就两个、拉开权重差，或用社区那类挂 cross-attention 的插件。

---

## 九、负面基线（已过滤 NSFW）

**必需层**

```text
worst quality, low quality, blurry, jpeg artifacts
```

**按需层**

```text
bad anatomy, bad hands, extra fingers, missing fingers, fused fingers, deformed, cropped,
chromatic aberration, watermark, text, signature, multiple views
```

**可选层**

| 加什么 | 什么时候 |
| --- | --- |
| `3d, realistic, photo` | 要纯二次元 |
| `sketch, lineart, monochrome` | 要成品彩图 |
| `artist name` | 画师串带偏了，要踢掉风格签名 |
| `armband, halo, mosaic censoring` | 角色出现不该有的配件 |

**总量纪律**：5–12 项，硬上限 15。Anima 不适合长负面（群内实测：长负面会压细节）；同义词 `worst / low / poor quality` 最多留两个。

**质量词到底在干什么**：`masterpiece` 与 `score>6` 的作用不是「提升画质」，而是把**游戏素材、游戏 CG/截图、涂鸦、像素图**这一类往下压（HuggingFace 讨论区 Anima #247 的官方答复）。所以：

- 想要赛璐璐动画质感，质量词可以留；
- 想要厚涂、水彩、手绘稿这类「非成品感」，把 `score_*` 与 `masterpiece` 降下来，否则模型会往「高完成度插画」方向推，笔触被抹平。

**已过滤 NSFW 项**：原始语料里 `nsfw`、`loli`、`child`、`cleavage`、`nipples` 等词条已全部移除，替换为 `bad proportions`、`distorted face`、`unnatural body`。本 skill 负面基线不含任何 NSFW 词条。

---

## 十、Aesthetic 档差异

| 项目 | Base | Aesthetic |
| --- | --- | --- |
| 正向质量 | `masterpiece, best quality, score_7` | 可留 `masterpiece, best quality`，`score_*` 全删 |
| 负向 | 可写 `score_1, score_2, score_3` | **删掉全部 `score_*`** |
| 默认风格 | 很素，靠画师与描述拉起 | 已做过审美微调，默认就不素 |

两档同时勾选时，冲突只有 score 前缀这一处，按 Aesthetic 执行。理由：Aesthetic 是在「高质量图 + 剥掉质量标签」的语料上微调的，再喂 `score_8` 会被推过头，出现过度锐化、油腻高光、构图趋同。

---

## 十一、自检清单（交付前逐条过）

1. 双段齐：标签段 + NL 段；NL 段 1–3 句、每句以句号结束。
2. 标签段顺序对，人数只写一次，无同义堆砌。
3. 互斥对没有同时出现（朝向、机位、景别、姿势、眼睛、脚）。
4. 单人图默认写了 `looking at the viewer`（除非要背影）。
5. 多人：每人有外貌锚点与位置，NL 段写了动作归属。
6. token 计数 ≤512；超了按三步精简法砍过，且编剧句与监督句都还在。
7. 画师带 `@`，≤2 位；要加权时写成 `(@名字:权重)`。
8. 负面 5–12 项，无 NSFW 词条，无同义词堆叠。
9. 每个逗号后都有一个空格；角色名/画师名里的括号已转义成 `\(...\)`。

---

## 十二、完整示例

### 示例 1｜单人 · 标准 Hybrid

```text
1girl, solo, name-original, long silver hair, red eyes, white sundress, thin straps,
sitting by the window, holding a small bouquet, looking at the viewer, light smile,
bust shot, from side, shallow depth of field,
indoors, afternoon, soft light from the left, warm tone, clean lineart, detailed eyes

A silver-haired girl with red eyes and a gentle smile sits by the window, looking at the viewer.
She wears a white sundress with thin straps and holds a small bouquet against her chest.
Soft afternoon light comes from the left, warming her hair and the windowsill.
Bust shot, shallow depth of field, clean lineart.
```

负向：

```text
worst quality, low quality, blurry, jpeg artifacts, bad anatomy, bad hands, extra fingers, text, watermark
```

### 示例 2｜双人 · 分段 + 关系句

```text
2girls, upper body, from side, eye-level, classroom after school, window light from the left,

long black hair, glasses, cardigan, holding a book, talking, slight smile,
short blonde hair, hoodie, hand on chin, listening, soft smile, looking at another,

warm afternoon, dust in the air, soft shading, detailed background

Two girls talk in an empty classroom. The black-haired girl on the left holds a book and speaks
with a slight smile, while the blonde girl on the right leans on her hand and listens.
Window light comes from the left and catches the dust in the air between them.
Upper body, from side, shallow depth of field.
```

负向：

```text
worst quality, low quality, blurry, jpeg artifacts, bad anatomy, bad hands, extra fingers,
fused fingers, text, watermark, multiple views
```

### 示例 3｜场景主导（人物小）

```text
no humans, wide shot, from above, snowy mountain valley, frozen lake, dead trees,
low sun, long shadows, cold blue palette, mist, ultra detailed background

An empty frozen valley at dusk seen from a high ridge, a pale sun low behind the peaks.
Long shadows stretch across the ice and the dead trees stand in rows.
A thin mist drifts over the lake and the whole scene stays cold and silent.
Wide shot, from above, cold blue palette.
```

负向：

```text
worst quality, low quality, blurry, jpeg artifacts, bad anatomy, watermark, text, signature
```

---

## 十三、常见错误

| 错误 | 后果 |
| --- | --- |
| 只写标签不写 NL | 画面空、没有叙事，表情与动作僵硬 |
| 只写 NL 不写标签 | 机位失控，人物占比乱 |
| NL 段写成负面句 | 反向激活不想要的东西 |
| `upper body` 又写膝脚 | 画面裁切矛盾，肢体乱长 |
| 3 人以上用 `close-up` | 挤成一团 |
| 负面写 30 项 | 细节被压平 |
| 靠权重调强弱 | 词序才是主手段，权重多半只添噪声 |
| 只写角色名不写服装 | 默认服装不稳，会掉到别的衣服 |
| Aesthetic 档还写 `score_8` | 过度锐化、油腻高光 |

---

## 十四、来源

- 模型卡：https://huggingface.co/circlestone-labs/Anima
- 自然语言 7 槽位与 P/N 规则：https://github.com/Nana7mi0721/anima-prompt-naturalang
- 标签 + 自然语言混合、多人分段、512 token 预算：https://github.com/Vincent54122/anima-tagger
- 群内参数经验：`资料文件夹/chats/全部消息_纯文本.txt`（2026-10-23 zabei72 谈负面长度与 CFG）
- 群聊语料拆解：`资料文件夹/群聊语料拆解报告.md`
