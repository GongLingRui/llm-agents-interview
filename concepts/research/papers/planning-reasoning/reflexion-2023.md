# Reflexion: Language Agents with Verbal Reinforcement Learning

## Metadata

- Authors / Year / Venue: Shinn et al. · 2023 · arXiv:2303.11366
- Links: https://arxiv.org/abs/2303.11366
- Tags: reasoning-planning, agent-loop, memory
- Last reviewed: 2026-10-07
- Reference: [paper-reflexion-2023](../../../references/papers.md#paper-reflexion-2023)

## TL;DR

用 **语言化反思** + 情节记忆缓冲改进后续 trial，**不更新模型权重**。工程上对应 reflection 钩子与短暂 episodic memory；名称含 RL，但不是训练式 RL。

## Problem setup

多 trial 任务上，Agent 重复犯同类错；需要一种低成本「从失败中学」而不启动梯度更新。

## Method / Mechanism

```text
Trial → 环境反馈 / 评分
     → 语言反思（失败原因、下次策略）
     → 写入记忆缓冲
下一 Trial 上下文注入反思摘要
```

「Verbal reinforcement」= 用自然语言强化信号塑造后续行为分布。

## How it changes the Agent Loop

| 环节 | 变化 |
|------|------|
| Observe | 除工具结果外，增加 trial 级评价信号 |
| Decide | 下一 trial 受反思记忆条件化 |
| Memory | 短中期 episodic，跨 trial 而非永久用户画像 |

## Experiments that matter

- 编程/决策类环境中，多 trial + 反思优于无记忆重试（见原文；生产需自建失败分类）。
- 关键工程点：反思质量依赖反馈信号清晰度。

## Engineering takeaways

1. 先有可判定失败信号（测试、rubric、工具错误码），再上反思。
2. 反思写入必须过 **写闸**：去密钥、去大段 dump、可遗忘。
3. 硬终止：max_trials；防止无限「再反思一次」。
4. 与 [reinforcement-learning](../../../runtime/reinforcement-learning/reinforcement-learning.md) 划界：本文是推理时策略，不是权重优化。

## Limitations & failure modes

- 错误归因的反思会强化错误策略。
- 记忆膨胀污染 Context。
- 无 held-out 时可能过拟合单一任务话术。

## When you should / shouldn't use it

| 应该 | 不应该 |
|------|--------|
| 可重复 trial + 清晰评分 | 单次用户对话的默认路径 |
| 代码修复等可验证域 | 当作正式 RLHF 替代 |

## Related handbook modules

- [reasoning-planning](../../../core/reasoning-planning/reasoning-planning.md)
- [memory](../../../core/memory/memory.md)
- [agent-loop](../../../core/agent-loop/agent-loop.md)
- [evaluation](../../../production/evaluation/evaluation.md)

## References

- [paper-reflexion-2023](../../../references/papers.md#paper-reflexion-2023)
