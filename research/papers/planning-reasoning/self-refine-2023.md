# Self-Refine: Iterative Refinement with Self-Feedback

## Metadata

- Authors / Year / Venue: Madaan et al. · 2023 · arXiv:2303.17651
- Links: https://arxiv.org/abs/2303.17651
- Tags: reasoning-planning
- Last reviewed: 2026-10-07
- Reference: [paper-self-refine-2023](../../../references/papers.md#paper-self-refine-2023)

## TL;DR

同一模型完成 **生成 → 自我反馈 → 按反馈精炼**，迭代改进输出。对应 reflection/refinement 控制流；**必须硬终止**，且无外部信号时自评可能自我合理化。

## Problem setup

一次生成质量不稳；人类迭代改稿有效，但希望在不训练、少依赖外部标注的情况下让模型自我改进。

## Method / Mechanism

```text
y0 = Generate(x)
for t in 1..T:
  fb_t = Feedback(x, y_{t-1})   # 同模型（或同家族）语言反馈
  y_t  = Refine(x, y_{t-1}, fb_t)
停止：达到 T、反馈称「足够好」、或外部停止条件
```

关键是把「批评」与「改写」拆成可观测步骤，而不是一次更长的生成。

## How it changes the Agent Loop

| 环节 | 变化 |
|------|------|
| Decide / 产出 | 多轮 refine 子循环套在最终 Act 前或后 |
| Observe | 反馈可来自自评；更强设定可换测试/工具结果 |
| Terminate | max_iters / 质量门槛 / 无改进早停 |

与 Reflexion：Self-Refine 偏 **同任务输出精炼**；Reflexion 偏 **跨 trial 语言记忆**。可组合但勿混为一谈。

## Experiments that matter

- 多任务上迭代精炼相对单次生成有改进（见原文任务集）；收益随轮次递减是工程常态预期。
- 无外部 oracle 时，后期轮次可能空转或风格改写而非实质纠错。

## Engineering takeaways

1. 硬停止：`max_refine`、无改进退出、成本预算——默认写进控制面。
2. 能接外部反馈（单测、rubric、检索冲突）时优先，勿只靠自评。
3. 每轮只保留差分或摘要进 Context，避免全文叠加爆炸。
4. 安全/合规输出：精炼轮仍需同一策略过滤器，防止「改着改着越界」。

## Limitations & failure modes

- 错误反馈强化错误改稿。
- 无限反思 / 润色循环耗尽预算。
- 对事实错误，无检索/工具时 refine 难纠正。

## When you should / shouldn't use it

| 应该 | 不应该 |
|------|--------|
| 文案、代码草稿、可本地验证的改写 | 默认每条用户消息都开多轮 |
| 与 Verifier/测试门禁组合 | 无停止条件的「再想想」 |

## Related handbook modules

- [reasoning-planning](../../../core/reasoning-planning/reasoning-planning.md)
- [agent-loop](../../../core/agent-loop/agent-loop.md)
- [evaluation](../../../production/evaluation/evaluation.md)

## References

- [paper-self-refine-2023](../../../references/papers.md#paper-self-refine-2023)
- [paper-reflexion-2023](../../../references/papers.md#paper-reflexion-2023)
