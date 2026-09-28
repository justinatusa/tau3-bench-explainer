# 证据索引

本说明书依据两份封存。后一份（2026-09-28 17:00–17:03 CST）覆盖前一份：它带有 `DATA_SUMMARY.md`、`cli.py`、`registry.py`、evaluator 入口、编排器与 agent 的 README、五篇论文摘要，以及官方博客的抓取文本。博客的纯文本是从 HTML 粗抽的，数字以 `.html` 与 `.md` 里能对上的句子为准。论文封存的是 arXiv 摘要，不是 PDF 全文。

PIN 写明：`repo/` 下的文件用 `?ref=b7ea9074c1cba482b30687fecdb5c8425fd6f619` 拉取。`config.py` 里 Gemini 转写语言默认 `en-US`，与该提交说明一致。

上游仓库：<https://github.com/sierra-research/tau2-bench>

## 版本钉

| 事实 | 值 | 依据 |
|------|----|------|
| 产品名 τ³-bench，读作 tau three | README 横幅与 “How do you say” 一节 | 仓库 `README.md` |
| 仓库名仍为 tau2-bench | `https://github.com/sierra-research/tau2-bench` | PIN；`pyproject.toml` 的 Repository URL |
| 包名 `tau2`，版本字符串 `1.0.1` | `name = "tau2"`，`version = "1.0.1"` | `pyproject.toml` |
| main | `b7ea9074c1cba482b30687fecdb5c8425fd6f619`，提交 `2026-09-17T21:57:12Z`，说明为 Gemini input_audio_transcription 的 en-US hint（#544） | 完整封存 `PIN.md` |
| tag v1.0.0 | `17e07b1da2bbc0cadfddeea36412686e0604127b`，提交 `2026-03-18T07:13:53Z`，Release `2026-03-18T07:14:12Z` | `PIN.md` |
| `voice-user-sim-v1.0` | 与 v1.0.0 同一提交 | `PIN.md` |
| CHANGELOG 的 1.0.0 日期是占位符 | `## [1.0.0] - 2026-MM-DD` | `CHANGELOG.md` |
| tag v1.0.1 的代码提交 | `fc0055dc4e0a316c3f83133267fbd6faaa770992`，提交 `2026-07-16T22:24:56Z` | `PIN.md` |
| v1.0.1 的 annotated tag 对象 | `b711c1ead46f55111bf765cf44d5da8bacc2d28c`，tagger `2026-07-22T21:22:10Z`，Release `2026-07-22T21:22:42Z` | `PIN.md`。这不是代码 SHA |
| CHANGELOG 的 1.0.1 标题日期 | `2026-07-15` | `CHANGELOG.md` |
| 修复先进入 main 的窗口 | 2026-07-14/15，当时没有改版本号 | `RELEASE_NOTES.md`，Release v1.0.1 |
| pre-v1.0.1 | `b51a6d69e26f0e94a9173e2e80fe8735a8dff650`（简写 `b51a6d6`） | `RELEASE_NOTES.md` |
| 计分相同的更早提交 | `2be6916`，2026-04 | `RELEASE_NOTES.md`，`CHANGELOG.md` |
| 跨版本不可比的范围 | 仅 `banking_knowledge`；`< 1.0.1` 对 `>= 1.0.1` | 仓库 `README.md` 的 2026-07 提示；Release v1.0.1 |

Release 页：

- <https://github.com/sierra-research/tau2-bench/releases/tag/v1.0.0>
- <https://github.com/sierra-research/tau2-bench/releases/tag/v1.0.1>
- <https://github.com/sierra-research/tau2-bench/releases/tag/pre-v1.0.1>

## 官方题目在哪

