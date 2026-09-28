# 一次仿真怎么往下走

编排器 README 规定了两种时间形状。`run()` 的循环体不在封存包里，所以第一句种子消息从哪来仍标 UNKNOWN。

两条环路不要看成「同一个循环，只是把文本换成音频」。半双工是轮流的消息。全双工是固定步长的时间轴，两边可以在同一步里都出声。

## 半双工文字

参与者：

- **agent**：默认 `LLMAgent`。每轮调用 `generate_next_message()`。它拿到域政策、工具列表，以及到目前为止的对话。模型名来自 `--agent-llm`。不写这个旗标时，`config.py` 的默认是 `gpt-4.1-2025-04-14`，温度 `0.0`。
- **user**：默认 `user_simulator`。`running_simulations.md` 的示例把它建成 `UserSimulator(tools=env.get_user_tools(), instructions=str(task.user_scenario), llm=...)`。默认用户模型同样是 `gpt-4.1-2025-04-14`，温度 `0.0`。没有用户工具的域，`get_user_tools()` 返回什么：UNKNOWN。
- **environment**：执行工具，改数据库。`Orchestrator` 里工具调用是同步的：文档没有写「工具在后台另一条时间线上跑」。
- **orchestrator**：`Orchestrator`。示例构造参数包括 `max_steps`、`max_errors`、`seed`。CLI 默认 `max_steps=200`、`max_errors=10`、`seed=300`。文档示例里手写 `max_steps=100` 只是示例，不是默认值。

编排器 README 把半双工画成：

```
Agent ──message──> User ──message──> Agent ──tool_call──> Environment ──result──> Agent
```

同一时刻只有一方在说话。工具调用是同步的，结果在同一步返回。轨迹是一条扁平的 `list[Message]`。agent 的 `generate_next_message` 输入类型是 `UserMessage | ToolMessage | MultiToolMessage`，输出是 `AssistantMessage`。示意图从 agent 发给 user 画起，函数签名却要求先有用户或工具消息。第一轮的种子消息由谁写入：`run()` 未封存，UNKNOWN。

agent 是否可以在同一条消息里既写字又调用工具：默认不强制。加上 `--enforce-communication-protocol` 之后，协议规则会禁止「文字和工具调用混在同一条消息」。这个旗标默认 `False`。

### 工具调用出错时

changelog 对 1.0.1 的描述：现场环境遇到 agent 幻觉出来的工具调用时，返回 `ToolMessage(error=True)`，不改状态。重放轨迹时（`Environment.set_state`）现在同样把这种调用当成空操作。连续错误由 orchestrator 的 `max_errors` 拦住，停止原因是 `TerminationReason.TOO_MANY_ERRORS`。

**10**：是默认的 `max_errors`，CLI 文档写的是「最多允许的连续工具错误」。不是「整次对话最多 10 次错误」。连续怎样计数（中间成功一次是否清零）：封存文档只写了 consecutive，没有写清零规则的代码。按「连续」理解，不要把它当成累计总数。

### 半双工何时停

`evaluator.py` 只把两种终止原因当成正常结束：`AGENT_STOP` 和 `USER_STOP`。其他 `termination_reason` 在评测器运行之前就得到奖励 0。步数用尽、连续工具错误，都会落进这扇门，除非实现把它们也标成这两种原因。实现没有封存，所以「`TOO_MANY_ERRORS` 是否算提前结束」按评测器的字面意思：只要原因不是那两个枚举值，奖励就是 0。

| 条件 | 默认 | 依据 |
|------|------|------|
| 步数到达 `--max-steps` | 200 | `cli.py` 把默认值绑到 `DEFAULT_MAX_STEPS` |
| 连续工具错误到达 `--max-errors` | 10 | changelog 的停止原因是 `TerminationReason.TOO_MANY_ERRORS` |
| 墙钟时间到达 `--timeout` | 不限 | `cli.py`：`default=None` |
| 失败任务的重试 | `--max-retries` 默认 3 | 这是整次任务重跑，不是对话内的一步 |
| 正常结束 | `AGENT_STOP` 或 `USER_STOP` | 否则 `evaluate_simulation` 返回 0 |

用户用什么记号触发 `USER_STOP`（文字模式下有没有 `###STOP###`）：UNKNOWN。`###STOP###` 只在语音交互指标文档里出现，见下一节。

种子 **300** 是默认随机种子（`DEFAULT_SEED`）。它让试验可重复，不是模型温度。温度默认已经是 0.0。

并发默认 **3**（`--max-concurrency`）。trial 默认 **1**。排行榜文档希望至少 4 次 trial，那是提交建议，不是 CLI 默认。

## 全双工语音

打开方式是 `--audio-native`。这时 `build_voice_orchestrator` 造一个 `FullDuplexOrchestrator`。agent 侧是 `DiscreteTimeAudioNativeAgent`，通过 `DiscreteTimeAdapter` 把供应商的 WebSocket 音频流收成 tick。用户侧是 `voice_streaming_user_simulator`。

