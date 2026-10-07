# Multi-Agent

> 职责：多 Agent 协作 — 分工、通信、共享状态、冲突、编排模式，以及**何时不该用**。  
> **立场**：默认单 Agent；Multi-Agent 是可选复杂度。

## Canonical 主文

| 文件 | 说明 |
|------|------|
| **[multi-agent.md](./multi-agent.md)** | Source of Truth（Canonical）：定义、判据、分工/通信/状态/冲突、Supervisor·Peer·Pipeline、失败模式、与 Architecture/Loop 边界、45 面试题 |

## 受众

在单 Agent + 工具面/上下文/验证已穷尽后，评估协作拓扑的系统设计与 Runtime 工程师。

## 前置依赖

硬：[Agent Architecture](../../core/agent-architecture/agent-architecture.md)、[Reasoning & Planning](../../core/reasoning-planning/reasoning-planning.md)  
软：[Agent Loop](../../core/agent-loop/agent-loop.md)、[Memory](../../core/memory/memory.md)、[`production/evaluation`](../../production/evaluation/README.md)

## 主题覆盖（已并入 Canonical）

主文已覆盖：何时用/不用、拓扑、通信、状态、冲突、失败模式与评测复杂度。需要案例集或运营剧本时再拆子文并回链 [multi-agent.md](./multi-agent.md)，禁止平行维护第二套机制正文。

## 非目标

- 「多智能体」名词堆砌与框架 API 手册
- 把一次多模型投票 / 单 Loop 角色扮演算 Multi-Agent 深文
- **复写** [Agent Loop](../../core/agent-loop/agent-loop.md) 控制回路

## 边界

| 主题 | 去哪里 |
|------|--------|
| 组件组装、模式目录位 | [Architecture](../../core/agent-architecture/agent-architecture.md) |
| 单身份决策回路 | [Agent Loop](../../core/agent-loop/agent-loop.md) |
| 计划/反思/校验策略 | [Reasoning](../../core/reasoning-planning/reasoning-planning.md) |
| 多身份协作机制 | **本文模块 Canonical** |

## 警告

Multi-Agent 增加可观测与调试成本；默认应先穷尽单 Agent + 好工具面。见主文 §0、§2。

## 上游 / 下游

- 上游：Architecture、Reasoning、Loop、Tool Use  
- 下游：Evaluation、Observability、Security、Design Patterns、垂直 agent-types  
