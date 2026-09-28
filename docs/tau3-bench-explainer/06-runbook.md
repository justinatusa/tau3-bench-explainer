# 按官方文档跑起来

命令只收录封存文档里出现过的原文。上游仓库是 [sierra-research/tau2-bench](https://github.com/sierra-research/tau2-bench)。本说明书仓库没有 `tau2` 包，在这里执行下面的命令不会成功。

冒烟用的 `--num-tasks 5` 或 `--num-tasks 1` 是文档里的小规模示例。排行榜希望至少 4 次 trial，并要求不要用 `--num-tasks` 截断。telecom 上，数据摘要把代码默认划分名 `base` 写成 114 题，把 `tasks.json` 写成 2285。榜单要哪一份：UNKNOWN。airline、retail 的当前题数不在运行命令里，见 [01-what-is-tau3.md](01-what-is-tau3.md)。

`--max-steps-seconds` 不写时，`cli.py` 使用 `config.py` 的 1200 秒。`docs/cli-reference.md` 和 voice README 的表写 600。以代码默认值为 1200，并把 600 当成文档表里的旧数字。下面不另造一条带 600 的命令。

## 1. 安装

Getting Started 与 README：

```bash
git clone https://github.com/sierra-research/tau2-bench
cd tau2-bench
uv sync                        # core only (text-mode: airline, retail, telecom, mock)
```

按需要加 extra：

```bash
uv sync --extra voice          # + voice/audio-native features
uv sync --extra knowledge      # + banking_knowledge domain (retrieval pipeline)
uv sync --extra gym            # + gymnasium RL interface
uv sync --extra dev            # + pytest, ruff, pre-commit (required for contributing)
uv sync --extra experiments    # + plotting libs for src/experiments/
uv sync --all-extras           # everything
```

语音在 macOS 上还要：

```bash
brew install portaudio ffmpeg
```

检查数据目录：

```bash
uv run tau2 check-data
```

如果不是可编辑安装（文档举例 `uv pip install .`）：

```bash
export TAU2_DATA_DIR=/path/to/your/tau2-bench/data
```

钥匙：

```bash
cp .env.example .env
```

然后填写 `.env`。哪一把钥匙对应哪条路径，见 [04-dependencies.md](04-dependencies.md)。

看域和命令概览：

```bash
tau2 intro
```

`tau2` 不带子命令时，CLI 文档说效果与 `tau2 intro` 相同。

## 2. 文字冒烟

README 与 Getting Started：

```bash
tau2 run --domain airline --agent-llm gpt-4.1 --user-llm gpt-4.1 \
  --num-trials 1 --num-tasks 5
```

`AGENTS.md` 的同一条命令不带反斜杠续行。结果在 `data/simulations/`。

`docs/running_simulations.md` 里的其他文字示例：

```bash
tau2 run --domain airline --agent llm_agent --agent-llm openai/gpt-4.1

tau2 run --domain retail --agent llm_agent --agent-llm openai/gpt-4.1 \
    --task-ids 0 1 --num-trials 3

tau2 run --domain telecom --agent llm_agent --agent-llm openai/gpt-4.1 \
    --max-concurrency 4 --auto-resume
```

版本发布前的快速检查（`VERSIONING.md`）：

```bash
tau2 run --domain mock --num-tasks 1
```

看结果：

```bash
tau2 view
```

可选旗标：`--dir`、`--file`、`--only-show-failed`、`--only-show-all-failed`、`--expanded-ticks`。

看某个域的政策和工具（然后打开文档给出的本机地址）：

```bash
tau2 domain airline
```

文档写的地址是 `http://127.0.0.1:8004/redoc`。

交互扮演：

```bash
tau2 play
```

环境 CLI（beta）：

```bash
make env-cli
```

## 3. 语音冒烟

先建音色。默认 ID 是 Sierra 内部的，外部账号不能用。`docs/voice-personas.md`：

```bash
python -m tau2.voice.scripts.setup_voices
python -m tau2.voice.scripts.setup_voices --complexity control
python -m tau2.voice.scripts.setup_voices --complexity regular
python -m tau2.voice.scripts.setup_voices --dry-run
python -m tau2.voice.scripts.setup_voices --preview
python -m tau2.voice.scripts.setup_voices --model eleven_multilingual_ttv_v2
```

把脚本打印的 `TAU2_VOICE_ID_*=...` 放进 `.env`。只建了两位 control persona 时，用 `--speech-complexity control`。

Getting Started 的最小语音命令：

```bash
tau2 run --domain retail --audio-native --num-tasks 1 --verbose-logs
```

v1.0.0 Release 的零售示例：

```bash
tau2 run --domain retail \
  --audio-native \
  --audio-native-provider openai \
  --audio-native-model gpt-realtime-1.5 \
  --num-tasks 5 \
  --audio-taps
```

CLI 参考里的其他语音示例：

```bash
tau2 run --domain retail --audio-native --audio-native-provider gemini \
  --tick-duration 0.2 --max-steps-seconds 240 --speech-complexity control \
  --verbose-logs --save-to my_audio_native_run

tau2 run --domain retail --audio-native --hallucination-retries 0 --num-tasks 1

tau2 run --domain retail --audio-native --speech-complexity control
tau2 run --domain retail --audio-native --speech-complexity regular
```

合成试听（voice persona 文档）：

```bash
python src/tau2/voice/synthesis/cli.py synthesize "Hello, I'd like to check on my order." --play
```

格式转换：

```bash
tau2 convert-results data/simulations/my_run --to dir
tau2 convert-results data/simulations/my_run --to json
```

## 4. 知识域冒烟

需要 `uv sync --extra knowledge`。BM25 不需要 API 钥匙，也不需要沙箱。README / Getting Started：

```bash
tau2 run --domain banking_knowledge \
  --retrieval-config bm25 \
  --agent-llm gpt-4.1 \
  --user-llm gpt-4.1 \
  --num-tasks 5
```

`AGENTS.md` 的示例用的是 `qwen_embeddings`（需要 `OPENROUTER_API_KEY`）：

```bash
tau2 run --domain banking_knowledge --retrieval-config qwen_embeddings --agent-llm gpt-4.1 --user-llm gpt-4.1 --num-tasks 5
```

带重排器（还要 `OPENAI_API_KEY`）：

```bash
tau2 run --domain banking_knowledge --retrieval-config openai_embeddings_reranker \
  --agent-llm gpt-4.1 --user-llm gpt-4.1 --num-tasks 5
```

默认配置是 `alltools`，不是 `bm25`。`alltools` 需要 OpenAI 嵌入钥匙和 sandbox-runtime。安装沙箱（knowledge README）：

```bash
npm install -g @anthropic-ai/sandbox-runtime@0.0.23
```

Linux 再装 `ripgrep`、`bubblewrap`、`socat`。macOS 再装 `ripgrep`。检查：

```bash
which srt rg bwrap socat   # Linux
which srt rg               # macOS
```

## 5. 重打分和复查

v1.0.1 Release 的重打分。`--fresh-tasks` 用当前数据目录里的题目，而不是结果文件里嵌的旧题目：

```bash
tau2 evaluate-trajs --fresh-tasks path/to/results.json -o regraded/
```

CLI 参考里的基本形式（那一页的表没有列出 `--fresh-tasks`；旗标的说明在 `CHANGELOG.md` 和 Release）：

```bash
tau2 evaluate-trajs <paths...>
```

对话复查：

```bash
tau2 review <path>
```

要复现 1.0.1 之前的 `banking_knowledge` 计分，Release 写的是：

```bash
pip install git+https://github.com/sierra-research/tau2-bench@pre-v1.0.1
```

这是钉 tag 的例外。日常安装仍用 `uv sync`。`pre-v1.0.1` 指向提交 `b51a6d69e26f0e94a9173e2e80fe8735a8dff650`。

## 6. 排行榜相关命令

这些命令来自 `docs/leaderboard-submission.md` 和 CLI 参考。提交还要 fork 上游仓库、改 `manifest.json`、开 pull request。本说明书不代替那份指南。

文字，文档要求各域使用相同的 `--agent-llm` 和 `--user-llm`，不要加 `--num-tasks`：

```bash
tau2 run --domain retail --agent-llm gpt-4.1 --user-llm gpt-4.1 --num-trials 4 --save-to my_model_retail
tau2 run --domain airline --agent-llm gpt-4.1 --user-llm gpt-4.1 --num-trials 4 --save-to my_model_airline
tau2 run --domain telecom --agent-llm gpt-4.1 --user-llm gpt-4.1 --num-trials 4 --save-to my_model_telecom
```

banking 在同一份文档里单独给出：

```bash
tau2 run --domain banking_knowledge --retrieval-config alltools --agent-llm gpt-4.1 --user-llm gpt-4.1 --num-trials 4
tau2 run --domain banking_knowledge --retrieval-config alltools-qwen --agent-llm gpt-4.1 --user-llm gpt-4.1 --num-trials 4
```

打包提交：

```bash
tau2 submit prepare \
  data/simulations/my_model_retail \
  data/simulations/my_model_airline \
  data/simulations/my_model_telecom \
  --output ./my_submission
```

跳过校验：`tau2 submit prepare ... --output ./my_submission --no-verify`。

语音提交示例（必须 `regular`）。文档建议先开 PR 并联系维护者，因为用户模拟器依赖多把钥匙：

```bash
tau2 run --domain retail --audio-native \
    --audio-native-provider openai --audio-native-model gpt-4o-realtime-preview \
    --speech-complexity regular --verbose-logs \
    --save-to my_model_voice_retail

tau2 run --domain airline --audio-native \
    --audio-native-provider openai --audio-native-model gpt-4o-realtime-preview \
    --speech-complexity regular --verbose-logs \
    --save-to my_model_voice_airline

tau2 run --domain telecom --audio-native \
    --audio-native-provider openai --audio-native-model gpt-4o-realtime-preview \
    --speech-complexity regular --verbose-logs \
    --save-to my_model_voice_telecom
```

```bash
tau2 submit prepare \
  data/simulations/my_model_voice_retail \
  data/simulations/my_model_voice_airline \
  data/simulations/my_model_voice_telecom \
  --output ./my_voice_submission --voice
```

校验：

```bash
tau2 submit validate ./my_submission
tau2 submit validate <submission_dir> --mode public
tau2 submit verify-trajs <paths...>
```

`--mode` 的取值文档写的是 `public` 或 `private`。

交互指标：

```bash
tau2 submit interaction-metrics <experiment-dirs-or-trajectories-dir> --output metrics.json
```

`docs/interaction-metrics.md` 里的路径示例是：

```bash
tau2 submit interaction-metrics data/tau2/simulations/my_voice_run
```

这条路径的前缀是 `data/tau2/simulations/`。Getting Started、`AGENTS.md` 和提交文档把 `tau2 run` 的结果写在 `data/simulations/`。封存文档没有解释这两个前缀。运行 `tau2 run` 时以 `data/simulations/` 为准；上面这条是指标文档里的原样示例。

以及维护者从公开轨迹重算时用到的：

```bash
aws s3 sync s3://sierra-tau-bench-public/submissions/<dir>/trajectories/ /tmp/trajs --no-sign-request
tau2 submit interaction-metrics /tmp/trajs --output interaction_metrics.json
```

终端里看榜：

```bash
tau2 leaderboard
tau2 leaderboard --domain airline --metric pass_1
```

`--domain` 的文档取值是 `retail`、`airline`、`telecom`、`banking_knowledge`。`--metric` 可以是 `pass_1`、`pass_2`、`pass_3`、`pass_4`、`cost`。

## 7. telecom 消融

CLI 参考的 Advanced 一节。这些不是默认评测。

```bash
tau2 run \
  --domain telecom \
  --agent llm_agent_solo \
  --agent-llm gpt-4.1 \
  --user dummy_user

tau2 run \
  --domain telecom \
  --agent llm_agent_gt \
  --agent-llm gpt-4.1 \
  --user-llm gpt-4.1

tau2 run \
  --domain telecom-workflow \
  --agent-llm gpt-4.1 \
  --user-llm gpt-4.1
```

## 8. 开发者测试

`AGENTS.md` 与 CLI 参考：

```bash
make test
make test-voice
make test-knowledge
make test-gym
make test-all
make lint
make format
make lint-fix
make check-all
make clean
```

`make test` 需要事先 `uv sync --extra dev`，并且不覆盖语音、知识域和 gym。

排除要打现场 API 的集成测试：

```bash
pytest -m "not full_duplex_integration"
```

## 9. 程序接口（文档中的入口，不是 CLI）

v1.0.0 Release：

```python
from tau2.runner import run_simulation
from tau2.runner import build_text_orchestrator
from tau2.runner import run_domain
```

`docs/running_simulations.md` 还有 `TextRunConfig`、`VoiceRunConfig`、`build_voice_orchestrator`、`get_tasks`、`run_tasks`、`run_single_task`。字段和默认 `evaluation_type` 的差别见 [02-architecture.md](02-architecture.md)。这里不另造一套示例参数。

Gym 的导入出现在 0.2.1 Release Notes：

```python
from tau2.gym import AgentGymEnv, UserGymEnv
```

需要 `--extra gym`。该 README 未封存，构造参数 UNKNOWN。

## 不要从别的地方抄来的做法

- 不要把 `pip install -e .` 当成 τ³ 的当前安装步骤。那是 0.1.x / 从 τ² 升级说明里的旧方法。
- 不要把 Hugging Face `HuggingFaceH4/tau2-bench-data` 或 ModelScope `evalscope/tau3-bench-data` 当作官方题库去替换仓库 `data/`。
- 不要发明本页没有的子命令。旗标全表以仓库 `docs/cli-reference.md` 为准；本页只收了文档里完整出现过的命令。

## 本页依据

`README.md`，`docs/getting-started.md`，`docs/cli-reference.md`，`docs/running_simulations.md`，`docs/leaderboard-submission.md`，`docs/voice-personas.md`，`docs/interaction-metrics.md`，`src/tau2/knowledge/README.md`，`src/tau2/voice/README.md`，`AGENTS.md`，`VERSIONING.md`，`CHANGELOG.md`，`RELEASE_NOTES.md`，Release `v1.0.0` / `v1.0.1`。见 [99-evidence.md](99-evidence.md)。