| 事实 | 依据 |
|------|------|
| 权威位置是仓库内 `data/` | PIN「Dataset (authoritative)」 |
| 域文件在 `data/tau2/domains/<name>/` | `AGENTS.md` |
| `data/` 顶层只有 `tau2` 和 `voice` | `data_tree_summary.json` 的 `top_level_under_data` |
| 非可编辑安装要用 `TAU2_DATA_DIR` | `docs/getting-started.md` |
| `HuggingFaceH4/tau2-bench-data` 是 τ² 时期镜像，不是 τ³ 官方 Dataset | PIN |
| `evalscope/tau3-bench-data` 是 EvalScope 包装，不是主源 | PIN |
| 社区轨迹集（例：`debdootmiitd/tau3-bench-qwen3.6-…`）不是规范题库 | PIN |

## 规模数字

| 数字 | 是什么 | 依据 |
|------|--------|------|
| 10 / 9 | mock 的 `tasks.json` 长度 / 语音 `configs` 长度。是数据摘要里的数，不是 airline/retail 的当前条数 | `DATA_SUMMARY.md` |
| 50 | 修订博客对**原先** airline 域的说法（original airline (50 tasks)）。**不是**当前 `tasks.json` 长度。当前长度 UNKNOWN | `blogs/tau3-task-fixes.md` |
| 114（retail，博客） | 同一篇博客对**原先** retail 域的说法（retail (114 tasks)）。**不是**当前 `tasks.json` 长度。当前长度 UNKNOWN | `blogs/tau3-task-fixes.md` |
| 114（telecom `base`） | 数据摘要里 telecom 的 `split.base`。不是博客里的原 retail，也不是 telecom 的 2285 | `DATA_SUMMARY.md` |
| 2285 | 数据摘要里 telecom 的 `tasks.json`、`split.full`、语音配置。不是代码默认会跑的 `base` | `DATA_SUMMARY.md` |
| 20 | 数据摘要里 telecom 的 `split.small` | `DATA_SUMMARY.md` |
| airline / retail 的 train、test、语音配置 | UNKNOWN。`DATA_SUMMARY.md` 写过 airline train 30 / test 20 / base 50 / 语音 50，retail train 74 / test 40 / base 114 / 语音 114。JSON 正文不在封存包里，本地分析未把这些当成已解析的当前长度 | `DATA_SUMMARY.md` 与本地分析的冲突，见下节 |
| 97 | `banking_knowledge` 任务数。Release / CHANGELOG / 公告写 97 tasks；清单里 97 个 `tasks/task_*.json` | Release v1.0.0；`CHANGELOG.md`；Sierra 公告；`data_tree_summary.json` |
| 缺号 9、11、13、30、42 | 上述任务**文件名**从 001 到 102 的缺口，不是另一套题数。97 个文件与 `tasks.json` 是否逐字节相同：UNKNOWN | `data_tree_summary.json` 的 `task_related_json` |
| 698 | 该域文档数。`documents_count` 与 CHANGELOG / Release / 公告一致。公告还写 21 个产品类、约 19.5 万 token | `data_tree_summary.json`；`CHANGELOG.md`；Sierra 公告 |
| 815 | 该域目录 `file_count`，不是题数，也不是文档数 | `data_tree_summary.json` |
| telecom 榜单划分 | UNKNOWN。文档同时写 `base` 和「全部题目」 | 封存 `INDEX.md`；`docs/leaderboard-submission.md`；`cli.py` 默认 split 名 `base` |
| 27 / 26 | v1.0.0 点名的 airline / retail 修订题数。不是题库大小 | Release v1.0.0；`CHANGELOG.md` |
| 20+ | changelog 对 banking 修订条目的约数 | `CHANGELOG.md` 1.0.0 Fixed |
| 75+ | 发布口径，不是 27+26 | 仓库 `README.md`；Release v1.0.0「By the Numbers」 |
| 278 | τ-Voice 摘要中的题数。摘要没有给出域拆分。不要用 50+114+114 去凑 | `papers/2603.13686.md` |
| 约 25.5% / 约 25% / 40% | τ-Knowledge 摘要与 Sierra 公告，早于 v1.0.1 | `papers/2603.04370.md`；`blogs/sierra-bench-knowledge-voice.md` |
| 172 / 53 / 4 | v1.0.0「By the Numbers」：新源文件、新测试文件、新文档指南。不是 main 的当前文件总数 | Release v1.0.0 |
| 约 9 / 约 100 | `ACTION` 出现在 banking `reward_basis` 中的文档约数 | `docs/evaluation.md` |
| 9 个 wav | `data/voice/` 背景声与突发声 | `data_tree_summary.json` 的 `voice_assets` |
| pass^1 表 | 见 01。最大一格 gpt-5-5：37.37 → 46.39 | Release v1.0.1；`RELEASE_NOTES.md`（多 glm-5-think 与 qwen 两行） |
| $8.00 / $14.50 | task_074 的旧金标准与政策金额 | Release v1.0.1 |
| `--max-steps-seconds` | **以代码为准：默认 1200**。`docs/cli-reference.md` 与 `voice/README` 的表写 600。排行榜示例 JSON 里的 600 是示例值，不是 `cli.py` 的默认。不一致见下节 | `src/tau2/cli.py`（`default=DEFAULT_MAX_STEPS_SECONDS`）；`src/tau2/config.py`（`DEFAULT_MAX_STEPS_SECONDS = 1200`） |

