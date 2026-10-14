# DistilBERT, a distilled version of BERT: smaller, faster, cheaper and lighter

## Metadata

- Authors / Year / Venue: Sanh et al. · 2019 · arXiv:1910.01108
- Links: https://arxiv.org/abs/1910.01108
- Tags: model-optimization
- Reference: [paper-distilbert-2019](../../../references/papers.md#paper-distilbert-2019)
- Last reviewed: 2026-10-07

## TL;DR

经典知识蒸馏：教师 BERT → 更小 DistilBERT。教学锚点说明蒸馏动机；**不是**现代 Agent 轨迹蒸馏本身。

## Problem setup

部署需要更小更快的编码器模型。

## Method / Mechanism

用教师软标签/隐藏状态训练学生，减少层数与参数。

## How it changes the Agent Loop

- 对应优化层「能力换资源」。
- Agent 场景更常见：小模型做分类/路由/草稿，大模型做主 Decide。

## Experiments that matter

- 经典结果量级（见原文）：约一半参数、显著更快推理，GLUE 等上保留教师 BERT 的大部分表现。
- 蒸馏目标与任务分布对齐时收益稳定；换任务后学生易崩——迁移需重蒸馏/微调。
- 对 Agent：这是编码器蒸馏锚点，不证明「轨迹蒸馏」有效；工具序列需另设实验。

## Engineering takeaways

1. 蒸馏目标任务要与线上一致。
2. Agent 轨迹蒸馏需另设（工具调用序列），勿直接套 BERT 叙事。

## Limitations & failure modes

- 任务迁移后学生崩。
- 与生成式 Agent 主模型不同代。

## When you should / shouldn't use it

| 应该 | 不应该 |
|------|--------|
| 编码器侧加速、教学对比 | 当作 Agent RL/轨迹学习说明书 |

## Related handbook modules

- [model-optimization](../../../runtime/model-optimization/model-optimization.md)
- [fine-tuning](../../../runtime/fine-tuning/fine-tuning.md)

## References

- [paper-distilbert-2019](../../../references/papers.md#paper-distilbert-2019)
