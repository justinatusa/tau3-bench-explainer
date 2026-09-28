# 文档里没有单独起名、但会绊倒人的概念

下面每一节都是读代码和发布说明时会撞上、但没有单独教程页的说法。证据不够的句子标 UNKNOWN。

## dual-control

**是什么**：0.1.0 Release Notes 用它描述这套环境——agent 和 user 都可以跟系统交互。τ² 论文标题就是 *Evaluating Conversational Agents in a Dual-Control Environment*（arXiv `2506.07982`）。在代码文档里，对应的事实是：域可以提供 user tools；telecom 被 CLI 点名为有用户工具；`tau2 play` 可以让人扮演用户，由 LLM agent 当客服。

**不是什么**：不是 τ³ 新造的开关，也不是 `--audio-native`。全双工解决的是「两边同时出声」。dual-control 说的是「两边都能操作业务系统」。一个文字版 telecom 任务可以是 dual-control，同时仍然是半双工。

用户工具在一次仿真里由谁调用、调用结果写进哪一边的数据库：封存文档只写到 `UserSimulator` 接收 `env.get_user_tools()`。更细的控制权划分 UNKNOWN。τ² 摘要说 telecom 里 agent 和用户都用工具改同一环境，并把它写成 Dec-POMDP。摘要没有给出状态空间。不把 Dec-POMDP 展开成公式。

和它相邻、但不要混成一个概念的消融（CLI 参考，仅 telecom 示例）：

| 名字 | 命令里的标志 | 文档中的含义 |
|------|----------------|--------------|
| no-user | `--agent llm_agent_solo --user dummy_user` | LLM 事先拿到全部工具和信息，没有用户来回 |
| oracle-plan | `--agent llm_agent_gt` | LLM 拿到一份 oracle plan，不再自己做行动规划 |
| workflow policy | `--domain telecom-workflow` | 换一套 workflow 体裁的政策，用来看政策写法的影响 |

`dummy_user` 是否完全不说话：文档只给了命令，没有定义类的行为。标 UNKNOWN。

## DB 哈希奖励

**是什么**：`RewardType.DB`。仿真结束后，把预测环境的数据库做成哈希，和目标哈希比较。目标环境是一份干净库，重放 `evaluation_criteria.actions`。哈希相同则这项奖励为 1，否则为 0。agent 不必走出和 `actions` 相同的工具顺序。

**不是什么**：不是「把 agent 的工具调用列表和标准答案做字符串匹配」。那是 `RewardType.ACTION`，默认不进 airline、retail、telecom 的 `reward_basis`。

最终奖励是 `reward_basis` 各项的乘积。默认 `["DB", "COMMUNICATE"]` 时，奖励 = `db_reward * communicate_reward`。其中一项为 0，总分就是 0。

`docs/evaluation.md` 用 airline 任务 `1` 做了例子：参考动作是两次只读查询；重放后数据库不变，所以目标哈希等于初始哈希。正确行为是拒绝取消。agent 只要不写库，DB 这项就是 1。`communicate_info` 为空，沟通这项自动为 1。题目里虽然写了 `nl_assertions`（「Agent should not approve the cancellation.」），但 `NL_ASSERTION` 不在 `reward_basis` 里，这条断言只是诊断，不进分数。因此一个礼貌拒绝、一次工具都不调的 agent，奖励可以是 1.0。

**约 9 / 约 100**：`evaluation.md` 写，`ACTION` 只出现在 `banking_knowledge` 的一小部分题的 `reward_basis` 里，「about 9 out of ~100」。airline、retail、telecom 完全不用 `ACTION`。这是文档里的约数。任务 JSON 未封存，无法复核成精确分数。不是「97 题里恰好 9 题」，97 是任务文件数，见 01。

### 参考轨迹和金标准

`actions` 是一条能解出这道题的工具序列，通常是出题的人走过的那条。它始终会被重放，用来生成目标数据库。只有当 `ACTION` 在 `reward_basis` 里时，它才变成「唯一可接受的轨迹」。

`partial_action_reward` 是诊断：参考动作匹配了多少。agent 可以是 `0/n` 同时奖励为 1，只要它用另一条路径到达了同一个数据库终态。排行榜不看这个诊断分。

