# 要装什么、连哪里、用哪把钥匙

安装命令的可复制形式在 [06-runbook.md](06-runbook.md)。这一页解释每一组依赖在评测里干什么。版本号都来自封存的 `pyproject.toml`、`config.py` 和文档，不是事后查的最新版。

## Python 和安装器

**`>=3.12, <3.14`**：是 `pyproject.toml` 的 `requires-python`。不是「3.12 或任意更高版本」。3.14 被排除。`.python-version` 把 uv 会下载的版本钉在 3.12。Getting Started 里的「Python 3.12+」要和这个上界一起读，否则会把 3.14 算进去。

**`>=3.10`**：是 τ³ 之前的要求（0.1.0 Release Notes，以及 README 的升级说明「was >=3.10」）。不是当前 τ³ 的要求。

安装器是 [uv](https://docs.astral.sh/uv/getting-started/installation/)。`uv sync` 按锁文件造虚拟环境并提供 `tau2` 命令。默认路径不再是 `pip install -e .`。唯一在 Release 里仍写成 `pip install git+...` 的，是为了钉 `pre-v1.0.1` 这一个 tag。

不用可编辑安装时（文档举例 `uv pip install .`），必须自己设 `TAU2_DATA_DIR` 指向仓库的 `data/`。可编辑安装会自己找到数据目录。`tau2 check-data` 用来确认文件在。

包版本字符串是 `1.0.1`，许可证 MIT。作者字段写的是 Victor Barres、Honghua Dong。构建后端是 hatchling。

## uv extras：装了才有哪一块

| extra | 你得到什么 | 不装的后果 |
|-------|------------|------------|
| （无，`uv sync`） | 文字模式的 airline、retail、telecom、mock | 没有语音库，没有 `banking_knowledge` 的检索依赖 |
| `voice` | 全双工语音 / audio-native | `--audio-native` 缺少 elevenlabs、deepgram、websockets、pyaudio 等 |
| `knowledge` | `banking_knowledge` 的检索（`rank-bm25`、`openai`） | 跑不了这个域的检索配置 |
| `gym` | Gymnasium RL 接口 | `tau2 play` 的程序化 Gym 路径需要它；是否连 `tau2 play` 命令本身都缺：UNKNOWN |
| `dev` | pytest、ruff、pre-commit | 跑不了 `make test` |
| `experiments` | plotly、matplotlib、seaborn、scikit-learn | 只影响 `src/experiments/` 的作图 |
| `live` | 依赖 `tau2[voice]`，再加 `aiortc==1.15.0` 和 `httpx[socks]` | 文档没有把它写成跑 `--audio-native` 的必需项。它和实时语音的差别：UNKNOWN |
| `all` | voice、live、knowledge、gym、dev、experiments | `uv sync --all-extras` 是文档里的「全部」写法 |

核心依赖（总是安装）里和架构直接相关的是：`litellm>=1.80.15,<1.82.7`（LLM 调用）、`fastapi>=0.115.11` 与 `uvicorn>=0.34.0`（API 服务）、`typer`（CLI）、`deepdiff`（比较）。`pyproject.toml` 的依赖表没有单独写 `pydantic`；`AGENTS.md` 要求数据模型用 Pydantic `BaseModel`。`litellm` 的上界是 `<1.82.7`，不是「任意新版 litellm」。

`langfuse` 和 `redis` 不在依赖表里。`AGENTS.md` 写：只有把 `config.py` 里的 `USE_LANGFUSE` 或 `LLM_CACHE_ENABLED` 打开时才要手动 `uv pip install langfuse redis`。两者默认都是 `False`。

### 语音的系统包

macOS，文档原文：`brew install portaudio ffmpeg`。Linux 对应包名：UNKNOWN（封存的 Getting Started 只写了 macOS）。

`banking_knowledge` 若用 shell 类检索，还要 Node 全局包和系统沙箱工具，见下文检索一节。那不是 `uv` extra。

## 密钥

`.env.example` 里出现的名字：

| 变量 | 文档说它供给谁 |
|------|----------------|
| `OPENAI_API_KEY` | LiteLLM 下的 OpenAI 模型；OpenAI Realtime；`alltools` / `openai_embeddings*` 的嵌入；`*_reranker` 的 LLM 重排 |
| `ANTHROPIC_API_KEY` | LiteLLM 下的 Anthropic 模型。复查默认模型是 `claude-opus-4-5`，会用到它 |
| `ELEVENLABS_API_KEY` | 用户模拟器 TTS |
| `DEEPGRAM_API_KEY` | 转写（nova-2、nova-3） |
| `OPENROUTER_API_KEY` | `qwen_embeddings*` 和 `alltools-qwen` |
| `PINE_API_KEY` | 只给 Pine 的 OpenAI 兼容实时模型（`pine-*`） |
| `PINE_REALTIME_BASE_URL` | Pine 的 WebSocket 地址，由使用者填写 |
| `TAU2_VOICE_ID_*` | 七个 persona 的 ElevenLabs voice id。示例文件里是注释，不填则用仓库内置 ID |

voice README 另外列出、但 `.env.example` 没写的：

| 变量 | voice README 的说法 |
|------|---------------------|
| `GOOGLE_API_KEY` | Gemini Live |
| `XAI_API_KEY` | xAI Grok Voice |

`config.py` 对 Gemini 的注释是另一套名字：`GEMINI_API_KEY` 走 AI Studio；`GOOGLE_SERVICE_ACCOUNT_KEY` 或 `GOOGLE_APPLICATION_CREDENTIALS` 走 Vertex AI，区域常量 `us-central1`。适配器源码未封存，运行时到底读 `GOOGLE_API_KEY` 还是 `GEMINI_API_KEY`：UNKNOWN。两处都记在这里，不要只留一个。

Nova Sonic 用 AWS（`boto3`、区域常量 `us-east-1`）。凭证变量名：UNKNOWN（不在 `.env.example`）。

Qwen 实时语音的钥匙变量名：UNKNOWN。嵌入走的是 `OPENROUTER_API_KEY`，和实时语音不是同一个用途。

`XAI_API_KEY` 是否还有别名：UNKNOWN。

## 模型默认值

文字侧（可被 CLI 覆盖）：

| 用途 | `config.py` 默认 |
|------|------------------|
| agent | `gpt-4.1-2025-04-14`，温度 0.0 |
| user simulator | 同上 |
| NL assertion 评判 | 同上，温度 0.0 |
| 环境接口 LLM | 同上 |
| 对话复查 `--review-model` | `claude-opus-4-5` |
| `DEFAULT_LLM_EVAL_USER_SIMULATOR` | `claude-opus-4-5`。这个常量被谁调用：UNKNOWN |

排行榜提交文档建议用户模拟器用 `gpt-5.2`，并说这会印在榜上。这是提交建议。不是 `config.py` 的默认。

语音 agent 侧，`config.py` 注册表（main 摘录）：

| provider 键 | 默认模型 | 类型字段 | 推理 effort 默认 |
|-------------|---------|----------|------------------|
| `openai` | `gpt-realtime-1.5` | audio_native | 无 |
| `openai_live` | `gpt-live-1-diamond-alpha`（注释：limited-access alias） | audio_native | 无 |
| `gemini` | `gemini-3.1-flash-live-preview` | audio_native | `high` |
| `xai` | `grok-voice-think-fast-2.0` | audio_native | `high` |
| `nova` | `amazon.nova-2-sonic-v1:0` | audio_native | 无 |
| `qwen` | `qwen3.5-omni-plus-realtime` | audio_native | 无 |
| `livekit` | `dummy` | cascaded | 无 |

**7**：在 v1.0.0 Release 里是「7 real-time voice providers」。名单是：完全支持的 OpenAI Realtime、xAI Grok Voice、Gemini Live；实验性的 Nova Sonic、Qwen、Deepgram（cascaded）、LiveKit（cascaded）。

**7**：在 main 的 `cli.py` 里是 `--audio-native-provider` 的 `choices`：`openai`、`openai_live`、`gemini`、`xai`、`nova`、`qwen`、`livekit`。`config.py` 的模型表是同一组键。没有 `deepgram` 这个键。CLI 参考文档的表只写了 `openai`、`gemini`、`xai`，那是文档表比代码窄，不是代码拒绝另外四个名字。

Deepgram 在当前树里仍是转写 SDK，并且 `audio_native/README.md` 把 LiveKit 写成级联：LiveKit + Deepgram + OpenAI。不要把 Deepgram 说成 `cli.py` 里的一个 provider 选项。

`audio_native/README.md` 的模型列和 `config.py` 有两处不一致，两处都保留：xAI 在 README 表里写 `xai-realtime`，`config.py` 的默认是 `grok-voice-think-fast-2.0`；Qwen 在 README 表里写 `qwen3-omni-flash-realtime`，`config.py` 的默认是 `qwen3.5-omni-plus-realtime`。跑起来的默认以 `config.py` 被 `cli.py` 引用的常量为准。

旧模型常量还留着：`_LEGACY_OPENAI_REALTIME_MODEL = "gpt-realtime-2025-08-28"`，`_LEGACY_GEMINI_MODEL = "gemini-live-2.5-flash-native-audio"`。它们不是当前默认。

xAI 注释：别名 `grok-voice-latest` 自 2026-08-05 起指向 `grok-voice-think-fast-2.0`。`reasoning.effort` 接受 `high` 或 `none`，API 默认 `high`。

用户侧 TTS 默认 `eleven_v3`，决策模型 `gpt-4.1`。嵌入：`alltools` 用 OpenAI `text-embedding-3-large`；`alltools-qwen` 用 `qwen3-embedding-8b`（经 OpenRouter）。

Qwen 实时模型的速率限制写在 `config.py` 注释里：`qwen3.5-omni-plus-realtime` 为每分钟 60 次请求、每分钟 100000 token。这是注释中的供应商限额，不是 τ³ 自己的并发默认（并发默认是 3）。

## 检索后端

省略 `--retrieval-config` 时，`banking_knowledge` 默认 **`alltools`**：BM25 + 稠密检索 + 只读 shell。要离线、不要钥匙、不要沙箱，文档让你改用 `bm25`。

历史上，从 `2be6916` 到 `pre-v1.0.1` 这个窗口里，默认曾从 `bm25` 改成 `alltools`。那是仿真侧变化。当前默认以 knowledge README 为准，是 `alltools`。

| 配置 | 给 agent 的工具 | 钥匙 / 系统 |
|------|-----------------|-------------|
| `no_knowledge` | 无 | 离线。提示词里放什么：UNKNOWN |
| `full_kb` | 无 | 离线。是否把全文放进提示词：UNKNOWN |
| `golden_retrieval` | 无 | 离线。是否只放入该题相关文档：UNKNOWN |
| `grep_only` | `grep` | 离线 |
| `bm25` | `KB_search` | 离线 |
| `openai_embeddings` | `KB_search` | `OPENAI_API_KEY` |
| `qwen_embeddings` | `KB_search` | `OPENROUTER_API_KEY` |
| `terminal_use` | `shell`（只读沙箱） | sandbox-runtime |
| `terminal_use_write` | `shell` | sandbox-runtime。和 `terminal_use` 的写权限差别，README 只在表里分成两行，没有更细的权限表。标「写」来自名字，具体允许的写操作 UNKNOWN |
| `alltools` | `KB_search_bm25`、`KB_search_dense`、`shell` | OpenAI 嵌入 + sandbox-runtime |
| `alltools-qwen` | 同上 | Qwen 嵌入 + sandbox-runtime |

`bm25`、`openai_embeddings`、`qwen_embeddings` 还可以加后缀 `_reranker`、`_grep`，或两个都加，例如 `openai_embeddings_reranker_grep`。`*_reranker` 即使用 Qwen 做嵌入，重排器仍要 `OPENAI_API_KEY`。

**12**：是 v1.0.0 changelog 的说法：「12 retrieval configurations」，并点名离线六种（`no_knowledge`、`full_kb`、`golden_retrieval`、`bm25`、`bm25_grep`、`grep_only`）、嵌入类（`openai_embeddings*`、`qwen_embeddings*`）、以及 `terminal_use`、`terminal_use_write`。不是 knowledge README 那张表的行数。那张表是 11 行基座配置，其中多了后来作为默认的 `alltools` 和 `alltools-qwen`，少了被写成后缀的 `bm25_grep`。引用「12」时要说它是 1.0.0 发布口径；引用当前默认时要说 `alltools`。

BM25 工具的 `k` 默认 10（`KB_search_bm25` / `KB_search_dense` 的结果条数）。这是工具参数默认，不是任务数。

### sandbox-runtime

`terminal_use`、`terminal_use_write`、`alltools`、`alltools-qwen` 需要：

```bash
npm install -g @anthropic-ai/sandbox-runtime@0.0.23
```

**0.0.23** 是文档钉死的 npm 版本。不是「任意新版 sandbox-runtime」。

还要系统二进制。只装 npm 包不够。Linux：`ripgrep`、`bubblewrap`、`socat`（`which srt rg bwrap socat`）。macOS：`ripgrep`（`which srt rg`）。从 tau2 1.0.x 起，缺任何一个，`SandboxManager` 在构造时抛 `SandboxRuntimeError`，而不是把「沙箱不可用」当成一条普通工具结果交回给 agent。

嵌入缓存在 `data/.embeddings_cache`。文档内容变了会失效。目录被 gitignore，不在官方题目里。

## 网络出站

文字模式经 LiteLLM。具体主机取决于你写的模型字符串（`openai/...`、`anthropic/...` 等）。封存文档没有列出 LiteLLM 的全部主机名。

语音和检索里写明的端点：

| 用途 | 地址或区域 |
|------|------------|
| OpenAI Realtime | `wss://api.openai.com/v1/realtime` |
| xAI Realtime | `wss://api.x.ai/v1/realtime`（模型放在 query 参数 `model`） |
| Qwen Realtime | `wss://dashscope-intl.aliyuncs.com/api-ws/v1/realtime` |
| Gemini | AI Studio 或 Vertex AI `us-central1`。主机名未在摘录中写出 |
| Nova Sonic | AWS 区域 `us-east-1` |
| Pine | 使用者提供的 `PINE_REALTIME_BASE_URL` |
| ElevenLabs | TTS 与 Voice Design。主机名未在摘录中写出 |
| Deepgram | 转写。主机名未在摘录中写出 |
| OpenRouter | `https://openrouter.ai/`，给 Qwen 嵌入 |
| Redis（可选缓存） | `localhost:6379`，前缀 `tau2`，TTL `60 * 60 * 24 * 30` 秒（30 天），默认关闭 |
| 排行榜网站 | 当前文档写 [taubench.com](https://taubench.com)。changelog 说网站从 S3 桶 `sierra-tau-bench-public` 取提交和轨迹，不再直接从 GitHub Pages 提供 |
| 域文档查看器 | `127.0.0.1:8004`，本机 |
| API 常量 | `API_PORT = 8000`，见 02 的端口说明 |

适配器计时常量（注释写 fixed）：VoIP 包间隔 20 ms，连接超时 30 秒，断开超时 5 秒，线程 join 2 秒，tick 超时缓冲 30 秒，无活动 40 秒判停滞。

LLM 调用重试常量：最多 3 次，等待 1.0 到 10.0 秒，乘数 1.0。这是 `config.py` 的基础设施常量。CLI 的 `--retry-delay` 默认 1.0 秒，对应的是失败任务重试，不是这一组。

## 开发依赖，和跑分无关

`make test` 只跑核心测试，跳过 voice、streaming、gym、`banking_knowledge`，需要 `--extra dev`。`make test-voice`、`make test-knowledge`、`make test-gym`、`make test-all` 各自要对应 extra。全双工集成测试需要现场的 OpenAI Realtime 和 TTS，可用 `pytest -m "not full_duplex_integration"` 排除。单个供应商测试由 `{PROVIDER}_TEST_ENABLED=1` 打开。

Ruff 行宽 88。这些不影响评测数字。

## 本页依据

`pyproject.toml`，`.env.example`，`src/tau2/config.py`，`AGENTS.md`，`docs/getting-started.md`，`docs/cli-reference.md`，`src/tau2/knowledge/README.md`，`src/tau2/voice/README.md`，`CHANGELOG.md`，Release `v1.0.0`。见 [99-evidence.md](99-evidence.md)。
