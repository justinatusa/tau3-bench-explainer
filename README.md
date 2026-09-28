# τ³-bench 零背景说明书

这是一份中文说明书，给没接触过 τ-bench、Sierra 或客服 agent 评测的人用。它解释 Sierra 的 **τ³-bench** 在测什么、程序怎么分成几块、一次仿真怎么往下走、要装什么、以及官方文档里的运行命令。

本仓库只有说明文字。评测代码、题目和业务数据都不在这里。

官方代码在 GitHub 仓库 [sierra-research/tau2-bench](https://github.com/sierra-research/tau2-bench)（MIT）。产品名是 τ³-bench（读作 tau three）。仓库名仍是 `tau2-bench`。Python 包名和命令仍是 `tau2`。这三件名字的差别见 [00-readme.md](docs/tau3-bench-explainer/00-readme.md)。

官方题目在该仓库的 `data/tau2/domains/`（`airline`、`retail`、`telecom`、`mock`、`banking_knowledge`）。`data/simulations/` 是一次运行写出来的结果，不是题目。Hugging Face 上的 `HuggingFaceH4/tau2-bench-data` 是 τ² 时期的镜像，不是 τ³ 的官方 Dataset。

## 阅读顺序

| 顺序 | 文档 | 读完能回答 |
|------|------|------------|
| 0 | [00-readme.md](docs/tau3-bench-explainer/00-readme.md) | 这套东西是什么；为什么仓库还叫 tau2-bench；先记住哪些钉 |
| 1 | [01-what-is-tau3.md](docs/tau3-bench-explainer/01-what-is-tau3.md) | 测什么；五个域；文字和语音；τ / τ² 只作对照时差在哪 |
| 2 | [02-architecture.md](docs/tau3-bench-explainer/02-architecture.md) | simulation、orchestrator、agent、user、environment、domain、evaluator 怎么拼 |
| 3 | [03-agent-loop.md](docs/tau3-bench-explainer/03-agent-loop.md) | 半双工文字和全双工语音各怎么转；tick；工具调用；何时停 |
| 4 | [04-dependencies.md](docs/tau3-bench-explainer/04-dependencies.md) | Python、uv extras、密钥、模型、检索后端、要连的网络 |
| 5 | [05-unnamed-concepts.md](docs/tau3-bench-explainer/05-unnamed-concepts.md) | dual-control、pass^k、DB 哈希、NL assertion、retrieval-config、AudioTap、hallucination retry |
| 6 | [06-runbook.md](docs/tau3-bench-explainer/06-runbook.md) | 官方文档里的安装和 `tau2 run` 命令 |
| 证据 | [99-evidence.md](docs/tau3-bench-explainer/99-evidence.md) | 每条关键事实对应的上游路径、tag、URL |

## 版本钉

封存日期：2026-09-28（Asia/Shanghai）。下表日期来自封存说明和发布页，不是本仓库的提交日期。

| 名字 | Git 对象 | 日期 | 是什么 |
|------|----------|------|--------|
| `main` | `b7ea9074c1cba482b30687fecdb5c8425fd6f619` | 提交时间 `2026-09-17T21:57:12Z`（CST 2026-09-18 05:57:12） | 本说明书所依据的代码摘录。提交说明是 `feat(voice): pass default en-US language hint in Gemini input_audio_transcription (#544)` |
| tag `v1.0.0` | `17e07b1da2bbc0cadfddeea36412686e0604127b` | 提交 `2026-03-18T07:13:53Z`；Release 发布 `2026-03-18T07:14:12Z` | τ³-bench 1.0.0：Voice、Knowledge、Task Quality。`CHANGELOG.md` 这一节标题写成了占位符 `2026-MM-DD`。tag `voice-user-sim-v1.0` 指向同一提交 |
| tag `v1.0.1` | 代码提交 `fc0055dc4e0a316c3f83133267fbd6faaa770992` | 提交 `2026-07-16T22:24:56Z`；Release 发布 `2026-07-22T21:22:42Z`；`CHANGELOG.md` 标题写 `2026-07-15` | `banking_knowledge` 评分修复。这个域上，`< 1.0.1` 的分数不能和 `>= 1.0.1` 的分数比。其他域不受影响 |

**`b711c1ead46f55111bf765cf44d5da8bacc2d28c`**：是 v1.0.1 这个 annotated tag 对象自己的 SHA。不是代码提交。代码提交仍是 `fc0055dc…`。

同一份封存摘录里，`pyproject.toml` 的 `version` 仍是 `1.0.1`。这是包版本号字符串，不是「main 停在 v1.0.1 tag」。main 比该 tag 新，至少包含上面这条语音提交。

官方题目路径是 `data/tau2/domains/<域名>/`。`banking_knowledge` 是 **97** 道题、**698** 篇文档。数据摘要里，mock 的 `tasks.json` 是 **10**（语音配置 **9**），telecom 的 `tasks.json` 是 **2285**（`split.base` **114**，`small` **20**）。telecom 排行榜默认用哪一份划分：UNKNOWN。airline 与 retail 的**当前** `tasks.json` 条数也是 UNKNOWN：修订博客只把 **50** 和 **114** 写成这两个域**原先**的规模，不要把它们写成钉上已经数过的数组长度，也不要把 telecom 的 114 和 retail 的 114 当成同一批题。详见 [01-what-is-tau3.md](docs/tau3-bench-explainer/01-what-is-tau3.md)。

## 官方入口

- 代码：<https://github.com/sierra-research/tau2-bench>
- 排行榜（当前 README 与提交文档）：<https://taubench.com>
- v1.0.0 Release：<https://github.com/sierra-research/tau2-bench/releases/tag/v1.0.0>
- v1.0.1 Release：<https://github.com/sierra-research/tau2-bench/releases/tag/v1.0.1>
- Sierra 公告：<https://sierra.ai/blog/bench-advancing-agent-benchmarking-to-knowledge-and-voice>
- 题目修订说明：<https://taubench.com/blog/tau3-task-fixes.html>
- 知识域说明：<https://taubench.com/blog/tau-knowledge.html>

`CHANGELOG.md` 文末和 0.2.0 一节写的是 <https://tau-bench.com>。完整封存的 PIN 把 `taubench.com` 当作当前主站，并注明旧文档有时写 `tau-bench.com`。包里没有跳转证明。

τ-Knowledge 论文是 arXiv [2603.04370](https://arxiv.org/abs/2603.04370)，τ-Voice 论文是 arXiv [2603.13686](https://arxiv.org/abs/2603.13686)。封存的是摘要，不是 PDF。τ 与 τ² 只在说明书的对照节出现。缺证据的地方写 UNKNOWN。证据路径见 [99-evidence.md](docs/tau3-bench-explainer/99-evidence.md)。