## 文档和摘要互相不一致的地方

这里只记已经对过的冲突。说明书采用的读法写在「采用」一列。没有第三份材料时，不把其中一边改写成另一边。

| 冲突 | 一边 | 另一边 | 采用 |
|------|------|--------|------|
| `--max-steps-seconds` 默认值 | `docs/cli-reference.md` 与 `src/tau2/voice/README.md` 的表写 **600** | `config.py` 的 `DEFAULT_MAX_STEPS_SECONDS = 1200`；`cli.py` 的旗标 `default` 绑定这个常量 | **代码 1200**。600 留在本表，不写进运行命令 |
| airline / retail 当前题数 | 修订博客：原先 airline **50**、retail **114**。本地分析：当前 `tasks.json` 数组**未解析**，只有文件大小 | `DATA_SUMMARY.md` 与封存 `INDEX.md` 写「已核验」：airline `tasks.json_len` 50（train 30 / test 20 / base 50 / 语音 50），retail 114（train 74 / test 40 / base 114 / 语音 114） | **当前条数 UNKNOWN**。50 和 114 只作为博客里的原规模引用。摘要里的划分数字不升格 |
| Python | Getting Started 有「Python 3.12+」的说法 | `pyproject.toml` 的 `requires-python` 是 `>=3.12,<3.14`；`.python-version` 是 3.12 | **以 `pyproject.toml` 的上下界为准**。3.14 不在范围内 |
| 排行榜域名 | 当前 README 与提交文档写 `taubench.com` | `CHANGELOG.md` 文末和 0.2.0 一节写 `tau-bench.com` | PIN 把 `taubench.com` 当作当前主站。封存包没有跳转证明 |

`DATA_SUMMARY.md` 对 mock、telecom、`banking_knowledge` 的长度，本说明书仍然引用，并标明来源是这份摘要。banking 的 97 与 698 另外有 Release、CHANGELOG、公告和目录清单。airline、retail 没有同等的上游文档把「当前数组长度」写死，所以这两域单独降级。

## 博客和站点 URL

| URL | PIN 中的角色 |
|-----|----------------|
| <https://sierra.ai/blog/bench-advancing-agent-benchmarking-to-knowledge-and-voice> | Sierra 的 τ³ 公告 |
| <https://taubench.com/blog/tau3-task-fixes.html> | 题目修订 |
| <https://taubench.com/blog/tau-knowledge.html> | τ-knowledge |
| <https://taubench.com> | 当前 README、提交文档中的排行榜 |
| <https://tau-bench.com> | `CHANGELOG.md` 文末与 0.2.0 一节。PIN 把 taubench.com 当作当前主站，并注明旧文档有时写 tau-bench.com。封存包没有跳转证明 |
| <https://sierra.ai/blog/benchmarking-agents-in-collaborative-real-world-scenarios> | 仓库 README 徽章。PIN 未把它列为 τ³ 主公告 |

