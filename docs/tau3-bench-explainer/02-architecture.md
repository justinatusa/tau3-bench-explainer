# 程序分成哪几块

这一页讲模块怎么拼。一次仿真在时间上怎么走，见 [03-agent-loop.md](03-agent-loop.md)。封存包有编排器、agent、域的 README，以及 `evaluator.py` 的入口和 `registry.py`。编排器的 `run()` 循环体仍然不在包里。

## 一次评测的拓扑

```
tau2 run / run_domain()
    runner 批处理（并发、断点、重试、写盘）
        对每一道题、每一次 trial：
            registry 按名字取出 domain、agent、user
            orchestrator
                agent          被测的客服
                user           模拟顾客
                environment    政策 + 工具 + 数据库
            仿真结束后 evaluator 计算 reward
    metrics（avg_reward、pass_hat_ks 等）
    写入 data/simulations/<run_name>/
```

`AGENTS.md` 把编排器写成两个实现：

| 类 | 通信 | 工具执行 |
|----|------|----------|
| `Orchestrator` | 半双工，一轮一轮 | 同步 |
| `FullDuplexOrchestrator` | 全双工，按 tick | agent 和 user 可以在同一 tick 里都有活动 |

Agent 也有两个基类，构造函数相同：`__init__(self, tools, domain_policy)`。

| 模式 | 基类 | 往前走一步的方法 | 默认实现 |
|------|------|------------------|----------|
| 半双工 | `HalfDuplexAgent` | `generate_next_message()` | `LLMAgent`，注册名 `llm_agent` |
| 全双工 | `FullDuplexAgent` | `get_next_chunk()` | `DiscreteTimeAudioNativeAgent`，注册名 `discrete_time_audio_native_agent` |

用 LLM 的 agent 再混入 `LLMConfigMixin`，从而带上 `llm` 和 `llm_args`。

用户侧的默认注册名：半双工是 `user_simulator`；全双工是 `voice_streaming_user_simulator`。`build_user` 造半双工用户；`build_voice_user` 造全双工语音用户。

## runner 的三层

`docs/running_simulations.md` 把 `tau2.runner` 分成三层。上层调用下层，也可以从中间进入。

| 层 | 模块 | 做什么 | 你什么时候用 |
|----|------|--------|----------------|
| 1 | `runner.simulation` | 跑一个已经建好的 orchestrator。不查 registry | `run_simulation(orchestrator)` |
| 2 | `runner.build` | 用名字和配置，经 registry 造出 environment、agent、user、orchestrator | `build_text_orchestrator` / `build_voice_orchestrator` |
| 3 | `runner.batch` | 并发、断点、重试、日志 | `run_domain`、`run_tasks`、`run_single_task` |

CLI 的 `tau2 run` 走第 3 层。配置类型是 `TextRunConfig`（文字）或 `VoiceRunConfig`（语音）。`RunConfig` 是这两者的联合类型。两者共有的字段在 `BaseRunConfig`，文档点名的有 `domain`、`num_trials`、`seed`、`hallucination_retries`。

v1.0.0 Release 写的三个入口：

```python
from tau2.runner import run_simulation          # 低：跑一个 orchestrator
from tau2.runner import build_text_orchestrator  # 中：从配置造
from tau2.runner import run_domain              # 高：整批
```

`run_domain` 负责载入题目、过滤、保存路径和指标。`run_tasks` 的默认 `evaluation_type` 是 `ALL_WITH_NL_ASSERTIONS`。`run_simulation` 的默认是 `ALL`。这两个默认不一样：批处理会把 NL assertion 算进诊断，单次 `run_simulation` 默认不算。它们是否因此改变最终 reward，还要看题目自己的 `reward_basis` 里有没有 `NL_ASSERTION`。见 05。

## registry

`src/tau2/registry.py` 在 import 时登记这些名字（封存的就是这份文件）：

| 种类 | 登记名 |
|------|--------|
| 用户 | `user_simulator`、`dummy_user`；装了语音依赖时还有 `voice_streaming_user_simulator` |
| agent | `llm_agent`、`llm_agent_gt`（带 `LLMGTAgent.check_valid_task`）、`llm_agent_solo`（metadata `solo_mode: True`）、`discrete_time_audio_native_agent` |
| 域 | `mock`、`airline`、`retail`、`telecom`（manual policy）、`telecom-workflow`（workflow policy）、`banking_knowledge` |
| 任务集 | `mock`、`airline`、`retail`、`telecom`、`telecom-workflow`、`telecom_full`、`telecom_small`、`banking_knowledge` |

