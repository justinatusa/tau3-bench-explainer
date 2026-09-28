# τ³-bench 在测什么

## 评测对象

被测的是客服 agent：它读一段域政策，看见一组工具，和用户对话，在需要时调用工具改数据库或查记录。用户不是真人，是 user simulator，由另一个 LLM 按题目里的人设说明来说话。语音模式下，这个用户的话还会被合成成音频。

τ³-bench 同时测三类能力，对应 v1.0.0 的三块发布说明：

1. **按政策办业务**（airline、retail、telecom，以及 mock）。政策写在域目录里，工具由该域的 toolkit 提供。
2. **先检索、再办事**（`banking_knowledge`）。相关规定散落在大量文档里，agent 不能假设政策已经整段放在提示词里。具体能不能看到全文，取决于 `--retrieval-config`。三种不给检索工具的配置（`no_knowledge`、`full_kb`、`golden_retrieval`）在提示词里究竟放了什么，封存的 `knowledge/README.md` 只写了「Tools: None」，没有写提示词差异。这里标 UNKNOWN。
3. **用语音把同一类业务做完**（`--audio-native`）。v1.0.0 Release 写：airline、retail、telecom、mock 可以做全双工语音。数据清单里 `banking_knowledge` 也有 `tasks_voice.json`。排行榜提交文档的语音示例只跑了 retail、airline、telecom。语音榜是否收录 banking：UNKNOWN。

