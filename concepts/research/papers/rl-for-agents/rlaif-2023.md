# RLAIF: Scaling Reinforcement Learning from Human Feedback with AI Feedback

## Metadata

- Authors / Year / Venue: Lee et al. · 2023 · arXiv:2309.00267
- Links: https://arxiv.org/abs/2309.00267
- Tags: reinforcement-learning
- Reference: [paper-rlaif-2023](../../../references/papers.md#paper-rlaif-2023)
- Last reviewed: 2026-10-07

## TL;DR

系统研究用 AI 偏好替代/扩展人类偏好做 RL 对齐；表明合适设定下可逼近 RLHF，但必须以人类金标校准，并隔离训练奖励与发布门禁。

## Problem setup

RLHF 人类偏好瓶颈限制规模。

## Method / Mechanism

用 LLM 生成偏好标签 → 训 RM 或直接优化；与人类标签对比。

## How it changes the Agent Loop

- 训练数据飞轮；不改变单次 Agent Loop。
- Eval Canonical：发布门禁 ≠ 训练奖励。

## Experiments that matter

- 合适提示/协议下，AI 偏好标签训练的对齐可逼近 RLHF 人类偏好表现（见原文对比设定）。
- 关键消融：反馈模型选择、标注协议与人类校准——决定「可扩展」还是「偏差放大」。
- 工程信号：训练奖励与发布门禁必须隔离；无人类抽检的全自动闭环不可上线。

## Engineering takeaways

1. AI 反馈可扩展，但不能闭环自我颁奖。
2. 保留人类抽检与对抗集。
3. Agent 轨迹级 RLAIF 更难——过程奖励设计见 RL Canonical。

## Limitations & failure modes

- 反馈模型偏差放大。
- 与真实用户效用脱节。

## When you should / shouldn't use it

| 应该 | 不应该 |
|------|--------|
| 扩展偏好数据 | 无校准地全自动对齐上线 |

## Related handbook modules

- [reinforcement-learning](../../../runtime/reinforcement-learning/reinforcement-learning.md)
- [evaluation](../../../production/evaluation/evaluation.md)

## References

- [paper-rlaif-2023](../../../references/papers.md#paper-rlaif-2023)