编排器 README 把全双工画成每个 tick 上 agent 和 user 各产出一块 chunk，两块可以重叠。工具调用仍然同步，并且在该 tick 结束前返回。轨迹不是扁平消息列表，而是 `list[Tick]`。每个 `Tick` 有 `tick_id`、`agent_chunk`、`user_chunk`、双方的工具调用和结果、`user_transcript`，以及配置的 tick 时长和实际墙钟时长。`cli.py` 的 `--audio-native-provider` 接受 `openai`、`openai_live`、`gemini`、`xai`、`nova`、`qwen`、`livekit`。没有 `deepgram` 这个选项。Deepgram 出现在 LiveKit 级联（STT→LLM→TTS）的说明里，也出现在转写 SDK 里。

### tick 是什么

**0.2 秒**：是默认 tick 长度。CLI `--tick-duration` 默认 `0.2`，`config.py` 的 `DEFAULT_TICK_DURATION_SECONDS = 0.20`。交互指标按这个粒度量化时间，并写明和 Full-Duplex-Bench 的数字不能直接比，原因之一就是 200 毫秒量化。

每个 tick 记录：这一步用户和 agent 是否在出声（`contains_speech`）、用户模拟器的轮次决定（`turn_taking_action`）、注入了哪些音效（`speech_effects` / `source_effects`）。

同一 tick 里两边都可以出声。这就是文档说的 full-duplex：不是「等对方说完再开口」的对讲机。

### 用户模拟器在语音里做的事

它不是一个只负责朗读的 TTS。提交文档把它写成多组件系统：LLM、ElevenLabs TTS、Deepgram 转写、音效管线、决策模型。

能从文档拼起来的顺序是：

1. 决策模型决定这一拍怎么说话。`config.py` 里 `VOICE_USER_SIMULATOR_DECISION_MODEL = "gpt-4.1"`，注释写可覆盖。版本号 `VOICE_USER_SIMULATOR_VERSION = "v1.0"`，注释写改行为时要 bump，并在提交文档里对应 git tag `voice-user-sim-<version>`。封存包没有这个 tag 的 SHA。
2. ElevenLabs 把要说的字合成 PCM。默认合成采样率 16000 Hz（`--pcm-sample-rate`）。默认 TTS 模型名 `eleven_v3`。
3. 音效管线按 `--speech-complexity` 加背景噪声、突发声（汽车喇叭、狗叫等）、电话压缩、丢帧。噪声文件在 `data/voice/background_noise_audio_pcm_mono_verified/`，封存清单有 9 个 wav：突发声 `car_horn`、`dog_bark`、`engine_idling`、`ringing_phone`、`siren`；持续噪声 `busy_street_iphone_mic`、`medium_size_room_tv_news_iphone_mic`、`people_talking`、`street_and_metro_station_iphone_mic`。
4. 送给 agent 的音频是电话格式：G.711 μ-law，8000 Hz，每样本 1 字节。静音字节常量是 `b"\x7f"`。

**8000** 在这里是电话采样率（Hz），也是 `--telephony-rate` 的默认。不是 `API_PORT` 的 8000。两个 8000 来自不同的常量。

persona 决定音色。`control` 用两位美式口音：Matt Delaney、Lisa Brenner。`regular` 再用五位：Mildred Kaplan、Arjun Roy、Wei Lin、Mamadou Diallo、Priya Patil。默认音色 ID 是 Sierra 内部的，外部账号调用会失败，必须用 `TAU2_VOICE_ID_*` 换成自己在 ElevenLabs 建的音色。建音色的命令见 06。

轮次阈值（CLI 默认，单位秒）。这些是用户模拟器的参数，不是交互指标里的检测窗口。两套数字不要混用。

| CLI 旗标 | 默认 | 文档中的含义 |
|----------|------|----------------|
| `--wait-to-respond-other` | 1.0 | 用户在 agent 说过话之后，至少再等这么久才回应 |
| `--wait-to-respond-self` | 5.0 | 用户自己说过话之后，至少再等这么久才再说 |
| `--yield-when-interrupted` | 1.0 | agent 打断用户时，用户还继续说这么久 |
| `--yield-when-interrupting` | 5.0 | 用户打断 agent 时，用户还继续说这么久 |
| `--interruption-check-interval` | 2.0 | 检查打断的间隔 |
| `--integration-duration` | 0.5 | 线性化用的整合时长 |
| `--silence-annotation-threshold` | 4.0 | 标注用的静音阈值 |

`config.py` 还有附和声（backchannel）的默认：最短 3 秒、最长 12 秒、泊松率 `1/10`，以及 `DEFAULT_USE_LLM_BACKCHANNEL = True`。这些常量怎样映射到 CLI：封存的 CLI 表没有同名旗标。标 UNKNOWN。