`telecom_full` 绑定 `get_tasks_full`，`telecom_small` 绑定 `get_tasks_small`。它们是任务集名字，不是第六、第七个业务域。`--task-set-name` 不写时，CLI 使用该域的默认任务集。telecom 默认任务集仍走带划分的 `get_tasks`，代码里的默认划分名是字符串 `base`，不是任务集名 `telecom_full`。数据摘要把 telecom 的 `base` 写成 114 题，把 `tasks.json` / `telecom_full` 写成 2285。这里的 114 是 telecom 的划分长度，不是修订博客里原先 retail 的 114。榜单提交是否改用 `telecom_full`：UNKNOWN。

文档给出的登记方式：

```python
registry.register_agent_factory(create_my_agent, "my_agent")
registry.register_domain(get_environment, "my_domain")
registry.register_tasks(get_tasks, "my_domain", get_task_splits=get_tasks_split)
```

新的 agent、domain、user 不在这里登记，CLI 就用不了。`llm_agent_solo` 的 metadata 标明 `solo_mode: True`。`build_environment(..., solo_mode=True)` 时环境内部改读哪份政策：环境实现未封存。标 UNKNOWN。不要因为磁盘上有 `main_policy_solo.md` 就断言 solo 模式会打开它。

## environment、domain、数据库

`environment/` 放环境、数据库、server、toolkit 基类。一个域目录 `src/tau2/domains/<name>/` 通常有：

| 文件 | 作用 |
|------|------|
| `data_model.py` | 该域数据库的 Pydantic 模型 |
| `tools.py` | `ToolKitBase` 子类，agent 能调用的工具 |
| `environment.py` | `get_environment()`、`get_tasks()`、`get_tasks_split()` |
| `user_tools.py` | 可选。用户模拟器能调用的工具 |
| `utils.py` | 数据路径 |

`banking_knowledge` 在这套之外还有 `retrieval.py`、`retrieval_mixins.py`、`retrieval_toolkits.py`、`db_query.py`。它的工具和提示词随 `--retrieval-config` 变。检索实现在 `src/tau2/knowledge/`，不在域目录里。

数据目录是 `data/tau2/domains/<name>/`。封存清单的 `data/` 顶层只有 `tau2` 和 `voice`。`data/voice/` 放背景噪声和突发声的 wav，不是题目。

`build_environment(domain, solo_mode=False)` 从域名造环境。CLI 上与之对应的消融是 telecom 的 `--agent llm_agent_solo --user dummy_user`，见 05。

## evaluator

仿真结束之后才打分。`docs/evaluation.md` 把奖励写成 `reward_basis` 里各项的乘积。五项 `RewardType`：

| RewardType | 谁来算 | 检查什么 |
|------------|--------|----------|
| `DB` | `EnvironmentEvaluator` | 预测环境的数据库哈希是否等于目标哈希。目标哈希 = 干净环境重放 `evaluation_criteria.actions` |
| `ENV_ASSERTION` | `EnvironmentEvaluator` | `env_assertions` 是否都在预测环境上成立 |
| `COMMUNICATE` | `CommunicateEvaluator` | `communicate_info` 里每个字符串是否出现在 agent 的消息里（子串） |
| `NL_ASSERTION` | `NLAssertionsEvaluator` | LLM 是否判定每条自然语言断言为真。文档标成实验性 / WIP |
| `ACTION` | `ActionEvaluator` | agent 的工具调用是否匹配 `actions` 里的每一条 |

airline、retail、telecom 的默认 `reward_basis` 是 `["DB", "COMMUNICATE"]`。`ACTION` 不在里面。DB 哈希、参考轨迹和「多余的读调用」见 05。

每种评测器都有半双工和全双工两套。全双工先把 `Tick` 列表转成消息再打分。环境评测器转写工具调用时，每个 tick 里先取用户、再取 agent。`evaluate_simulation()` 按 `CommunicationMode` 选套，再按题目的 `reward_basis` 把各项相乘。

如果 `termination_reason` 不是 `AGENT_STOP` 或 `USER_STOP`，函数在任何评测器之前直接返回奖励 0。缺了 `evaluation_criteria` 则返回 1.0，意思是「没有标准可评」，不是「失败」。