`EvaluationType.ALL_IGNORE_BASIS` 会把环境、动作、沟通（以及可选的 NL）乘在一起，不管 `reward_basis`。这仍然是「像不像这一条参考路径」，不是「行为对不对」。

### v1.0.1 的读日志白名单

**是什么**：`call_discoverable_agent_tool` 曾经把每一次调用都写进表 `agent_discoverable_tools`，而这张表参加 DB 哈希。agent 多做一次金标准里没有的读——例如开户之后再列出账户——哈希就对不上，奖励变成 0。知识库本身鼓励这种核对。修复后，写调用仍始终记录；读调用只在该题金标准要求时记录（per-task `read_log_allowlist`）。

**不是什么**：不是「读工具不再影响分数」。金标准要求的读仍然要出现，否则该项断言仍能把题判负。多余的核对读不再污染哈希。

Release 写：这项（#329）构成了几乎全部的分数移动。

重打分（`tau2 evaluate-trajs`）现在使用同一份白名单。1.0.1 之前录下的轨迹在重放时用 `Environment.set_state(strict=False)`，这样 `25` 和 `25.0` 这种表面上的工具输出差异不会让重放中断。现场评测仍然严格。

### 整数和浮点

**是什么**：#397。`25` 和 `25.0` 在 JSON 里是同一个数，以前会生成不同的、可复现的记录 ID 和数据库哈希。现在数值参数会规范化。

**不是什么**：不是排行榜上单独涨分的主因。Release 写这项自己没有改变已有榜单的最终分数，只是去掉了一种潜伏的误伤。

### task_074 的金标准金额

**8.00 和 14.50**：是美元金额，只属于 `banking_knowledge` 的 task_074。Light Blue 账户文档允许每月两次免费的网外 ATM 和两次免费的境外 ATM。旧金标准只退了各一次，金额是 $8.00。1.0.1 把金标准改成按政策的 $14.50。

**不是什么**：不是全库的费率，也不是 pass^1 的单位。走旧金额 $8.00 的轨迹在 1.0.1 上这道题不及格；按政策退 $14.50 的及格。Release 写：在检查过的轨迹集里，没有 trial 正好命中旧的 $8.00，所以没有因此从过变挂；有一次 gemini-3-1-pro-preview 的 trial 因为退了正确的 $14.50 而变成及格。

题面里原先把答案写在流水描述上，例如 `(SHOULD BE FREE - 1ST OF 2)`。十二处被改成普通文本 `NON-RHO ATM FEE` / `FOREIGN ATM FEE`。task_072 和 task_073 上四条同类注释也删了，但那只改描述，不改分数。

### 其他 banking 修复（不改「跨版本不能比」这个结论的范围）

- #402：任务 077–086（卡丢失或被盗）的金标准轨迹补上了真实 agent 必须做的读。
- #403：`get_bank_account_transactions_9173` 文档写最近的在前，工具以前按插入顺序、最旧的在前。现在按日期降序、稳定排序。task_085 的夹具行改成和 083/084 同一套「争议最早那笔重复扣款」的打破平局办法。金标准动作没有改。重打分确认这项不改变已有榜单分数。
- #388：Platinum Rewards 文档里同一张卡写了两个返现比例，现已改成一致。具体两个比例的数字：UNKNOWN（Release 没写数值）。

## pass^k

**是什么**：排行榜和 `AgentMetrics.pass_hat_ks` 使用的成功率家族。提交 JSON 的字段是 `pass_1`…`pass_4`，文档定义为成功率百分比，范围 0–100，或 `null`。CLI `tau2 leaderboard --metric` 的可选项是 `pass_1`、`pass_2`、`pass_3`、`pass_4`、`cost`，默认 `pass_1`。0.1.0 Release Notes 写成 “Pass@k”（at 符号）。后来的 Release 写成 `pass^k`。封存文档没有解释两个记号的差别。

