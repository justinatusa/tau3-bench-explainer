# 先读这一页

τ³-bench 是 Sierra 用来评测客服对话 agent 的仿真程序。它不训练模型。它准备一套假业务（政策、工具、数据库、题目），让被测 agent 和一个由模型扮演的用户把事情办完，再检查两件事：业务数据库最后的状态对不对，以及该告诉用户的信息有没有说出来。

从 v1.0.0（2026-03-18）起，这套程序加上了三块：全双工语音、银行知识库检索，以及对旧题的一批修订。产品名改称 τ³-bench。代码仓库没有改名。

## 三个名字

| 字符串 | 是什么 | 不是什么 |
|--------|--------|----------|
| **τ³-bench**（读作 tau three） | 产品名。仓库 README 写明：We just say "tau three" | 一个新的 Git 仓库，或一个叫 `tau3` 的 Python 包 |
| **tau2-bench** | GitHub 仓库名，`sierra-research/tau2-bench`。从 τ² 时期沿用 | 「τ³ 发布之后被废弃的旧仓库」。τ³ 的代码就在这个仓库的 v1.0.0 及之后 |
| **tau2** | `pyproject.toml` 里的包名 `name = "tau2"`，版本字符串在封存摘录中为 `1.0.1`。命令行入口是 `tau2`，`import tau2` | PyPI 上另一个产品名。封存摘录里没有 `tau3` 这个分发包名 |

仓库 README 的大标题仍写作 τ-Bench，arXiv 徽章指向 τ² 论文 `2506.07982`。正文横幅才写「τ³-bench is here」。读仓库时以横幅、`CHANGELOG.md` 的 1.0.0 / 1.0.1 和本说明书的钉为准，不要把标题徽章当成「当前产品仍是 τ²」。

## 一页纸全景

五个域，都在仓库里注册。题目文件在 `data/tau2/domains/<域名>/`。下表把「文档里写过的规模」和「当前 `tasks.json` 有没有数过」分开。修订条数不是题库大小。

| 域 | 在测的事 | 题数怎么读 |
|----|----------|------------|
| `mock` | 框架自带的小域，用来做安装后的冒烟 | 数据摘要：`tasks.json` **10**，语音配置 **9**。9 不是「少了一道文字题」的解释 |
| `airline` | 航空客服。v1.0.0 点名修订了其中 27 道题 | **当前** `tasks.json` 条数 UNKNOWN。修订博客把**原先**规模写成 50。27 是修订条数 |
| `retail` | 零售客服。v1.0.0 点名修订了其中 26 道题 | **当前** `tasks.json` 条数 UNKNOWN。修订博客把**原先**规模写成 114。26 是修订条数 |
| `telecom` | 电信客服。用户侧也可以有工具，并带消融模式 | 数据摘要：`tasks.json` **2285**。默认划分名 `base` 是 **114**。榜上用哪一份：UNKNOWN |
| `banking_knowledge` | v1.0.0 新增。客服必须从文档里找出规定，再调用交易工具 | **97** 道题，另有 **698** 篇文档 |

两种通信方式，由不同的 orchestrator 驱动：

| 模式 | 日常说法 | 代码里的开关 |
|------|----------|----------------|
| 半双工文字 | 一轮一轮聊天，可以调用工具 | 默认。`tau2 run --domain ...` 不加语音旗标 |
| 全双工语音 | 用户和 agent 可以同时出声 | `--audio-native` |

一次跑完的分数，默认不是「有没有按标准答案的工具顺序做」。airline、retail、telecom 的默认 `reward_basis` 是 `["DB", "COMMUNICATE"]`：数据库终态要和标准终态一致，并且该说的字符串要出现在 agent 的话里。最终奖励是这几项的乘积。`evaluation_criteria.actions` 是一条参考轨迹，用来在一份干净环境里重放、算出目标数据库；agent 可以走别的工具顺序，只要终态相同。

`banking_knowledge` 在 v1.0.1 改了计分方式。这个域上，tau2-bench `< 1.0.1` 的分数不能和 `>= 1.0.1` 的分数放在一起比。airline、retail、telecom、mock 不受这次改动影响。

题目数据的权威位置是仓库内 `data/tau2/domains/<域名>/`。`data/simulations/` 是运行结果。不要把 Hugging Face `HuggingFaceH4/tau2-bench-data` 写成 τ³ 官方 Dataset。那个仓库是 τ² 时期的镜像。ModelScope `evalscope/tau3-bench-data` 是 EvalScope 的打包，也不是主源。

## 先记住的钉

| 对象 | 值 |
|------|-----|
| 代码摘录 | `main` = `b7ea9074c1cba482b30687fecdb5c8425fd6f619`（2026-09-17） |
| τ³ 首个产品 tag | `v1.0.0` = `17e07b1da2bbc0cadfddeea36412686e0604127b`（2026-03-18） |
| 评分修复 tag | `v1.0.1` = `fc0055dc4e0a316c3f83133267fbd6faaa770992` |
| 要复现修复前的 `banking_knowledge` 计分 | tag `pre-v1.0.1`，提交 `b51a6d69e26f0e94a9173e2e80fe8735a8dff650`（文档里常简写 `b51a6d6`） |
| Python | `>=3.12, <3.14`。`.python-version` 钉的是 3.12 |
| 安装 | `uv sync`。从 τ² 升级时，不再以 `pip install -e .` 为默认 |

**v1.0.1 的日期在封存材料里有三处写法。** 是同一 tag 的不同时间戳，不是三个版本：PIN 把该提交钉在 2026-07-16；GitHub Release 正文的 `published` 是 `2026-07-22T21:22:42Z`；`CHANGELOG.md` 标题是 `2026-07-15`。修复内容先在 2026-07-14/15 进入 `main`，当时没有改版本号。

**`1.0.1` 这个版本字符串** 是封存摘录里 `pyproject.toml` 的 `version`。不是「main 等于 v1.0.1 tag」。main 比 tag 新。

## 这套说明书怎么读

按编号往下。00 只建立地图。01 讲测什么。02 讲模块。03 讲一次仿真的时间顺序。04 讲依赖和网络。05 讲容易叫错的概念。06 只收录官方文档里出现过的命令。99 是证据索引。

τ 和 τ² 只在 [01-what-is-tau3.md](01-what-is-tau3.md) 的对照节出现，用来说明名字从哪来。本说明书的主体是 v1.0.0 及之后的 τ³。

第二份封存包补上了 `cli.py`、`registry.py`、evaluator 入口、编排器与 agent 的 README，以及五篇论文的 arXiv 摘要（含 τ-Knowledge `2603.04370`、τ-Voice `2603.13686`）。编排器的 `run()` 实现仍然不在包里。摘要没有给出的公式，下文仍标 UNKNOWN。telecom 排行榜默认划分是封存索引自己标成 UNKNOWN 的一项。

## 本页依据

`README.md`、`pyproject.toml`、`CHANGELOG.md`、`RELEASE_NOTES.md`、`AGENTS.md`，以及 tag `v1.0.0` / `v1.0.1` 的 Release 正文。路径表见 [99-evidence.md](99-evidence.md)。
