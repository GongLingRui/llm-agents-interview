# Training language models to follow instructions with human feedback (InstructGPT)

## Metadata

- Authors / Year / Venue: Ouyang et al. (OpenAI) · 2022 · arXiv:2203.02155
- Links: https://arxiv.org/abs/2203.02155
- Tags: reinforcement-learning, fine-tuning
- Reference: [paper-instructgpt-2022](../../../references/papers.md#paper-instructgpt-2022)
- Last reviewed: 2026-10-07

## TL;DR

经典 RLHF 流水线：SFT → 人类偏好训练奖励模型 → PPO 优化策略（含 KL 约束）。奠定指令对齐工程范式；是模型训练，不是推理时 Reflexion。

## Problem setup

预训练模型不擅长遵循用户意图；需要对齐数据与稳定优化。

## Method / Mechanism

```text
演示数据 SFT → 偏好排序训 RM → PPO(π) 相对 π_ref + KL
```

## How it changes the Agent Loop

- **不在 Loop 内运行**；改变的是基础策略分布。
- Agent 仍需 Tool/Eval；对齐模型 ≠ 可靠 Agent。
- 与 Verbal RL（Reflexion）严格区分。

## Experiments that matter

- 经典人类偏好结果：较小的 InstructGPT 在遵循指令上可优于更大的未对齐 GPT-3（见原文；勿当永恒 SOTA）。
- 流水线消融意义：SFT 打底、RM 学偏好、PPO + KL 约束优化——缺一环会改变稳定性与对齐质量。
- 工程信号：代理奖励会黑客化；发布仍需人类金标与任务 Eval，不能只看训练奖励上升。

## Engineering takeaways

1. Eval 先于 RL：无稳定任务集不要调权重。
2. 代理奖励会黑客化；保留人类金标门禁。
3. 多数产品问题先停在 Prompt/Tool/SFT。

## Limitations & failure modes

- 奖励黑客、分布外退化、标注成本。
- PPO 工程复杂。

## When you should / shouldn't use it

| 应该 | 不应该 |
|------|--------|
| 有偏好数据与训练预算的对齐 | 用「上 RLHF」回避工具/评测缺口 |

## Related handbook modules

- [reinforcement-learning](../../../runtime/reinforcement-learning/reinforcement-learning.md)
- [fine-tuning](../../../runtime/fine-tuning/fine-tuning.md)
- [evaluation](../../../production/evaluation/evaluation.md)

## References

- [paper-instructgpt-2022](../../../references/papers.md#paper-instructgpt-2022)
