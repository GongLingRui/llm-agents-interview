# Reasoning & Planning

> 职责：推理与计划如何进入 Agent Loop — 何时显式计划、何时靠隐式推理、如何反思与修正。

## Canonical 文档

**主文（Source of Truth）**：[reasoning-planning.md](./reasoning-planning.md)

覆盖：Reasoning / Planning / Replanning / Reflection / Verification 的定义与边界、何时用/不用、模式比较、预算、失败模式、与 Agent Loop 的关系。Phase 1 多文件计划已合并进主文。

## 受众

设计 Agent 决策策略、控制推理成本与失败恢复的工程师。

## 前置依赖

硬前置：[Agent Loop](../agent-loop/agent-loop.md)、[Tool Use](../tool-use/tool-use.md)

## 非目标

- 控制回路主体 → [`agent-loop`](../agent-loop/agent-loop.md)
- 工具执行管线 → [`tool-use`](../tool-use/tool-use.md)
- 搜索/树搜索训练式 RL → [`reinforcement-learning`](../../runtime/reinforcement-learning/README.md) 与 papers
- Multi-Agent 分工计划 → [`multi-agent`](../../agent-types/multi-agent/README.md)
- 评测体系全文 → [`evaluation`](../../production/evaluation/README.md)

## 边界

- **Planning ≠ Agent Loop**；**Reasoning ≠ CoT 文本堆砌**
- **Reflection ≠ Verification ≠ Evaluation ≠ RL**
- ReAct / Plan-Execute / ToT 等视为策略模式，映射回 Loop + 本文

## 上游 / 下游

- 上游：`core/agent-loop`
- 下游：`core/agent-architecture`、复杂 Coding Agent、`agent-types/multi-agent`