**不是什么**：封存的 τ 论文摘要只说 pass^k 用来看多次试验是否稳定，并给出 retail 上 pass^8 低于 25% 这一句结果。摘要没有写出公式。`metrics/agent_metrics.py` 不在封存包里。公式 UNKNOWN。本地分析把 pass^k 读成「k 次全部成功才算过，不是 k 次里任意一次成功」。那是分析对 τ 摘要的转述，不是摘要里的公式，本说明书不把它写成已核对的定义。

文档能够确定的用法：

- 同一道题要有至少 4 次 trial，pass^4 才算得出来。glm-5-think 有的任务只有 3 次 trial，所以 pass^4 不能重算。
- 语音提交通常只报 pass^1，更高的 pass^k 可以是 `null`，因为音频多次 trial 很贵。
- 排行榜重算 pass^k 时，把 infrastructure-error 仿真算作失败 trial。
- `CHANGELOG.md` 另有一句：`INFRASTRUCTURE_ERROR` 被排除在 `pass^k` 和 `avg_reward` 之外。这句话描述的是修复「幻觉工具调用导致整题被标成基础设施错误」之前的指标行为。两句都留着：指标代码曾把这类结果排除出平均；v1.0.1 重算榜单时的惯例是把它们算作失败 trial。不要合成一句「官方只有一种处理」。
- 榜上的 Overall 怎样由各域平均：交互指标文档说 pass^1 的 Overall 是各域平均（缺的域跳过）。这句是在类比交互指标，不是 pass^k 公式。

`avg_reward` 是 `compute_metrics` 返回的平均奖励，和 pass^k 一起出现在 `AgentMetrics` 里。它是不是「奖励的算术平均、且排除基础设施错误」：changelog 只说基础设施错误曾被排除出 `avg_reward`。精确聚合 UNKNOWN。

## NL assertions

**是什么**：`evaluation_criteria.nl_assertions`，一组自然语言句子，由 `NLAssertionsEvaluator` 交给 LLM 判断真假。默认评判模型 `gpt-4.1-2025-04-14`，温度 0.0。只有 `reward_basis` 含 `NL_ASSERTION` 时才乘进奖励。文档把这项标成 Experimental / WIP。

**不是什么**：不是默认分数的一部分。airline 任务 `1` 有一条 NL 断言，但不进奖励。

CLI 批处理默认 `EvaluationType.ALL_WITH_NL_ASSERTIONS`，所以诊断里会跑这些断言。`run_simulation` 默认 `EvaluationType.ALL`，默认不跑。0.1.3（2025-08-26）曾去掉「默认的自然语言断言检查」，因为它们造成问题。当前默认是否重新打开：以 `evaluation.md` 为准——进不进奖励看 `reward_basis`，诊断路径看 `evaluation_type`。

v1.0.1 重打分保留了原来的 NL 判断，没有用新模型重判。

零售题 33、34 的 changelog：错误的 `nl_assertions` 和 `communicate_info` 被删掉，`reward_basis` 从 `["DB", "NL_ASSERTION"]` 改成 `["DB"]`。这说明个别题曾经用 NL 断言做门槛，但不是域的默认。

## retrieval-config

**是什么**：只对 `banking_knowledge` 有意义的旗标，决定 agent 怎样接触那 698 篇文档。省略时默认 `alltools`。排行榜要求在该域结果里填写 `retrieval_config`，榜上会显示成徽章。

**不是什么**：不是模型名字，也不是任务划分。同一个 agent 模型配 `bm25` 和配 `alltools`，是两次不同的评测。提交文档说各域的 agent 模型和用户模型必须一致；检索配置是 banking 结果上的附加字段。

`no_knowledge`、`full_kb`、`golden_retrieval` 三行的工具都是 None。它们之间的差别在封存 README 里没有写。UNKNOWN。不要凭名字把 `full_kb` 解释成「全文塞进上下文」除非以后读到实现。

`terminal_use` 在提交文档里被描述为：agent 用 shell（grep、cat、find 等）在知识库里找。`alltools` 的 shell 是同一只读沙箱。

嵌入缓存、沙箱版本 0.0.23、后缀 `_reranker` / `_grep`，见 04。

## AudioTap、verbose-logs、audio-debug

三件都是调试输出，都不是分数。

