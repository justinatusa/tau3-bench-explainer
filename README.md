# τ³-bench 说明书

这是一份给零背景读者的中文说明。它解释 Sierra 的评测 **τ³-bench**（口语常说 “tau three”）：在真实客服场景里，让代理一边跟用户对话、一边调工具，必要时还要检索内部文档或走语音全双工。

本仓库只有说明，没有完整题目数据，也不能代替官方运行。官方代码与题目在 [`sierra-research/tau2-bench`](https://github.com/sierra-research/tau2-bench)（产品名是 τ³-bench，仓库名仍叫 `tau2-bench`，Python 包名仍是 `tau2`）。题目主源在该仓的 `data/` 目录，不是 Hugging Face 上某个叫 tau2-bench-data 的第三方镜像。

核对钉死（封条时）：

| 钉 | 值 |
| --- | --- |
| main | `b7ea9074c1cba482b30687fecdb5c8425fd6f619`（2026-09-17） |
| v1.0.0（τ³ 发布） | `17e07b1da2bbc0cadfddeea36412686e0604127b` |
| v1.0.1（banking 评分修复） | `fc0055dc4e0a316c3f83133267fbd6faaa770992` |

官网与榜：https://taubench.com

## 状态

初版骨架已就位；完整说明书正文由云端写稿代理产出后，会质检并覆盖到本仓 `docs/tau3-bench-explainer/`。

## 先读哪一页

| 顺序 | 文件 | 读完你能回答 |
| --- | --- | --- |
| 0 | 本页 | 这套评测在测什么，仓库名为什么叫 tau2 |
| 1 | [`00-readme.md`](docs/tau3-bench-explainer/00-readme.md) | 材料谁说了算 |
| 2 | [`01-what-is-tau3.md`](docs/tau3-bench-explainer/01-what-is-tau3.md) | τ→τ²→τ³、有哪些域 |
| 3 | [`02-architecture.md`](docs/tau3-bench-explainer/02-architecture.md) | 一次仿真怎么拼起来 |
| 4 | [`03-agent-loop.md`](docs/tau3-bench-explainer/03-agent-loop.md) | 半双工与全双工每一步 |
| 5 | [`04-dependencies.md`](docs/tau3-bench-explainer/04-dependencies.md) | 依赖、密钥、外部服务 |
| 6 | [`05-unnamed-concepts.md`](docs/tau3-bench-explainer/05-unnamed-concepts.md) | 难命名的横切概念 |
| 7 | [`06-runbook.md`](docs/tau3-bench-explainer/06-runbook.md) | 官方怎么跑 |
| 8 | [`99-evidence.md`](docs/tau3-bench-explainer/99-evidence.md) | SHA 与出处 |

## 对照不要混淆

- **τ-bench**（2024）与 **τ²-bench**（2025）是前代；本说明书主线是 **τ³**（v1.0.0+）。
- `HuggingFaceH4/tau2-bench-data` 是 τ² 时代镜像，**不是** τ³ 官方 Dataset。
