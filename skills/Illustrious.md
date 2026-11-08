# 光辉 · Illustrious XL 提示词规范（SDXL · 3B）

## 模型定位

- OnomaAI Research 出品，从 Kohaku XL Beta 5 续训的 SDXL 架构模型，3B 参数。
- 训练集：Danbooru2023 大规模标注数据及其变体，覆盖角色、画风、构图、画师风格。
- 强项：标签精准度、角色还原、大量画风储备、对外来 LoRA 兼容好。
- 弱项：**没有默认风格**。不写画风/画师/媒介，出来的是素底图。自然语言理解强于同期 SDXL，仍以标签为主。

## 输出契约

| 字段 | 内容 |
| --- | --- |
| `finish.positive` | Danbooru 标签数组，逗号分隔 |
| `finish.negative` | 负向标签数组 |
| `finish.texts` | 画面文字（SDXL 渲染弱，通常留空） |

## 标签格式

- 全小写 Danbooru 标签，逗号分隔，不要下划线（`long hair` 优于 `long_hair`）。
- 标签求准不求多：25–60 个标签是常规区间，语义重复的（`beautiful` + `pretty`）直接删。
- 画师写**纯名字**，不要 `@` 前缀。后处理会把 `@` 剥掉，写了也白写。
- 作品名里的圆括号要转义成 `\(...\)`，例如 `kaguya-sama wa kokurasetai \(series\)`，否则会被权重语法截断。
- LoRA 触发词跟 LoRA 一起用，位置放在角色标签附近，权重按 LoRA 卡说明。
- 不要用 Pony 的 `score_9, score_8_up, score_7_up`，也不要用 Anima 的 `score_N`。两套评分体系在这里都不成立。

## 质量标签

官方支持这一套：

```
worst quality, bad quality, average quality, good quality, best quality, masterpiece
```

正向用 `best quality, masterpiece` 或 `good quality, best quality` 就够。堆到 6 个质量词只会互相稀释。

`masterpiece (quality)` 这种带括注的写法是官方示例，等价于 `masterpiece`，二选一。

## 构图与镜头

官方明确警告：**别堆互相冲突的构图标签**。`close-up` + `full body` + `cowboy shot` 同时出现，模型会自己挑一个，画面不可控。

选一个主构图，其余用补充描述：

| 需要 | 用 |
| --- | --- |
| 特写 | `portrait`、`upper body` |
| 半身到膝 | `cowboy shot` |
| 全身 | `full body` |

视角同理，`from above` / `from below` / `from behind` 只留一个。透视标签（`dutch angle`、`fisheye`、`vanishing point`）一次一个。

## 查词注意

- **`safe` 不要查词。** Danbooru 里有 `safe (container)`（保险箱），查 `safe` 会命中它，照抄就把保险箱画进图里。安全层直接用 `safe`、`sensitive`、`nsfw`、`explicit`，或用 `rating:general` / `rating:nsfw`。
- **画师查不到也照写纯名。** 标签库收录不全，查空只代表没收录。用户点名的画师写进去，风格由文本编码器自己处理。
- 负向按用户否定项一一对应，不做联想扩展。

## 画师与风格

- 画师名直接写：`ask (askzy)`、`wlop`、`anmi`。
- 风格词优先用 Danbooru 认的：`watercolor (medium)`、`oil painting`、`sketch`、`monochrome`、`retro artstyle`、`official art`。
- 不写风格 = 素底图。这是 base 模型的既定行为，不是 bug。
- 衍生底模（NoobAI、各类 merge）有各自的触发词与推荐参数，以它的模型卡为准，别把 v0.1 的经验套过去。

## 负面词

官方示例可直接用：

```
worst quality, bad quality, low quality, lowres, displeasing, very displeasing,
bad anatomy, bad hands, scan artifacts, monochrome, greyscale, signature,
twitter username, jpeg artifacts, multiple views, comic, 2koma, 4koma,
extra digits, fewer digits
```

按场景裁剪：要黑白稿就去掉 `monochrome, greyscale`；要漫画分格就去掉 `comic, 2koma, 4koma`；要彩色就保留 `monochrome`。

负面只排除「不想要的」和「缺陷」，不要否定用户点名的画风。

## 生成参数

| 参数 | 值 |
| --- | --- |
| 采样器 | Euler a |
| 步数 | 20–28 |
| CFG | 5–7.5 |
| 分辨率 | 832×1216 / 1216×832 / 1024×1024 / 1536×1024（SDXL 原生 1024² 基准） |
| 高分辨率修复 | 需要时开，denoise 0.35–0.5 |

CFG 低于 5 画面会发灰，高于 8 容易过饱和发脆。

## 与其它模型的分界

| 项目 | 光辉 | Anima | NAI | MOSS |
| --- | --- | --- | --- | --- |
| 画师前缀 | 无 | `@` | `artist:` | `@`（标签写法） |
| 评分标签 | 不用 | `score_N` | 不用 | 可选 |
| 权重语法 | `(tag:1.2)` | `(tag:2)` 起 | `{}` / `[]` / `N::` | 基本不用 |
| 主输入 | 标签 | 标签+句子 | 标签+句子 | 整段描述 |

跨模型复用 prompt 时，这四行是必须改的地方。

## 常见错误

| 错误 | 后果 |
| --- | --- |
| 不写画风/画师 | 素底图，用户以为模型坏了 |
| 画师加 `@` | 被后处理剥掉 |
| 构图标签互相打架 | 画面随机裁切 |
| 用 Pony 评分词 | 无效，占 token |
| 负向写了 100 个词 | 关键词被稀释 |
| 把 v0.1 参数套到 NoobAI 上 | 出图发灰或崩坏 |

## 自检

1. 人数标签对，角色名带作品名（跨作品混搭时尤其要写）。
2. 主构图只有一个，视角只有一个。
3. 画师纯名，无 `@`、无 `artist:`。
4. 没有 `score_N` 系列。
5. 负向与画面需求不冲突（黑白稿别否 monochrome）。
6. 写了风格或画师，画面不会素。

## 来源

- https://huggingface.co/OnomaAIResearch/Illustrious-xl-early-release-v0
- 技术报告：https://arxiv.org/abs/2409.19946
