# 基准测试：直接要 HTML vs Answer me with HTML

同一个问题、同一个模型，各出一页。每格是 3 次运行的中位数（Claude Sonnet 5.5）。我们在两种环境下运行，因为成本取决于环境加载了多少上下文。

数据直接来自 `claude -p --output-format json`：`modelUsage.outputTokens`、`duration_ms` 和 `total_cost_usd`。

## 模型写了什么（固定）

语料是下面普通环境运行中的 9 页手写 HTML 和 9 份 Markdown 稿件。它是一个[发布版本下载](https://github.com/QingYunA/answer-me-with-html/releases/download/v0.4.14/bench-corpus-2026-10-07.zip)，不纳入 git，因为插件安装会复制整个仓库。[count-tokens.mjs](count-tokens.mjs) 用模型自己的分词器（Claude Sonnet 5.5，无需 API key）统计了它们的 token，并保存在 [corpus/tokens.json](corpus/tokens.json) 中。要重新统计，请将下载内容解压到 `bench/corpus/` 并运行 `node bench/count-tokens.mjs`。`node bench/corpus.mjs` 会报告这些数据：

| | 手写 HTML | Markdown 稿件 |
| :--- | ---: | ---: |
| 平均 token 数 | 4,893 | **612**（少 8.0 倍） |
| SVG 图表 | 47% | 由 CLI 生成 |
| CSS | 15% | 由 CLI 生成 |
| HTML 标签 | 17% | 由 CLI 生成 |
| 正文 | 21% | 稿件 |

语料及其计数不会改变，因此这些数字也不会改变。两次统计对每个文件都得到相同的结果。下面的运行还包括工具调用和结束回复，它们会因运行而异，也会随 Claude Code 版本而变化。

## 普通环境（2026-10-07）

仅使用项目设置的 Claude Code，没有 MCP 服务器，并将本仓库的 skill 安装在工作文件夹中（`BENCH_LEAN=1`）。这里每轮读取约 25,000 个 token 的上下文。

| 主题 | 输出 token | 时间 | 每次回答花费 |
| :--- | :--- | :--- | :--- |
| TCP 握手与断开 | 5,399 → **1,120**（4.8 倍） | 32 秒 → **13 秒**（2.4 倍） | $0.098 → $0.079 |
| Redis vs Memcached | 4,536 → **979**（4.6 倍） | 29 秒 → **12 秒**（2.5 倍） | $0.086 → $0.077 |
| git merge vs rebase | 4,861 → **922**（5.3 倍） | 31 秒 → **11 秒**（2.8 倍） | $0.090 → $0.076 |
| **平均** | 4,932 → **1,007**（4.9 倍） | 31 秒 → **12 秒**（2.6 倍） | $0.091 → **$0.077**（便宜 15%） |

在全部 9 次配对运行中，skill 都更便宜（$0.075 至 $0.097 对 $0.084 至 $0.103）。原始数据行在 [results/lean/results.json](results/lean/results.json)。生成的页面不纳入 git；运行脚本即可获得它们。

2026-10-05 的一次更早运行，使用了更长的 `SKILL.md`，测得输出 token 5,341 → 870，花费 $0.092 → $0.067。运行之间和版本之间的变化就有这么大，所以请将比率视为大致量级。

## 加载很重的环境（2026-10-07）

我们自己的 Claude Code 配置，包含许多插件、规则和 skill，并将 skill 作为插件安装（0.4.13）。这里每轮读取约 86,000 个 token 的上下文。

| 主题 | 输出 token | 时间 | 每次回答花费 |
| :--- | :--- | :--- | :--- |
| TCP 握手与断开 | 8,194 → **1,149**（7.1 倍） | 52 秒 → **14 秒**（3.6 倍） | $0.165 → $0.119 |
| Redis vs Memcached | 8,146 → **1,031**（7.9 倍） | 54 秒 → **14 秒**（3.8 倍） | $0.194 → $0.147 |
| git merge vs rebase | 6,627 → **1,031**（6.4 倍） | 45 秒 → **14 秒**（3.1 倍） | $0.301 → $0.147 |
| **平均** | 7,656 → **1,070**（7.2 倍） | 50 秒 → **14 秒**（3.5 倍） | $0.220 → **$0.138**（便宜 37%） |

在 9 次配对运行中有 8 次 skill 更便宜。原始数据行在 [results/results.json](results/results.json)。

此环境中的花费主要取决于提示缓存。在 18 次运行中有 7 次（4 次普通、3 次 skill）缓存未命中，整个上下文被重新写入：这些运行花费 $0.28 至 $0.54。缓存命中时，普通回答平均花费 $0.159，skill 为 $0.127。2026-10-02 的一次更早运行测得此环境中 skill 贵约 20%。它没有记录缓存 token，因此我们无法判断其有多少次运行未命中缓存。

## 这些数字意味着什么

- **输出 token 和时间大幅下降。** 模型写一份简短的 Markdown 稿件。CLI 写 CSS、版面和每一个 SVG 坐标。
- **花费降得比输出 token 少。** 输出只占账单的一部分，参见[花费去向](#where-the-cost-goes)。加速在每种环境中都成立。
- **页面不同。** 直接要 HTML 会得到一篇更长的文章，文字更多。skill 给出一个紧凑的页面，通常一到两屏。参见 [docs/images/plain-vs-skill.png](../docs/images/plain-vs-skill.png)。

## 花费去向

`total_cost_usd` 与这些价格完全吻合（每一行精确到 $0.0001）：输出每百万 token $10，缓存读 $0.20，缓存写在普通环境中为 $4，在加载很重的环境中为 $2.50（两种环境保留缓存的时间不同）。按这些价格拆分，普通环境下的平均回答花费为：

| | 直接要 HTML | Answer me with HTML | 差异 |
| :--- | ---: | ---: | ---: |
| 输出 | 5,097 token · $0.051（55%） | 1,123 token · $0.011 | −$0.040 |
| 缓存写 | 7,919 token · $0.032 | 12,361 token · $0.049 | +$0.018 |
| 缓存读 | 49,627 token · $0.010 | 95,456 token · $0.019 | +$0.009 |
| 其他输入 | 960 token · $0.001 | 954 token · $0.001 | 0 |
| **总计** | **$0.093** | **$0.081** | **−14%** |

这些是全部 9 次运行的平均值，因此与上面的中位数略有不同。

- **输出占普通回答的 55%。** 如果 skill 写出零个输出 token，它最多也只能省 55%。
- **缓存写增加。** 普通运行在下一轮读取时将它自己写的 HTML 写回缓存。skill 运行则改为写入它的指令（`SKILL.md`，约 4,000 个 token），这更大。在一次会话中，指令会留在缓存里，因此后续页面不应再为此付费。我们尚未测量这一点。
- **缓存读增加。** skill 运行有 4 轮而不是 2 轮，每轮都会再次读取上下文。缓存读很便宜，所以这里只增加约一美分。
- **更短的 `SKILL.md` 花费更少。** 将设置和视频说明移到 agent 仅在需要时读取的文件中，使 skill 在普通环境下的花费从 $0.083 降到 $0.077。页面保持同一类型：3 到 6 个面板，相同的组件，没有写作警告。

## 解释视频

我们自己的加载很重的环境，一个主题（TCP 握手），无配音。普通提示要求一个自包含、自动播放的 3Blue1Brown 风格页面，含标题与图表场景、逐步字幕、播放/暂停和进度条。skill 提示要求 `am video`，使用 3b1b 主题和 `--voice off`。Claude Sonnet 5.5，2026-10-05。

| 运行 | 普通：token / 时间 | `am video`：token / 时间 |
| :--- | :--- | :--- |
| 1 | 27,839 / 202 秒 | 1,564 / 16 秒 |
| 2 | 27,375 / 199 秒 | 1,567 / 19 秒 |
| 3 | 34,402 / 270 秒 | 不适用 |
| **中位数** | **27,839 / 202 秒** | **1,566 / 17 秒** |

- 第 3 次运行来自同一脚本的更早一次运行；那次运行的其余部分遇到限流，因此只保留了这一行。`am video` 一侧有两次运行。
- 输出 token 和时间：少 17.8 倍，快 11.8 倍。
- 我们不报告视频的花费：同一环境不同次运行之间波动太大。
- 所有页面都在 Chrome 中打开过。三个手写页面会自动播放，有分场景、字幕和进度条。`am video` 页面还有镜头聚焦和场景变形，这是普通页面的提示没有要求的。
- 一个主题上的五次运行只是大致量级，不是精确比率。

复现：普通环境用 `BENCH_LEAN=1 node bench/run.mjs sonnet bench/results/lean 3`，视频用 `BENCH_KIND=video BENCH_TOPICS=tcp node bench/run.mjs sonnet bench/results/video 2`。原始数据行在 [results/video/results.json](results/video/results.json)。生成的页面不纳入 git。

## 自己运行

你需要安装 Claude Code 和本 skill。

```bash
node bench/run.mjs sonnet bench/results 3          # 3 个主题 × 2 种方式 × 3 次运行，在你自己的环境中
BENCH_LEAN=1 node bench/run.mjs sonnet /tmp/b 3     # 小上下文：仅项目设置，无 MCP 服务器
BENCH_TOPICS=tcp node bench/run.mjs sonnet /tmp/b 1  # 一个主题，一次运行
```

你自己的环境决定了每轮重新读取多少上下文，因此也决定了花费。每一行在输出 token 旁边记录 `inputTokens`、`cacheWriteTokens` 和 `cacheReadTokens`，因此你可以看到花费来自哪里。

每次运行都在一个全新的临时文件夹中进行。"direct" 运行只能使用 Write 工具；"skill" 运行可以使用 skill 和 Bash。

---

# 基准测试：直接要 HTML vs Answer me with HTML

同一个问题、同一个模型，各出一页。每格是 3 次运行的中位数（Claude Sonnet 5.5，2026-10-07）。上表即结果。

- **模型要写的 token（固定）：** 语料是下面普通环境那次运行的 9 页手写 HTML 和 9 份 Markdown 稿件，可以[下载](https://github.com/QingYunA/answer-me-with-html/releases/download/v0.4.14/bench-corpus-2026-10-07.zip)，不入库。用模型自己的分词器数过 token，结果存在 [corpus/tokens.json](corpus/tokens.json)。`node bench/corpus.mjs` 输出：手写 HTML 平均 4,893 个 token，其中 SVG 占 47%、CSS 15%、标签 17%、正文 21%；稿件平均 612 个 token，少 8.0 倍。语料和计数都不变，这些数字就不变。
- **输出 token 和耗时大幅下降：** 模型只写简短的 Markdown 稿件，CSS、版面和所有 SVG 坐标都由 CLI 生成。
- **普通环境**（`BENCH_LEAN=1`，每轮读约 2.5 万个 token 的上下文）：输出 token 4,932 → 1,007（少 4.9 倍），耗时 31 秒 → 12 秒（快 2.6 倍），花费 $0.091 → $0.077（便宜 15%），9 次配对里 skill 每次都更便宜。
- **加载很重的环境**（我们自己的 Claude Code 配置，每轮读约 8.6 万个 token）：输出 token 7,656 → 1,070（少 7.2 倍），耗时 50 秒 → 14 秒（快 3.5 倍），花费 $0.220 → $0.138（便宜 37%），9 次里 8 次更便宜。这里花费主要看缓存：18 次里有 7 次缓存没命中，整段上下文被重新写入，单次花费 $0.28–0.54。2026-10-02 那次测到 skill 贵约 20%，但那次没有记录缓存数据，无法判断有几次没命中。
- **花费降得比 token 少：** 按 $10/百万（输出）、$4 或 $2.5/百万（缓存写）、$0.2/百万（缓存读）拆开，直接写 HTML 时输出只占花费的 55%，所以即使 skill 一个输出 token 都不写，最多也只能省 55%。skill 多出的花费主要是缓存写：它要载入约 4,000 token 的 `SKILL.md`。把设置和视频的说明拆到按需读取的文件后，普通环境下 skill 的花费从 $0.083 降到 $0.077，页面质量不变。
- **页面风格不同：** 直接要 HTML 得到的是更长、文字更多的长文；用 skill 得到的是一到两屏的紧凑页面。

**解释视频：** 同样的方法，一个题目（TCP 握手），不配音。手写要求是一个自包含、自动播放的 3Blue1Brown 风格页面，含标题与图表场景、逐步字幕、播放/暂停和进度条；skill 一侧用 `am video`，3b1b 主题，`--voice off`。Claude Sonnet 5.5，2026-10-05。

- 输出 token 中位数 27,839 → 1,566（少 17.8 倍），耗时 202 秒 → 17 秒（快 11.8 倍）。
- 手写 3 次、`am video` 2 次。第 3 次手写来自同一脚本的更早一次运行，那次其余的运行遇到限流，只保留了这一行。
- 视频不报花费：同一种方式在不同次之间波动太大。
- 所有页面都在 Chrome 里打开检查过：3 个手写页面会自动播放，有分场景、字幕和进度条。`am video` 的页面还有镜头聚焦和场景间的变形，这是手写提示里没要求的。
- 一个题目、5 次运行，只能看大概量级，不是精确倍数。

复现：普通环境用 `BENCH_LEAN=1 node bench/run.mjs sonnet bench/results/lean 3`（原始数据在 [results/lean/results.json](results/lean/results.json)）；视频用 `BENCH_KIND=video BENCH_TOPICS=tcp node bench/run.mjs sonnet bench/results/video 2`。原始数据在 [results/video/results.json](results/video/results.json)，生成的页面不入库。

复现方法见上方 "Run it yourself"。