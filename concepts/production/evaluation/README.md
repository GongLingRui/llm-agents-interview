# Agent Evaluation

> 职责：评估 Agent — 任务成功、轨迹质量、成本、安全与回归。

## Canonical 文档

**主文（Source of Truth）**：[evaluation.md](./evaluation.md)

核心立场：

- **测什么**：Success / Quality / Cost / Latency / Safety（五维缺一不可）  
- **Offline 门禁 + Online 确认**；失败 case 必须回流  
- **过程（trajectory）与工具使用**是一等公民，终态对 ≠ 可发布  
- **Agent Eval ≠ 预训练榜 ≠ Observability ≠ RL**

## 受众

负责 Agent 发布质量、回归门禁与实验设计的工程师 / Tech Lead。

## 前置依赖

硬前置：[Agent Loop](../../core/agent-loop/agent-loop.md)、[Tool Use](../../core/tool-use/tool-use.md)  
建议：[Agent Architecture](../../core/agent-architecture/agent-architecture.md)

## 主题覆盖（已并入 Canonical）

测什么、Offline/Online、数据集与 rubric、轨迹与工具评测、回归门禁与反模式均见 [evaluation.md](./evaluation.md)。不再平行维护历史大纲文件。

## 非目标

- 模型预训练基准刷榜 → **禁止作为主体**  
- Traces / 回放 / 调试 playbook 深文 → [`observability`](../observability/README.md)  
- RLHF / Agent RL / 奖励黑客训练 → [`runtime/reinforcement-learning`](../../runtime/reinforcement-learning/README.md)  
- 威胁模型全文 → [`security`](../security/README.md)

## 上游 / 下游

- 上游：Agent Loop、Tool Use、Architecture  
- 下游：Observability（信号）、Security（红队协同）、Infrastructure（门禁接入）、RL（消费评测信号）、Design Patterns / 发布决策

## 边界（易混）

| 易混 | 分界 |
|------|------|
| vs Observability | Eval = 好不好/能否发；Obs = 如何看见与诊断 |
| vs RL | Eval = 度量与门禁；RL = 用反馈改进策略 |
| vs Model Benchmark | 通用能力榜可粗筛，不替代任务/轨迹评测 |
