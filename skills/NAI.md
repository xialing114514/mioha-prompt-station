# NovelAI Diffusion 提示词规范（V4 / V5）

## 模型定位

- NovelAI 自研扩散模型，V4 起完全重写。分 Full 与 Curated 两条产品线，V5 是当前主力。
- 强项：动漫角色一致性、画师风格串、标签 + 自然语言双吃、multi-character 分区。
- 弱项：写实照片、精确文字（V5 有改善，仍不稳）、超长 prompt（有上下文上限）。

## 版本差异（先确认用户选的是哪版）

| 版本 | 默认质量标签（Add Quality Tags 开启时） | 位置 |
| --- | --- | --- |
| **V5 Full / V5 Curated** | Light：`very aesthetic, amazing quality`；Standard：`very aesthetic, masterpiece` | prompt 末尾 |
| V4.5 Full | `location, very aesthetic, masterpiece` | 末尾 |
| V4.5 Curated | `location, masterpiece, no text, -0.8::feet::, rating:general` | 末尾 |
| V4 Full | `no text, best quality, very aesthetic, absurdres` | 末尾 |
| V4 Curated | `rating:general, amazing quality, very aesthetic, absurdres` | 末尾 |
| V3 及更早 | `best quality, amazing quality, very aesthetic, absurdres` | 末尾；V2 在开头 |

V5 默认走第一行。**用户在界面里开了 Add Quality Tags 的话，正向不用自己再写一遍质量词**，写了等于双份。

## 输出契约

| 字段 | 内容 |
| --- | --- |
| `finish.positive` | 标签数组或带权重的标签串 |
| `finish.negative` | 负向标签 |
| `finish.texts` | 画面文字（V5 渲染能力有限，慎用） |

## 标签顺序

**重要内容放前半段。** 越靠前，权重越高，这是 NAI 与 SDXL 差异最大的一点。

推荐结构：

```
[质量/年份/安全] [人数] [角色+作品] [画师] [外观/服装] [动作/表情] [构图/镜头] [场景]
```

`artist:` 串建议紧跟角色之后，太靠后容易被稀释。

## 权重语法

两套并存，互不冲突：

**括号法（V2 起，全局可用）**

- `{tag}` 权重 ×1.05，`{{tag}}` ×1.1025，嵌套继续乘。
- `[tag]` ÷1.05，`[[tag]]` 继续除。
- 可在 Undesired Content 里用，但方向反转：`{tag}` = 更想避开。

**数值法（V4 起）**

- `1.5::rain, night ::` 把中间一段抬到 1.5。
- `0.5::coat ::` 压低到 0.5。
- 末尾一个 `::` 自动闭合所有括号，不用数括号。
- V4.5 起支持负数：`-1::hat ::` 让帽子消失，`-2.5::flat color ::` 拉回细节。负数值适合点杀，不适合替代大段 UC。

## 画师

- 格式 `artist:画师名`，例如 `artist:ask (askzy)`、`artist:anmi`。
- 多个画师用逗号并列，2–4 个是常用区间；超过 4 个会互相打架。
- 画师名查本地 NAI5 画师词典（18,366 条）比猜更靠谱。
- 已经写了 `artist:` 前缀时，后处理不会再重复加。

## 质量与年份

- V5 Full 正向质量：`very aesthetic, amazing quality`（轻）或 `very aesthetic, masterpiece`（标准）。
- 年份/时代：`year 2025`、`newest`、`recent`、`old`。
- 安全层：`rating:general`、`rating:sensitive`、`rating:nsfw`、`rating:explicit`。V4/V5 认这套，比 `safe` 更稳。

## 查词注意

- 安全层优先用 `rating:general` / `rating:sensitive` / `rating:nsfw` / `rating:explicit`。Danbooru 里另有 `safe (container)`（保险箱），写成裸 `safe` 再交给查词工具容易串味。
- 画师名查不到也照写 `artist:原名`。本地画师词典是抽样收录，查空不代表不存在。
- 负向按用户的否定项对应，不要因为查了一个词就把同族物件一并塞进去。

## 负面词（Undesired Content）

直接用官方预设，按模型对号：

**V5 Full · Light**

```
lowres, bad hands, bad anatomy, artistic error, sepia, white haze, worst quality, very displeasing, jpeg artifacts, 0::ai-generated::
```

**V5 Full · Heavy**

```
lowres, artistic error, film grain, scan artifacts, worst quality, bad quality, jpeg artifacts, very displeasing, chromatic aberration, dithering, halftone, screentone, multiple views, logo, too many watermarks, negative space, blank page
```

**V4.5 Full · Heavy**

```
lowres, artistic error, film grain, scan artifacts, worst quality, bad quality, jpeg artifacts, very displeasing, chromatic aberration, dithering, halftone, screentone, multiple views, logo, too many watermarks, negative space, blank page
```

**V4 Full · Light**

```
blurry, lowres, error, worst quality, bad quality, jpeg artifacts, very displeasing
```

界面里选了 UC 预设时，负向框不要再堆一遍同样的词。

## 生成参数

| 参数 | 值 |
| --- | --- |
| 步数 | 28（Opus 档 ≤28 步不耗 Anlas） |
| Prompt Guidance | 5–6（V3 及以后）；低一点更绘画化，高一点更锐利 |
| 采样器 | Euler Ancestral |
| SMEA | 开（DYN 也行，Ancestral 系建议开） |
| 分辨率 | 按界面预设（832×1216 / 1216×832 / 1024×1024），超范围耗 Anlas |

Prompt Guidance 太高会出现边缘过饱和，可开 Decrisper 或 Prompt Guidance Rescale 压一下。

## 自然语言与标签混用

- V4/V5 的文本编码器吃自然语言，可以写完整句子，不必全标签。
- 混用小技巧：句子负责关系（"她把手搭在他肩上，侧过脸看向镜头"），标签负责属性（`black hair, blunt bangs, school uniform`）。
- 上下文有上限，标签重复和同义词堆叠会浪费额度。看界面底部的上下文条。

## 常见错误

| 错误 | 后果 |
| --- | --- |
| 用 SD 的 `(tag:1.2)` 语法 | NAI 不认，原样当文字吃掉 |
| 开 Add Quality Tags 又手写质量词 | 双份质量词，画面发糊发腻 |
| 画师不带 `artist:` | 被当普通标签，风格不生效 |
| 重要内容写在后半段 | 被稀释，画面不听话 |
| 负向里堆 80 个词 | 互相抵消，还可能连带压掉要的东西 |
| 用 `masterpiece, best quality` 套 V5 | 能生效但不如 `very aesthetic` 贴模型 |

## 自检

1. 版本确认过，质量词与位置对得上。
2. 画师全是 `artist:` 格式，数量 ≤4。
3. 权重括号闭合，或用 `::` 收尾。
4. 负向没有和 UC 预设重复。
5. 关键元素在前半段。
6. 没有出现 `(tag:1.2)` 这种 SD 语法。

## 来源

- https://docs.novelai.net/en/image/basics/
- https://docs.novelai.net/en/image/strengthening-weakening/
- https://docs.novelai.net/en/image/undesiredcontent/
- https://docs.novelai.net/en/image/qualitytags/
- https://docs.novelai.net/en/image/stepsguidance/
- 本地 NAI5 画师词典（18,366 条，q_nai5.json）