现场评测的重放是严格的（`strict_replay=True`）。重打分旧轨迹时传入 `False`，这样工具输出只是写法不同（例如 `25` 和 `25.0`）不会让重放中断。这和 v1.0.1 的 Release 说明一致。

evaluator 在 v1.0.0 的 changelog 里写了「5 new modules」做 review、hallucination detection 和 auth classification。`evaluator/AGENTS.md` 点名了 `reviewer.py` 和 `hallucination_reviewer.py`。其余文件名不在封存清单的正文里。不把「5」拆成五个文件名。

CLI 批处理默认的 `EvaluationType.ALL_WITH_NL_ASSERTIONS` 会填上 `action_checks`，并给出 `partial_action_reward`（参考动作匹配了 m/n）。这是诊断，不是排行榜分数。排行榜用题目自己的 `reward_basis`。

## 语音模块在架构里的位置

`src/tau2/voice/` 不替代 orchestrator。它给全双工编排器提供适配器和用户侧音频：

| 子目录 | 作用 |
|--------|------|
| `audio_native/` | 实时供应商适配器。每个供应商实现 `DiscreteTimeAdapter`，把供应商的流式 API 接到 tick 仿真上。封存包没有收录该目录的 README 正文 |
| `synthesis/` | 用户说话：ElevenLabs TTS，再叠加背景噪声、突发声、丢帧，并转成电话格式 G.711 μ-law 8 kHz |
| `transcription/` | 评测用的语音转文字。Deepgram（nova-2、nova-3）和 OpenAI（whisper-1、gpt-4o-transcribe、gpt-4o-mini-transcribe） |
| `utils/` | 音频格式、WAV |

`metrics/voice_interaction_metrics.py` 在仿真之后，从 tick 轨迹离线计算延迟、抢话、选择性。它不改变任务是否及格。定义见 05 和上游 `docs/interaction-metrics.md`。

## 知识模块在架构里的位置

`src/tau2/knowledge/` 是可插拔的检索管线，目前文档说只有 `banking_knowledge` 在用。changelog 列出的零件：embedder（OpenAI、经 OpenRouter 的 Qwen）、retriever（BM25、余弦、grep）、后处理（BGE reranker、逐点 LLM reranker、Qwen reranker）、文档和输入的预处理器。磁盘缓存目录是 `data/.embeddings_cache`（gitignore）。

`--retrieval-config` 决定 agent 看见哪些工具。默认 `alltools`：BM25、稠密向量、只读 shell。配置表在 04 和 05。

## 旁边的入口，不是主评测路径

这些模块在树上，但不改变「`tau2 run` 打一次分」的主路径：

| 模块 | 文档里的角色 |
|------|----------------|
| `gym/` | Gymnasium 环境 `AgentGymEnv`、`UserGymEnv`。`uv sync --extra gym`。`tau2 play` 让人亲手扮演 agent 或用户 |
| `api_service/` | FastAPI 服务。依赖里有 `fastapi` 和 `uvicorn`。`tau2 start` 启动全部域服务器。`tau2 domain <domain>` 之后文档让你打开 `http://127.0.0.1:8004/redoc` 看政策和工具 |
| `scripts/` | CLI 子命令的实现 |
| `src/experiments/` | 研究代码，自包含，文档说不保证支持 |
| `metrics/` | 从结果算 `AgentMetrics`（`avg_reward`、`pass_hat_ks`） |

**8000** 是 `config.py` 的 `API_PORT`。**8004** 是域文档查看器的端口，写在 CLI 文档里。封存的 `cli.py` 没有出现 8004。这两个端口是否由同一进程：UNKNOWN。不要把 8004 当成 `API_PORT`。

## 结果放在哪

文字跑完：

```
data/simulations/<run_name>/results.json
```

语音跑完：

```
data/simulations/<run_name>/
├── results.json          # 元数据和题目，不含每条仿真的全部 tick
├── simulations/sim_*.json
└── artifacts/            # 需要 --verbose-logs
```

`tau2 convert-results` 在「一个大 JSON」和「一目录」之间转换。`Results.load()` 会自己判断磁盘上的格式。

## 本页依据

`AGENTS.md`，`docs/running_simulations.md`，`docs/evaluation.md`，`docs/getting-started.md`，`docs/cli-reference.md`，`src/tau2/voice/README.md`，`src/tau2/knowledge/README.md`，`src/tau2/config.py`，`CHANGELOG.md`，`data_tree_summary.json`。见 [99-evidence.md](99-evidence.md)。