S3 桶名 `sierra-tau-bench-public`：`CHANGELOG.md` 1.0.1；`docs/interaction-metrics.md` 的 `aws s3 sync` 示例。

## 论文（只作引用，PDF 未封存）

| 文献 | 在本说明书里的位置 |
|------|---------------------|
| arXiv `2406.12045` τ-bench，Yao 等，2024 | 01 的对照节。BibTeX 在仓库 `README.md` |
| arXiv `2506.07982` τ²-bench，Barres 等，2025 | 01 的对照节。README 徽章指向这里 |
| arXiv `2603.13686` τ-Voice | τ³ 的语音组件。摘要在 `papers/2603.13686.md`。BibTeX 在 `README.md` |
| arXiv `2603.04370` τ-Knowledge | τ³ 的知识组件。摘要在 `papers/2603.04370.md`。BibTeX 在 `README.md` |
| arXiv `2512.07850` SABER，Cuadron 等 | 题目修订所依据的分析，不是 τ³ 本身。README 还有 OpenReview 链接 |
| arXiv `2503.04721` Full-Duplex-Bench | 只在 `docs/interaction-metrics.md` 里作不可直接比较的对照 |

PIN 规定：v0.2.0 及更早的 tag 只作对照。

## 模块与循环

| 事实 | 依据 |
|------|------|
| 目录树：agent、api_service、domains、environment、evaluator、gym、knowledge、metrics、orchestrator、registry、runner、user、voice | `AGENTS.md` |
| `HalfDuplexAgent.generate_next_message` / `FullDuplexAgent.get_next_chunk` | `AGENTS.md` |
| `Orchestrator` 与 `FullDuplexOrchestrator` | `AGENTS.md` |
| runner 三层与函数签名 | `docs/running_simulations.md` |
| `run_tasks` 默认 `ALL_WITH_NL_ASSERTIONS`，`run_simulation` 默认 `ALL` | 同上 |
| 奖励五项与乘积、airline 任务 1 | `docs/evaluation.md` |
| tick 0.2 秒、轮次阈值、`###STOP###` | `docs/cli-reference.md`，`docs/interaction-metrics.md`，`src/tau2/config.py` |
| `--max-steps-seconds` 代码默认 1200 | `src/tau2/cli.py` 第 305–309 行附近，`default=DEFAULT_MAX_STEPS_SECONDS` |
| 提前结束奖励为 0，除非 `AGENT_STOP` 或 `USER_STOP` | `src/tau2/evaluator/evaluator.py`；`evaluator/AGENTS.md` |
| registry 登记名，含 `telecom_full` / `telecom_small` | `src/tau2/registry.py` |
| provider choices 含 `openai_live`、`nova`、`qwen`、`livekit`，不含 `deepgram` | `src/tau2/cli.py` |
| 半双工默认 max_steps 200、max_errors 10、seed 300、并发 3、trial 1 | `config.py` 与 `docs/cli-reference.md` |
| `TerminationReason.TOO_MANY_ERRORS` | `CHANGELOG.md` 1.0.1 |
| 语音输出目录、`--audio-taps`、`--audio-debug`、`--verbose-logs` | `docs/getting-started.md`，`docs/cli-reference.md` |
| 交互指标窗口 2.0 秒 / 1.0 秒 | `docs/interaction-metrics.md` |

仍标 UNKNOWN 的：半双工第一句种子消息（`run()` 未封存）、`USER_STOP` 对应的话语、`no_knowledge` / `full_kb` / `golden_retrieval` 的提示词差异、pass^k 公式、telecom 排行榜默认划分。

