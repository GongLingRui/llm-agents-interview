# Agent Loop

> 职责：Agent 控制回路 — 状态、步骤、终止、错误与重试。这是本仓库最核心的枢纽模块之一。

## Canonical 文档

**主文（Source of Truth）**：[agent-loop.md](./agent-loop.md)

Phase 1 计划拆成的多篇主题已合并进上述 Canonical 文档。Planning / Memory / Context 的深文仍在各自模块；本主文只定义 Loop 钩子与控制流变体统一视角。

## 受众

要实现 Agent Runtime / Executor 的工程师。

## 前置依赖

硬前置：[Tool Use Canonical](../tool-use/tool-use.md)

## 非目标

- 规划算法细节 → [`reasoning-planning`](../reasoning-planning/README.md)
- 组件级架构长文 → [agent-architecture.md](../agent-architecture/agent-architecture.md)
- 评测体系全文 → [`evaluation`](../../production/evaluation/README.md)

## 边界

- Planning 策略细节 → `core/reasoning-planning`
- 组件级架构图深文 → `core/agent-architecture`
- 评测 Loop 质量 → `production/evaluation`
- ReAct / Planner-Executor 等视为 **Loop 控制流变体**，不是无关概念

## 上游 / 下游

- 上游：`core/tool-use`
- 下游：几乎所有 Agent Types、Production、Practice 模块