`--speech-complexity` 默认 `regular`（有噪声、口音、打断）。`control` 是干净基线：无音效、美式口音、耐心的用户。消融预设：`control_audio`、`control_accents`、`control_behavior`，以及两两组合。排行榜语音提交必须是 `regular`；`prepare` 会跳过其他复杂度。

### agent 侧的实时音频

供应商适配器把流式 API 收成 tick。完全支持、并且 CLI 文档点名的三个是 `openai`、`gemini`、`xai`。默认供应商 `openai`，默认模型在 `config.py` 里是 `gpt-realtime-1.5`。各供应商的钥匙和端点见 04。

OpenAI 输出采样率常量是 24000 Hz（API 规定）。Gemini 输入 8000、输出 24000。Nova 与 Qwen 输入 16000、输出 24000。适配器内部怎样把这些采样率变到电话侧的 8000 Hz：封存的 voice README 只写了用户侧合成路径会转成 μ-law 8 kHz。agent 音频的重采样步骤标 UNKNOWN。

### 全双工何时停

| 条件 | 文档中的默认 | 注意 |
|------|----------------|------|
| `--max-steps-seconds` | **1200** | `cli.py` 把旗标默认值设为 `DEFAULT_MAX_STEPS_SECONDS`，该常量是 1200。`docs/cli-reference.md` 和 voice README 的表写着 600，那是文档表，不是 `cli.py` 的默认。排行榜示例 JSON 里的 600 是示例值 |
| `--timeout` | 不限 | 墙钟，和上一项不是同一个旗标 |
| `--hallucination-retries` | 3 | 仅全双工。检测到用户模拟器偏离题目说明时重跑。设为 0 关闭 |
| `max_errors` | 10 | changelog 把 `TOO_MANY_ERRORS` 写成 orchestrator 的守卫。语音路径是否使用同一个计数器：文档没有单独说明。不要假设语音没有这个上限，也不要假设计数方式和文字完全相同 |
| 对话结束记号 `###STOP###` | 指标文档说对话以此结束 | 它会在末尾制造一个多余的、只有 1 tick 的用户语音。算交互指标前会删掉这个 tick。它是不是 orchestrator 的唯一正常结束条件：UNKNOWN |

停滞检测常量 `DEFAULT_AUDIO_NATIVE_MAX_INACTIVE_SECONDS = 40.0`，注释写 stall detection。它是否直接终止仿真：UNKNOWN。

音频供应商连接失败时的重试：`DEFAULT_AUDIO_NATIVE_MAX_RETRIES = 3`，间隔 `DEFAULT_AUDIO_NATIVE_RETRY_DELAY_SECONDS = 5.0`。这和任务级 `--max-retries`（默认 3，但 CLI 文档里 `--retry-delay` 默认 1.0 秒）不是同一组数字。

### 语音结果里能看到什么

`--verbose-logs` 会保存音频、LLM 日志和 tick。`--audio-taps` 在管线每一级存 WAV，必须同时有 `--audio-native`。`cli.py` 的帮助文本点名这些级：`pre-effects`、`post-noise`、`post-telephony`、`final`、`agent-input`。`--audio-debug` 存逐 tick 音频和计时，同样要求 `--audio-native`。

`tau2 view --expanded-ticks` 展开全双工的 tick。

转写发生在评测侧，用来做检查和指标，不是 agent 的实时 API 本身。默认转写模型常量 `DEFAULT_VOICE_TRANSCRIPTION_MODEL = "nova-3"`。OpenAI 侧另有 `DEFAULT_OPENAI_TRANSCRIPTION_MODEL = "gpt-4o-transcribe"`。Gemini 在 main 这版把转写语言码默认设为 `["en-US"]`（`DEFAULT_GEMINI_TRANSCRIPTION_LANGUAGE_CODES`），与 main 提交说明「Gemini en-US transcription hint」相符。

## 两边共用的外层循环

不管半双工还是全双工，`run_tasks` 在外面还有一层：

- 同一道题跑 `--num-trials` 次。
- `--max-concurrency` 路并行，默认 3。
- 任务失败时按 `--max-retries` 重试，默认 3。
- `--auto-resume` 从已有存档接着跑，不再询问。
- 跑完（或按 `evaluation_type`）调用 evaluator。
- 可选 `--auto-review`：再用一个 LLM 做对话复查，默认复查模型 `claude-opus-4-5`，模式 `full`（agent 和 user）或 `user`。

复查和 hallucination retry 不是任务分数本身。分数由 `reward_basis` 决定。用户模拟器如果被判定幻觉，全双工可以整段重跑，默认最多 3 次。

## 本页依据

`AGENTS.md`，`src/tau2/config.py`，`docs/cli-reference.md`，`docs/running_simulations.md`，`docs/interaction-metrics.md`，`docs/getting-started.md`，`docs/voice-personas.md`，`src/tau2/voice/README.md`，`CHANGELOG.md`，`data_tree_summary.json`。见 [99-evidence.md](99-evidence.md)。
