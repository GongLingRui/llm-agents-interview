# FrugalGPT: How to Use Large Language Models While Reducing Cost and Improving Performance

## Metadata

- Authors / Year / Venue: Chen, Zaharia, Zou · 2023 · arXiv:2305.05176
- Links: https://arxiv.org/abs/2305.05176
- Tags: model-optimization
- Reference: [paper-frugalgpt-2023](../../../references/papers.md#paper-frugalgpt-2023)
- Last reviewed: 2026-10-07

## TL;DR

通过提示适配、LLM **级联/路由** 与模型组合，在降本同时尽量保持任务表现；对应 Agent 里的模型路由策略早期系统化讨论。

## Problem setup

不同查询难度差异大，一律用最贵模型浪费。

## Method / Mechanism

按策略把查询送入不同模型或级联：便宜模型先试，必要时升级。

## How it changes the Agent Loop

- 在 Loop 的 **model_call** 前增加路由器。
- 不改变工具语义；影响成本/延迟/质量权衡。

## Experiments that matter

- 查询难度异构时，级联/路由相对「一律最贵模型」可显著降本，并在若干任务上接近或匹配强模型表现（见原文成本-质量曲线）。
- 提示适配与模型组合是补充旋钮；主效应来自「便宜模型先试、必要时升级」。
- 工程取用：必须用尾部难例切片验收——平均准确率掩盖路由静默失败。

## Engineering takeaways

1. 路由特征：任务类型、风险级别、历史失败。
2. 高危动作强制强模型 + HITL，不省。
3. 用 Eval 切片证明级联不伤尾部查询。

## Limitations & failure modes

- 路由错误导致静默质量崩。
- 级联增加尾延迟。

## When you should / shouldn't use it

| 应该 | 不应该 |
|------|--------|
| 成本敏感、查询难度异构 | 统一强模型更便宜的低流量场景 |

## Related handbook modules

- [model-optimization](../../../runtime/model-optimization/model-optimization.md)
- [llm-inference](../../../runtime/llm-inference/llm-inference.md)

## References

- [paper-frugalgpt-2023](../../../references/papers.md#paper-frugalgpt-2023)
