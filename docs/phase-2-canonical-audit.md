# Phase 2 Canonical Audit

> **Last reviewed**: 2026-10  
> **目的**：全仓完成度快照 — Canonical / Practice / Research / 治理 / Evergreen。

## 已完成（可视为仓库主体）

| 层 | 状态 |
|----|------|
| Foundations / Core / Protocols / Agent Types / Runtime / Production | 各模块 Canonical ≥650 行 + 45 面试题 |
| Practice | P0–P8 里程碑 · Interview 索引 · System Design |
| Research Papers | 已登记 27 篇均有机制笔记 |
| Industry | Anthropic×4 · Cursor×1 · OpenAI×1 |
| OSS | vLLM · MCP · OpenAI Agents SDK |
| Cases | Multi-Agent Research · Coding SWE-bench 路径 · Cursor Context Harness |
| Talks | Karpathy×2 已登记 |
| 导航治理 | 层 README / reading-paths / 根 README 阶段说明已对齐 |
| NICE_TO_HAVE polish（2026-10） | 根进度 Case×3·Talk×2；学习/面试路径模块链；Talks 正文回链；OSS 三笔记 Metadata；CONTRIBUTING→MIT |

## Critical Issues

**无。** 边界规则仍有效：MCP≠Tool Use；Sandbox≠Security；Eval≠RL；Prompt≠Context；默认单 Agent。

## 有意未「写完」的项（非缺口）

| 项 | 说明 |
|----|------|
| Practice 项目「练习时填写」 | 学习者作业位，不是文档欠债 |
| 大量 `TODO: Verify` | Evergreen 周期复核（厂商 API/价目/产品 UX），非缺机制正文 |
| ~~License 待定~~ | ✅ 已选 **MIT**（见根 `LICENSE`） |
| 无限 Case / Talk / 论文 | 按有 URL 来源增量，禁止为完整而虚构 |

## High（Evergreen）

| 项 | 说明 |
|----|------|
| OpenAI/Anthropic 工具文档 | ✅ [evergreen-2026-10-tool-docs.md](./evergreen-2026-10-tool-docs.md) |
| Structured Outputs / caching / seed | ✅ [evergreen-2026-10-llm-api.md](./evergreen-2026-10-llm-api.md) |
| Caching TTL / 最低 token · Gemini seed | ✅ [evergreen-2026-10-caching-seed.md](./evergreen-2026-10-caching-seed.md)（机制已核；计费倍率仍 Verify） |
| Streaming tool args / strict · refusal · Managed HITL · compaction 机制 | ✅ [evergreen-2026-10-agents-stream.md](./evergreen-2026-10-agents-stream.md) |
| 厂商价目数字 / 完整模型矩阵 / Managed 产品默认 UX | 仍周期复核 |
| MCP 扩展（Tasks / sampling / Apps）与 Host UX | ✅ 机制层 [evergreen-2026-10-mcp.md](./evergreen-2026-10-mcp.md)；Host UX / SDK 落地矩阵仍待 Verify |
| OTel GenAI conventions | ✅ [evergreen-2026-10-otel.md](./evergreen-2026-10-otel.md) |
| 位置偏差 / context rot 内部评测 | 需自有模型刷新曲线 |

## Decision for Next

继续 **Evergreen（价目数字 / 模型矩阵 / 产品 UX）** 与 **有来源增量**；不平行新建机制目录。