| 旗标 | 文档中的作用 |
|------|----------------|
| `--verbose-logs` | 保存详细日志：LLM 调用、音频、tick。语音目录下出现 `artifacts/.../audio/both.wav`、Audacity 标签、`llm_debug/*.json`、`task.log`、`sim_status.json` |
| `--audio-taps` | 在管线每一级存 WAV。必须同时 `--audio-native`。changelog 称这套机制为 AudioTap system |
| `--audio-debug` | 逐 tick 的音频文件和计时分析。必须同时 `--audio-native` |

`--llm-log-mode` 在 `--verbose-logs` 打开时取 `all` 或 `latest`，默认 `latest`。

`build_agent` / `build_voice_user` 的参数 `audio_taps_dir` 是程序接口上的同一件事。`cli.py` 的帮助文本列出五级：`pre-effects`、`post-noise`、`post-telephony`、`final`、`agent-input`。磁盘上的文件名模板仍然不在封存包里。

语音提交的 `prepare` 只保留每个任务的 canonical 仿真的 `audio/`。它会丢掉：非 canonical 的仿真目录、`hallucination_discarded/`、`llm_debug/`、`sim_status.json`、`task.log`。`hallucination_discarded/` 这个目录名说明幻觉重跑丢掉的轨迹会落盘；目录里的 schema UNKNOWN。

`sim_status.json` 在 changelog 里用来记录重试的来历。

## hallucination retry

**是什么**：`--hallucination-retries`，默认 3，仅全双工。用户模拟器偏离题目说明时，系统可以重跑。设为 0 关闭。v1.0.0 还描述了自动幻觉检测，以及受影响的评测可以重跑。

**不是什么**：不是 agent 幻觉工具名时的行为。agent 调用不存在的工具时，现场环境返回错误消息且不改状态，连续错误由 `max_errors` 终止。那条路径不叫 hallucination retry。

复查命令是另一条：`tau2 review`，默认模型 `claude-opus-4-5`，模式 `full` 或 `user`。`--auto-review` 可以在每次仿真后自动跑。复查结果是否写进 reward：文档把它放在质量检查，没有说它乘进 `reward_basis`。不要把它当成第六种 RewardType。

## 沟通协议、XML 提示、persona JSON

`--enforce-communication-protocol`：可选，禁止文字和工具调用混在同一条消息。默认关闭。

`--xml-prompt` / `--no-xml-prompt`：强制系统提示使用或不使用 XML 标签。默认是哪一种：CLI 表没有写默认，只写了两个开关。UNKNOWN。

`--user-persona`：JSON 字典，传给用户 persona 配置。和语音的 ElevenLabs persona 是否为同一结构：文档没有对照。UNKNOWN。不要假设一个 JSON 能同时选中 Matt Delaney 的音色。

## 语音交互指标

**是什么**：τ-voice 榜在 pass^1 旁边报的对话质量。全部从已上传的 tick 轨迹离线计算，维护者在审核时重算，提交者自己填的数字会被换掉。没有额外的裁判模型。

**不是什么**：不是任务是否及格，也不是 Full-Duplex-Bench（arXiv `2503.04721`）的数字。文档明确写两者不能直接比：对话不同、注入的信号不同、时间按 200 毫秒量化。

| 记号 | 字段 | 方向 | 一句话 |
|------|------|------|--------|
| L_R | `response_latency_mean` | 越低越好 | 用户一轮结束后，到 agent 开始回应的平均秒数 |
| L_Y | `yield_latency_mean` | 越低越好 | 用户打断后，agent 停止说话的平均秒数 |
| R_R | `response_rate` | 越高越好 | 用户那一轮在自己再次开口前得到了回应的比例 |
| R_Y | `yield_rate` | 越高越好 | 真正的打断里，agent 在窗口内让出的比例 |
| I_A | `agent_interruption_rate` | 越低越好 | agent 打断用户的次数，除以可回应的用户轮数。可以大于 1 |
| S_BC | `selectivity_backchannel` | 越高越好 | 对「mm-hmm」这种附和，agent 正确地继续说下去的比例 |
| S_VT | `selectivity_vocal_tic` | 越高越好 | 对「um」、咳嗽，agent 正确地忽略的比例 |
| S_ND | `selectivity_non_directed` | 越高越好 | 对「等一下」、旁人说话，agent 正确地忽略的比例 |

