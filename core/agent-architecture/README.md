# Agent Architecture

> 职责：把 Loop、Tools、Memory、Context、Eval 等组装成可讨论的系统架构。

## Canonical 文档

**主文（Source of Truth）**：[agent-architecture.md](./agent-architecture.md)

路径基线：`core/agent-architecture/`（**不是** `core/architecture/`）。Phase 1 多文件计划已合并进主文。

## 受众

做系统设计、Runtime 选型与生产落地的工程师。

## 前置依赖

硬前置：[Tool Use](../tool-use/tool-use.md)、[Agent Loop](../agent-loop/agent-loop.md)、[Reasoning & Planning](../reasoning-planning/reasoning-planning.md)

## 非目标

- 框架 API 教程（LangGraph / Agents SDK 等仅为 implementation mapping）
- Memory / RAG / Context / MCP / Multi-Agent / RL 深文 → 各自模块
- 可复用模式长清单 → [`design-patterns`](../../production/design-patterns/README.md)
- 云部署百科 → [`infrastructure`](../../production/infrastructure/README.md)

## 边界

- **Architecture = 组装与拓扑决策**；Loop/Tools/Reasoning = 机制 Source of Truth
- **Multi-Agent 非默认**；Supervisor 需评测证明
- State / Memory / Context 边界以主文为准

## 上游 / 下游

- 上游：`agent-loop`、`reasoning-planning`、`tool-use`
- 下游：System Design、Production Patterns、Multi-Agent、垂直 Agent Types
