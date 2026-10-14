# Constitutional AI: Harmlessness from AI Feedback

## Metadata

- Authors / Year / Venue: Bai et al. (Anthropic) · 2022 · arXiv:2212.08073
- Links: https://arxiv.org/abs/2212.08073
- Tags: reinforcement-learning
- Reference: [paper-constitutional-ai-2022](../../../references/papers.md#paper-constitutional-ai-2022)
- Last reviewed: 2026-10-07

## TL;DR

用「宪章」原则驱动 AI 反馈做 harmless 对齐（RLAIF 思想源头之一），减少对人类标注有害偏好的依赖。

## Problem setup

有害偏好人类标注昂贵且心理负担大；需要可扩展的 harmlessness 信号。

## Method / Mechanism

原则集合 → AI 批评/改写 → 偏好或强化学习阶段；评判协议需版本化。

## How it changes the Agent Loop

- 仍是 **训练时对齐**。
- 工程上引入「原则文档版本」与评判偏差风险。
- 不替代运行时 Security 政策。

## Experiments that matter

- 宪章原则驱动的 AI 批评/改写 + 后续偏好/RL 阶段，可扩展 harmlessness，减少对人类标注有害偏好的依赖（见原文人类评测对比）。
- 有用性与无害性存在权衡；原则集合与评判协议版本会改变结果分布。
- 工程信号：AI 反馈需人类金标校准；不能把「过了宪章评判」当唯一发布门禁。

## Engineering takeaways

1. 原则要可测试、可版本；防止静默漂移。
2. 必须用人类金标校准 AI 反馈。
3. 运行时仍要工具鉴权与注入防护。

## Limitations & failure modes

- 宪章漏洞 / 评判模型共谋偏差。
- 与有用性权衡。

## When you should / shouldn't use it

| 应该 | 不应该 |
|------|--------|
| 可扩展 harmlessness 对齐研究/训练 | 当作唯一安全层 |

## Related handbook modules

- [reinforcement-learning](../../../runtime/reinforcement-learning/reinforcement-learning.md)
- [security](../../../production/security/security.md)

## References

- [paper-constitutional-ai-2022](../../../references/papers.md#paper-constitutional-ai-2022)
- [paper-rlaif-2023](../../../references/papers.md#paper-rlaif-2023)
