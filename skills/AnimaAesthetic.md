# Anima · Aesthetic 差异卡

与 `Anima · Base` 规范一起使用时，以本卡为准覆盖以下条款。

## 只改这几条

| 项目 | Base | Aesthetic |
| --- | --- | --- |
| 正向质量前缀 | `masterpiece, best quality, score_7, safe` | 可留 `masterpiece, best quality,`，`score_*` 全删 |
| 负向 score | 可写 `score_1, score_2, score_3` | **删掉全部 `score_*`** |
| 默认风格 | 很素，靠画师和描述拉起 | 已经过审美微调，默认就不素 |

其余规则照旧：标签小写、空格分隔、画师加 `@`、权重从 `(tag:1.5)` 起、多人逐个绑定外貌、自然语言至少两句。

## 为什么删 score

Aesthetic 是在「高质量图 + 剥掉质量标签」的语料上微调的。再喂 `score_8` / `score_9`，模型会被推过头，画面出现过度锐化、油腻高光、构图趋同那类「slop」特征。

后处理也会在 `anima_aesthetic` 档位下自动过滤 `score_N` 标签，但正负提示词里一开始就别写更干净。

## 与 Base 同时勾选时

两份 skill 会一起注入。冲突只有一处：score 前缀。按本卡执行 —— 不写 score。

## 自检

1. 正向与负向都没有 `score_` 开头的标签。
2. 没有因为「Aesthetic 不需要质量词」而把 `masterpiece, best quality` 也删光——这两个留着无妨。
3. 画师仍带 `@`。