榜上的 Selectivity 列是 S_BC、S_VT、S_ND 的不加权平均，且三项都要有、每项至少 10 个事件才显示。事件少于 10 时该速率被藏起来。

检测窗口（和 03 的轮次阈值不是同一组）：

| 参数 | 默认 |
|------|------|
| 真实打断后的 no-yield 窗口 | 2.0 秒 |
| 附和 / 口头禅 / 非指向说话，agent 正在说时，在此时间内让出算错 | 1.0 秒 |
| 静默期间对口头禅或非指向说话作出回应，在此时间内算错 | 2.0 秒 |

分类优先级：backchannel，然后 vocal tic，然后 non-directed，然后才是真正的打断。

## 标准提交和自定义提交

**standard**：被测系统走基准规定的 agent 接口、工具、提示、用户模拟器、题集和评测器。语音系统内部可以有自己的 ASR、TTS、多个模型和路由，只要这些都在被测系统里面、不改基准一侧。只做传输适配，仍算 standard。`submission_type` 默认就是 `"standard"`。

**custom**：改了基准一侧。包括多模型路由、额外工具、改过的编排或提示、非默认用户模拟器、非默认语音配置、或在 τ-bench 的题和奖励上训练过。必须在 `submission.json` 里声明，并写 `methodology.notes`。

Verified 要求：有轨迹、`modified_prompts` 为 false、`omitted_questions` 为 false。否则榜上是 Unverified。

`legacy_submissions` 是旧基准版本的提交，界面上变暗并标 “v1”，默认隐藏。这个 “v1” 徽章是排行榜 UI 的记号。不是 tag `v1.0.0`，也不是 τ¹ 的论文。

## 任务划分和 solo 政策文件

`cli.py` 的 `--task-split-name` 默认是字符串 `base`。能写下来的长度如下。airline、retail 的当前数组长度不在这张表里。

| 域 | 划分 | 依据 |
|----|------|------|
| airline | 当前 train / test / base / 语音配置：**UNKNOWN** | 修订博客只写原先 **50** 道。数据摘要另写 train 30、test 20、base 50、语音 50，不升格为当前长度 |
| retail | 当前 train / test / base / 语音配置：**UNKNOWN** | 修订博客只写原先 **114** 道。数据摘要另写 train 74、test 40、base 114、语音 114，同样不升格 |
| telecom | 数据摘要：train 74，test 40，base **114**，`small` **20**，`full` **2285**，语音配置 2285 | `DATA_SUMMARY.md`。这里的 114 是 telecom 的 `base`，不是博客里的原 retail |
| banking_knowledge | 数据摘要没有划分长度。任务 **97**，文档 **698** | Release、CHANGELOG、公告，以及 97 个 `tasks/task_*.json` |
| mock | 摘要没有划分长度。文字 **10**，语音配置 **9** | `DATA_SUMMARY.md` |

telecom 排行榜默认划分：**UNKNOWN**。提交文档写「用 `base`」也写「跑完该域全部题目」。对 telecom，数据摘要里这两句分别是 114 和 2285。封存索引要求不要猜。`registry.py` 另有任务集 `telecom_full` 和 `telecom_small`，提交文档没有说榜单用它们。

README 说评测时用 `base`，以便和原先 τ-bench 的完整题集一致。这是划分的用途，不是 airline、retail 当前条数的测量。不要把这句话套到 telecom 的 2285 上，也不要用 50 和 114 填 airline、retail 的当前全长。

telecom 和 mock 的数据目录里有 `*_solo.md`。`llm_agent_solo` 的 metadata 是 `solo_mode: True`。环境实现是否因此改读 `*_solo.md`：实现未封存，UNKNOWN。

## 本页依据

`docs/evaluation.md`，`docs/cli-reference.md`，`docs/interaction-metrics.md`，`docs/leaderboard-submission.md`，`src/tau2/knowledge/README.md`，`src/tau2/config.py`，`CHANGELOG.md`，`RELEASE_NOTES.md`，Release `v1.0.0` / `v1.0.1`，`AGENTS.md`。见 [99-evidence.md](99-evidence.md)。
