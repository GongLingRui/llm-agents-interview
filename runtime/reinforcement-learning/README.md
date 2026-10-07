# Reinforcement Learning（for Agents / LLMs）

> 职责：用反馈信号优化模型或策略 — RLHF/RLAIF、可验证奖励、Agent 轨迹级 RL。

## Canonical 文档

**主文（Source of Truth）**：[reinforcement-learning.md](./reinforcement-learning.md)

核心立场：

- **Eval 先于 RL**；没有稳定评测就不要优化权重
- **可验证奖励优先于模糊 RM**（在任务允许时）
- **多数 Agent 问题应先停在 Prompt / Tool / SFT 阶梯**；RL 非默认下一站
- 推理时「言语强化」（Reflexion 等）≠ 训练式 RL

## 受众

做对齐训练、可验证奖励、轨迹级策略优化，或判断「该不该上 RL」的工程师。

## 前置依赖

硬前置：[`foundations/llm`](../../foundations/llm/README.md)  
强烈建议：[`production/evaluation`](../../production/evaluation/README.md)  
软前置：[`fine-tuning`](../fine-tuning/README.md)、[Agent Loop](../../core/agent-loop/agent-loop.md)

## 主题覆盖（已并入 Canonical）

RL 地图、奖励设计、轨迹级学习、可验证奖励、与 Eval 边界及风险见 [reinforcement-learning.md](./reinforcement-learning.md)。训练栈实现细则可另拆，评测方法论仍归 [evaluation](../../production/evaluation/evaluation.md)。

## 非目标

- Agent 任务评测设计 / 数据集长文 → [`production/evaluation`](../../production/evaluation/README.md)
- SFT / LoRA / 无 RL 数据飞轮 → [`fine-tuning`](../fine-tuning/README.md)
- 规划/反思控制流（不改权重）→ [Reasoning](../../core/reasoning-planning/reasoning-planning.md)
- 经典控制 RL 教科书 → 不写

## 上游 / 下游

- 上游：LLM 基础、Evaluation、Fine-tuning、Agent Loop / Tool Use
- 下游：对齐后的策略发布门禁、Coding Agent 等可验证任务、研究笔记 `research/papers/rl-for-agents/`

## 边界

| 易混 | 主责 |
|------|------|
| 评测与回归门禁 | `production/evaluation` |
| 模仿学习 / SFT | `runtime/fine-tuning` |
| 推理时重试与反思 | `core/reasoning-planning` / Loop |
| 危险执行隔离 | `runtime/sandbox` + `production/security` |
