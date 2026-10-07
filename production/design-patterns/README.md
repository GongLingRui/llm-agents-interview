# Agent Design Patterns

> 职责：在真实约束下可复用的 Agent 结构模式 — 带适用条件与失败模式。

## 受众

做 Agent 系统设计、模式选型、生产落地与面试准备的工程师。

## 前置依赖

硬前置：`core/agent-architecture`、`core/agent-loop`、`core/tool-use`  
软前置：`core/reasoning-planning`、`core/context-engineering`；建议 Eval / Obs / Multi-Agent 接口级了解

## Source of Truth

| 文件 | 状态 | 说明 |
|------|------|------|
| [design-patterns.md](./design-patterns.md) | Canonical | 模式语言、P01–P17 目录、组合/反模式、45 面试题 |

## 主题覆盖（已并入 Canonical）

模式语言、P01–P17、组合与反模式见 [design-patterns.md](./design-patterns.md)。单模式落地清单可另拆 checklist，但机制解释不重复 Core Canonical。

## 非目标

- 不重写 Loop / Tool / Memory / RAG / Context 机制正文 → 各 Core Canonical
- 不以框架 API 或厂商产品功能名当模式理论
- 不替代 Evaluation / Security / Multi-Agent 模块深文

## 上游 / 下游

| 方向 | 模块 |
|------|------|
| 上游 | Architecture、Loop、Tool Use、Reasoning、Context |
| 下游 | Evaluation、Observability、Security、Infrastructure、面试与系统设计练习 |
| 旁路接口 | Multi-Agent、Coding Agent、Memory、RAG（只挂载） |

## 与易混模块的边界

| 模块 | 边界 |
|------|------|
| `core/agent-architecture` | 层与所有权；本文是可复用拓扑清单 |
| `core/agent-loop` | 控制回路机制；本文 Tool-Loop 等只挂载 |
| `core/reasoning-planning` | 计划/校验策略语义；本文定结构落点 |
| `agent-types/multi-agent` | Supervisor–Workers 深文在彼；本文给适用/失败接口 |
| `production/evaluation` | 指标与数据集；本文 Eval Harness 是模式门禁结构 |