## 依赖、钥匙、端点

| 事实 | 依据 |
|------|------|
| Python `>=3.12,<3.14`；`.python-version` 为 3.12 | `pyproject.toml`；`docs/getting-started.md` |
| extras：voice、knowledge、gym、dev、experiments、live、all | `pyproject.toml`；`AGENTS.md` |
| litellm `>=1.80.15,<1.82.7` | `pyproject.toml` |
| `.env.example` 的变量名 | 仓库 `.env.example` |
| `GOOGLE_API_KEY` 与 `GEMINI_API_KEY` 两套写法 | `src/tau2/voice/README.md`；`config.py` 注释。实际读取 UNKNOWN |
| 供应商默认模型与 WebSocket URL | `config.py` |
| 检索配置表、默认 `alltools`、嵌入模型名、sandbox-runtime `0.0.23` | `src/tau2/knowledge/README.md`；`AGENTS.md` |
| 「12 retrieval configurations」 | `CHANGELOG.md` 1.0.0；Release v1.0.0 |
| Redis `localhost:6379`，默认关闭 | `config.py`；`docs/getting-started.md` |
| `API_PORT = 8000`；查看器 `127.0.0.1:8004` | `config.py`；`docs/cli-reference.md` |

## 命令

06 的每条命令都能在下列文件之一里找到原文：`README.md`，`docs/getting-started.md`，`docs/cli-reference.md`，`docs/running_simulations.md`，`docs/leaderboard-submission.md`，`docs/voice-personas.md`，`docs/interaction-metrics.md`，`src/tau2/knowledge/README.md`，`src/tau2/voice/README.md`，`AGENTS.md`，`VERSIONING.md`，`CHANGELOG.md`，`RELEASE_NOTES.md`，Release `v1.0.0`，Release `v1.0.1`。

`--fresh-tasks` 不在封存的 `docs/cli-reference.md` 表格里，在 `CHANGELOG.md` 与 Release v1.0.1。

## 封存包里没有、因此不能当证据的东西

- 论文 PDF 全文。封存的是 arXiv 摘要。
- `orchestrator` 的 `run()` 实现。封存的是该目录的 README。
- `data/tau2/domains/*/tasks.json` 的正文。airline、retail 的当前数组长度因此标 UNKNOWN。mock、telecom、banking 若引用长度，来源是 `DATA_SUMMARY.md` 或目录清单，不是题目正文。
- telecom 排行榜默认划分。索引明确要求不要发明。
- `src/tau2/gym/README.md`：完整封存包没有这份文件。`domains/README.md` 和 `voice/audio_native/README.md` 在完整封存包里。
- 排行榜当前名次。01 的分数表只是 v1.0.1 对已有 `banking_knowledge` 轨迹的重打分，不是完整榜单。
- pass^k 的数学公式。τ 摘要没有写出。本地分析的「k 次全部成功」是转述，不是公式。

## 本说明书的文件

| 路径 | 角色 |
|------|------|
| `README.md` | 仓库说明、阅读顺序、版本钉、上游链接 |
| `docs/tau3-bench-explainer/00-readme.md` | 三名陷阱与一页纸 |
| `docs/tau3-bench-explainer/01-what-is-tau3.md` | 测什么、域、文字/语音、v1.0.1、τ/τ² 对照 |
| `docs/tau3-bench-explainer/02-architecture.md` | 模块拓扑 |
| `docs/tau3-bench-explainer/03-agent-loop.md` | 半双工与全双工 |
| `docs/tau3-bench-explainer/04-dependencies.md` | 依赖、钥匙、模型、网络 |
| `docs/tau3-bench-explainer/05-unnamed-concepts.md` | 横切概念 |
| `docs/tau3-bench-explainer/06-runbook.md` | 官方命令 |
| `docs/tau3-bench-explainer/99-evidence.md` | 本页 |