它不测：模型训练，以及排行榜名次本身（名次在 [taubench.com](https://taubench.com)，本仓库不抄榜）。τ / τ² 摘要里的百分比只出现在文末对照节。

## 五个域

域由 `registry.py` 注册。每个域在 `src/tau2/domains/<name>/` 里实现环境，数据在 `data/tau2/domains/<name>/`。

| 域 | 封存清单里看得到的文件 | 题数 |
|----|------------------------|------|
| `airline` | `policy.md`、`db.json`、`tasks.json`、`tasks_voice.json`、`split_tasks.json`、`audio_difficulty.json` | **当前** `tasks.json` 条数 UNKNOWN。修订博客把原先规模写成 50。v1.0.0 点名修订 27 道，27 不是题库大小 |
| `retail` | 同上，另有子目录 `task_issues/` | **当前** `tasks.json` 条数 UNKNOWN。修订博客把原先规模写成 114。点名修订 26 道。`task_issues/` 里的文件是什么：UNKNOWN |
| `telecom` | `db.toml`、`user_db.toml`、`main_policy.md`、`main_policy_solo.md`、`tech_support_manual.md`、`tech_support_workflow.md`、`tech_support_workflow_solo.md`、`tasks.json`、`tasks_full.json`、`tasks_small.json`、`tasks_voice.json`、`split_tasks.json`、`audio_difficulty.json`，以及 `workflows/` | 数据摘要：`tasks.json` **2285**，语音配置 2285，并写 `tasks_full.json` 与 `tasks.json` 字节数相同。数据库是 TOML。默认划分不是 2285，见下文 |
| `banking_knowledge` | `db.json`、`tasks.json`、`tasks_voice.json`，子目录 `documents/`、`prompts/`、`tasks/` | **97** 道题，文档 **698**。数据摘要里语音配置也是 97 |
| `mock` | `db.json`、`user_db.json`、`policy.md`、`policy_solo.md`、`tasks.json`、`tasks_voice.json`、`split_tasks.json` | 数据摘要：`tasks.json` **10**，语音配置 **9**。域 README 称它为开发用的轻量域 |

域 README 给每个域的一句职责：`airline` 是航班预订、取消和客服；`retail` 是订单、退货和商品咨询；`telecom` 是账户和故障处理；`banking_knowledge` 是可配置检索的银行客服；`mock` 是开发用的轻量域。

**10**：是数据摘要里 mock 的 `tasks.json` 条数。不是语音配置条数。同一份摘要把语音 `configs` 长度记成 **9**。9 不是「少了一道文字题」的解释，摘要只记录两个长度不相等。mock 的划分长度不在摘要里，不补。

**50**：是题目修订博客对**原先** airline 域的说法，原文是 original airline (50 tasks)。是博客语境里的原规模。不是本说明书解析 pin 上 `tasks.json` 之后得到的当前条数，也不是「只留下修订过的 27 道」。当前数组长度、当前 `split.base`、当前语音配置条数、train/test 各有多少：封存包没有这份 JSON 正文，本地分析未解析数组，一律 UNKNOWN。数据摘要文件里写过 50 以及 train 30 / test 20 / base 50，那是摘要自己的断言；本说明书不把它升级成已核对的当前长度。冲突记在 [99-evidence.md](99-evidence.md)。

**114**：在 retail 上，是同一篇博客对**原先** retail 域的说法，原文是 retail (114 tasks)。不是当前 `tasks.json` 长度，也不是修订题数 26。当前数组长度和划分长度 UNKNOWN，理由与 airline 相同。数据摘要里写过 114 以及 train 74 / test 40，同样不升格。

**114**：在 telecom 上是另一回事。数据摘要把它写成 `split.base` 的长度。不是 `tasks.json` 的 2285，也不是博客里原先 retail 的那 114。两个 114 只是数字相同。

**2285**：是数据摘要里 telecom 的 `tasks.json` 长度，也是该摘要里的 `split.full` 和语音配置长度。摘要还写 `tasks_full.json` 与 `tasks.json` 字节数相同。不是 CLI 默认会跑的题数。`--task-split-name` 的代码默认是字符串 `base`。摘要里 telecom 的 `base` 是 114，`small` 是 **20**。摘要里的 train **74**、test **40** 与 retail 数据摘要里的划分数字相同；摘要没有说这是同一批题，而 retail 的当前划分长度本身是 UNKNOWN。

**排行榜上的 telecom 用哪一份：UNKNOWN。** 封存索引写明不要猜。提交文档同时说两件可能冲突的事：使用默认的 `base` 划分，以及跑完该域全部题目、不要用 `--num-tasks` 截断。对 telecom，数据摘要里这两句分别对应 114 和 2285。对 airline 和 retail，仓库 README 的意图是：评测时用 `base`，以便和原先完整题集一致。当前文件里 `base` 是否仍等于 `tasks.json` 全长：UNKNOWN，不能用博客里的 50 和 114 填上。`registry.py` 另外登记了任务集名字 `telecom_full` 和 `telecom_small`。榜单列的究竟是哪一个，提交文档没有写死。

**97**：是 `banking_knowledge` 的任务数。v1.0.0 Release、`CHANGELOG.md` 和 Sierra 公告都写 97 tasks；目录清单里 `tasks/task_*.json` 也是 97 个文件。数据摘要把 `tasks.json` 长度和语音配置也记成 97。97 个分文件与聚合 `tasks.json` 是否逐字节相同：没有做过 JSON diff，UNKNOWN。不是五个域的题数之和。覆盖范围，Release 写的是账户、信用卡、争议、转账。

**不是编号 1 到 97 连续。** 文件名从 `task_001.json` 到 `task_102.json`，清单里缺 `task_009`、`task_011`、`task_013`、`task_030`、`task_042`。这是文件名缺口，不是另一套题数。

不要把五个域的 `tasks.json` 长度加成排行榜题量。airline 和 retail 的当前长度是 UNKNOWN。telecom 的默认划分名 `base` 在数据摘要里不是 2285。

**698**：是 `banking_knowledge` 的 `documents/` 文件数。v1.0.0 称它们为 policy and procedure documents，并写只有少数几篇和某一道题有关。Sierra 公告（2026-03-18）写的是同一数字：698 documents，跨 21 个产品类，约 19.5 万 token。不是五个域的政策文件总数。airline、retail、telecom、mock 在数据清单里 `documents_count` 为 0。域 README 写的是 “700+ documents”，论文摘要写 “roughly 700”。698 是钉上的文件数；700 和 700+ 是约数。

**21** 和 **约 19.5 万 token**：是公告对这 698 篇文档的描述。不是任务数，也不是另一种文档计数。

**815**：是同一份清单里 `banking_knowledge` 域目录的 `file_count`。不是任务数，也不是文档数。97 个任务文件、`tasks.json`、`tasks_voice.json`、`db.json`、`prompts/` 等都算在这 815 里。

每个域还规定：一份 agent 必须遵守的 policy、一组 agent 工具、一组题目。用户侧工具是可选的。CLI 文档点名 telecom 有 user tools，所以 `tau2 play` 可以「扮演用户」。

`telecom-workflow` 不是第六个主域。它是 CLI 消融示例里的任务集名字，用来换一套 workflow 体裁的政策。见 [05-unnamed-concepts.md](05-unnamed-concepts.md)。

## 文字和语音

| | 文字（half-duplex） | 语音（full-duplex，audio native） |
|--|---------------------|-------------------------------------|
| 通道 | 轮流的文本，加工具调用 | 双方可以同时出声的音频 |
| 被测 agent 的默认实现 | `llm_agent`（`LLMAgent`） | `discrete_time_audio_native_agent` |
| 用户的默认实现 | `user_simulator` | `voice_streaming_user_simulator` |
| 编排器 | `Orchestrator` | `FullDuplexOrchestrator` |
| 时间单位 | 步。默认最多 200 步 | tick。文档中的 CLI 默认时长 0.2 秒；对话时长上限见 03 |
| 结果文件 | 单个 `results.json` | 目录：`results.json` + `simulations/sim_*.json`，详细日志另在 `artifacts/` |

语音不是「把文字对话用 TTS 读出来再打分」这一种东西。v1.0.0 的语音路径是：用户模拟器把要说的话合成音频，agent 通过实时语音 API 回音频，两边在同一条时间线上重叠。评测用的转写是另一条管线（Deepgram 或 OpenAI）。细节在 03、04。

`config.py` 里还有一节标题为 legacy half-duplex voice 的默认值（`DEFAULT_VOICE_ENABLED = False`，合成用 ElevenLabs，转写模型默认 `nova-3`）。怎样用 CLI 打开这条旧的半双工语音路径：封存摘录没有对应旗标说明。标 UNKNOWN。τ³ 文档里让人跑的语音是 `--audio-native`。

## 题目修订（Task Quality）

**75+**：是 v1.0.0 发布说明和 README 的口径，表示 airline、retail、banking 上修过的题很多。不是「每个域 75 题」，也不是「27+26=75」。

点名计数在发布说明里是：

- **27**：airline 上被点名修订的任务数。changelog 分类包括：删掉错误的期望动作、写清含糊的用户说明、改掉做不到的约束、堵政策空子、补上缺失的退路、改数据。
- **26**：retail 上被点名修订的任务数。例子包括删掉政策不支持的 PayPal 退款期望、改掉系统不支持的同 SKU 换货。
- **20+**：`CHANGELOG.md` 在 1.0.0 的 Fixed 里给 banking 修订写的约数（「20+ tasks」）。不是和 27、26 相加后等于 75 的第三项。75+ 与 27+26+20+ 的算术关系，发布说明没有给出。

这些修订的依据，README 写的是 SABER 的分析（Cuadron 等，arXiv `2512.07850`）。SABER 是修订所引用的论文，不是 τ³ 这套评测本身。摘要里的增益数字放在文末对照节，不放进 τ³ 的题数。

v1.0.1 又改了 `banking_knowledge` 的题和计分，和上面这批「75+」不是同一次发布。见下一节。

## v1.0.1：只有 banking_knowledge 的分数不能跨版本比

**是什么**：tag `v1.0.1` 修的是 `banking_knowledge` 的计分和若干题面。README 写：tau2-bench `< 1.0.1` 的结果与 `>= 1.0.1` 不可比；受影响的排行榜提交被重新计分。其他域不受影响。

**不是什么**：不是五个域全部换了评分标准，也不是「所有模型分数都涨了约 9 分」。约 9 分是 pass^1 涨幅的上沿，随模型变化。最大的一格是 gpt-5-5：37.37 → 46.39（+9.02）。

计分方案本身的修复（Release 列出的第 1–5 项）在重放排行榜轨迹时只会使分数上升：没有原先及格、重打分后不及格的仿真。第 6 项是题面金标准变了，方向可以不同，见 05 的 task_074。

下表是 Release / `RELEASE_NOTES.md` 里的重打分结果。列的是 pass^1 和 pass^4，单位是排行榜上的成功率数字（提交文档说 `pass_1`…`pass_4` 为 0–100 的成功率或 `null`）。`CHANGELOG.md` 把前两行称作 “GPT-5.5 xhigh”“GPT-5.4 xhigh”，表内名字仍是 `gpt-5-5`、`gpt-5-4`。下表保留表内名字。

| 模型 | pass^1 旧 | pass^1 新 | Δ | pass^4 旧 | pass^4 新 | 翻转 |
|------|-----------|-----------|---|-----------|-----------|------|
| gpt-5-5 | 37.37 | 46.39 | +9.02 | 20.62 | 27.84 | +35/−0 |
| gpt-5-4 | 30.67 | 39.43 | +8.76 | 16.49 | 21.65 | +34/−0 |
| gpt-5-2 | 24.74 | 32.22 | +7.48 | 11.34 | 18.56 | +29/−0 |
| claude-opus-4-7 | 25.26 | 30.15 | +4.89 | 12.37 | 15.46 | +19/−0 |
| claude-opus-4-6 | 24.48 | 27.32 | +2.84 | 10.31 | 11.34 | +11/−0 |
| gemini-3-flash | 20.62 | 27.32 | +6.7 | 4.12 | 7.22 | +26/−0 |
| gemini-3-1-pro-preview | 22.54 | 26.03 | +3.49 | 8.42 | 9.28 | +14/−0 |
| claude-sonnet-4-5 | 22.42 | 25.26 | +2.84 | 10.31 | 10.31 | +11/−0 |
| claude-opus-4-5 | 21.39 | 24.74 | +3.35 | 8.25 | 11.34 | +13/−0 |
| gemini-3-pro | 15.72 | 18.04 | +2.32 | 4.12 | 4.12 | +9/−0 |
| grok-4-2 | 17.57 | 18.04 | +0.47 | 7.29 | 8.25 | +2/−0 |
| grok-4-fast | 14.18 | 15.72 | +1.54 | 4.12 | 4.12 | +6/−0 |
| gemini-2-5-pro | 12.76 | 13.66 | +0.9 | 1.08 | 1.03 | +4/−0 |
| grok-4-1-fast | 12.4 | 13.14 | +0.74 | 5.21 | 5.15 | +3/−0 |
| gpt-5-2-none | 11.08 | 12.63 | +1.55 | 4.12 | 4.12 | +1/−0 |
| glm-5-think | 9.79 | 9.79 | 0 | 3.09 | — | +0/−0 |
| qwen3.5-397b-a17b-think | 9.79 | 9.79 | 0 | 5.15 | 5.15 | +0/−0 |

最后两行出现在仓库 `RELEASE_NOTES.md`。GitHub Release 正文的表停在 `gpt-5-2-none`，脚注仍讨论 glm-5-think。

脚注里的读法：

- 没有仿真从及格变成不及格。奖励变化都是上升。
- pass^k 按排行榜惯例重算时，把 infrastructure-error 的仿真算作失败的 trial。gemini-2-5-pro 和 grok-4-1-fast 的 pass^4 有不到 0.1 的下降，来自这条约定，不是某次仿真从过变挂。
- glm-5-think 的轨迹文件里，有的任务只有 3 次 trial，pass^4 无法重算，保留旧值。
- NL assertion 的判断沿用原打分，因为 v1.0.1 的修复都在环境一侧，重打分是确定的。
- 几乎所有翻转来自「多余的读工具调用把奖励打成 0」这一项（#329）。

pass^k 的数学公式见 05。封存源没有写出公式。

重打分命令（Release 原文）：

```bash
tau2 evaluate-trajs --fresh-tasks path/to/results.json -o regraded/
```

要复现修复前的行为，安装 `pre-v1.0.1`，不要用 v1.0.0 tag 来「代表修复前的全部行为」。`RELEASE_NOTES.md` 写：`v1.0.0` tag 比 `pre-v1.0.1` 早几个月；`pre-v1.0.1` 包含这之间除计分改动以外的内容。从提交 `2be6916`（2026-04）到 `pre-v1.0.1`，评测器、环境、orchestrator、banking 工具和数据库逐字节相同；这窗口里的变化只在仿真侧（task_053 的 `user_tools`、默认检索从 `bm25` 改为 `alltools`、检索提示词）。用这窗口里的提交重放已录好的轨迹，计分相同。

## 任务划分

`cli.py` 里 `--task-split-name` 的默认值是字符串 `base`。域 README 写：划分文件至少要实现名为 `base` 的划分，它是默认划分。README 又写：评测 agent（而不是训练）时用 `base`，以便和原先 τ-bench 的完整题集一致。

这句话是 README 对 `base` 的意图：评测时用它，以便和原先 τ-bench 的完整题集一致。它不是「五个域上 `base` 的长度都等于当前 `tasks.json`」的测量结果。airline 和 retail 的当前全长 UNKNOWN，不能用博客里的 50 和 114 代替。telecom 上，数据摘要写 `base` 是 114、`tasks.json` 是 2285，所以「`base` = 该域全部题」至少在 telecom 上不成立。telecom 榜单用哪一份仍是 UNKNOWN，见上一节。

`train` / `test` 给 RL 实验用（0.2.1 加入 Gymnasium）。mock 的划分长度不在数据摘要里，不补。airline、retail 的 train/test 当前长度也不补。

`--num-tasks 5` 是文档里的冒烟规模。排行榜提交要求不要用 `--num-tasks` 或 `--task-ids` 截断。这是提交约束。代码默认 `--num-trials` 是 1，`--num-tasks` 默认是 `None`（不截断）。截断与否，仍然先落到当前的 split 上。

## τ-Knowledge 和 τ-Voice（τ³ 的两块，不是旧代）

这两篇是 v1.0.0 的组成部分。摘要已封存。下面的百分比是论文或 2026-03-18 公告里的实验结果，不是 [taubench.com](https://taubench.com) 的当前名次，也不是 v1.0.1 对 `banking_knowledge` 的重打分。论文日期是 2026-03-04（知识）和 2026-03-14（语音），都早于 2026-07 的计分修复。

**τ-Knowledge**，arXiv `2603.04370`。摘要说：新域 τ-Banking 要在大约 700 篇互相引用的文档上做检索，再改账户状态。嵌入检索和终端搜索下，前沿模型即使用高推理预算，pass^1 也只有约 **25.5%**。公告把最好的一档写成 GPT-5.2（high reasoning）大约 **25%**，并且写：就算把完成任务所需的文档直接给模型，也只升到 **40%**。

**25.5%、25%、40%**：是这篇论文摘要和 Sierra 公告里的 pass 口径。不是 v1.0.1 重打分表里的数字。那张表里的 `gpt-5-2` 从 24.74 到 32.22，模型写法不同，计分规则也已改变，不要并成同一个数。

**τ-Voice**，arXiv `2603.13686`。摘要说框架把 τ²-bench 扩成可与文字直接对比的全双工语音评测；用户模拟器与墙钟解耦，所以可以用强 LLM 而不受实时限制。评测跨 **278** 道题。GPT-5（reasoning）文字约 **85%**；语音在干净条件下 **31–51%**，在噪声和多种口音下 **26–38%**，只保留文字能力的 **30–45%**；**79–90%** 的失败来自 agent 行为。公告把无推理文字模型写成约 **54%**，用来和干净语音的 31–51%、真实语音的 26–38% 对比。

**278**：是语音论文摘要里的题数。不是 telecom 数据摘要里的 2285，也不是五个域当前 `tasks.json` 的长度和。摘要没有写出这 278 道题由哪些域组成。不要用博客里的原 airline 50、原 retail 114，加上 telecom 的 `split.base`，去凑这个数。

**85%、31–51%、26–38%、30–45%、79–90%、约 54%**：是摘要和公告里的实验结果。不是本仓库代码算出的常量。

## 对照：τ 和 τ²

这一节只说明名字从哪来。下面引用的百分比来自封存的 arXiv 摘要，用来说明前代论文自己声称测过什么。它们不是 τ³ 的榜，也不是 v1.0.1 的重打分。全文 PDF 不在封存包里，摘要没写的表不补。

**τ-bench（2024）** 是论文，不是当前产品名。摘要已封存：Shunyu Yao、Noah Shinn、Pedram Razavi、Karthik Narasimhan，arXiv `2406.12045`，发布于 2024-06-17。摘要说：用户由语言模型扮演，agent 拿到域 API 和政策；评分比较对话结束时的数据库和标注的目标状态；并提出 pass^k，用来看多次试验是否稳。摘要里的实验句是：即便 gpt-4o 这类函数调用 agent，任务成功率也低于 50%，retail 上 pass^8 低于 25%。这些是 2024 年论文摘要里的数字，不是 τ³ 的榜。仓库 `docs/evaluation.md` 写：默认的 `DB + COMMUNICATE` 是为了和这篇论文的结果口径一致。0.1.0 的 Release Notes 把当时的框架描述成对话 agent 评测，域是 mock、airline、retail、telecom，安装方式是 `pip install -e .`，Python 是 3.10+。那是 τ³ 之前的发布说明。pass^k 的数学公式摘要没有给出，仍是 UNKNOWN。

**τ²-bench（2025）** 是论文，也不是一个仍在单独发布的产品线。摘要：Victor Barres、Honghua Dong、Soham Ray、Xujie Si、Karthik Narasimhan，arXiv `2506.07982`，发布于 2025-06-09。标题是 *Evaluating Conversational Agents in a Dual-Control Environment*。摘要说此前的基准是单方控制（只有 agent 能用工具），τ² 加入 telecom 双控域：agent 和用户都用工具改同一环境；任务由原子组件程序化生成；并比较 no-user 和 dual-control。仓库 README 的论文徽章指向这篇。从 τ² 升级到当前树的安装说明：从 `pip install -e .` 改为 `uv`；Python 从 `>=3.10` 改为 `>=3.12, <3.14`。摘要没有给出可引用的总分表，这里不补分数。

SABER（arXiv `2512.07850`，2025-11-26）是题目修订所引用的分析，不是 τ³ 本身。摘要说突变步骤上的偏差比只读步骤更能把成功打成失败，并发布了 τ-Bench Verified。τ³ 的 Task Quality 写明依据这篇分析。摘要里的 +28% 等增益是 SABER 自己的方法结果，不是 τ³ 榜上的分。

v0.2.0 及更早的 tag，PIN 规定只作对照，不当作 τ³。0.2.0（2025-10-06）是网页排行榜；0.2.1（2025-11-07）是 Gymnasium、`tau2 play`、train/test 划分。

## 本页依据

`DATA_SUMMARY.md`，`data_tree_summary.json`，`src/tau2/cli.py`，`src/tau2/registry.py`，`src/tau2/domains/README.md`，论文摘要 `papers/2406.12045.md`、`2506.07982.md`、`2512.07850.md`、`2603.04370.md`、`2603.13686.md`，Sierra 公告 `blogs/sierra-bench-knowledge-voice.md`，题目修订博客 `blogs/tau3-task-fixes.md`。airline、retail 的当前条数以 99 的冲突表为准，不以 `DATA_SUMMARY.md` 的 `tasks.json_len` 为准。见 [99-evidence.md](99-evidence.md)。
