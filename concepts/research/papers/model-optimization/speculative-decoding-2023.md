# Fast Inference from Transformers via Speculative Decoding

## Metadata

- Authors / Year / Venue: Leviathan, Kalman, Matias · 2023 · arXiv:2211.17192（ICML 2023）
- Links: https://arxiv.org/abs/2211.17192
- Tags: model-optimization, llm-inference
- Last reviewed: 2026-10-07
- Reference: [paper-speculative-decoding-2023](../../../references/papers.md#paper-speculative-decoding-2023)

## TL;DR

用廉价 **草稿模型** 快速提议多 token，再由 **目标模型** 并行验证；理想情况下输出分布与单独跑目标模型一致，墙钟加速取决于 **接受率 α**。机制属模型优化；挂载调度在 Inference——**不是** Agent Loop。

## Problem setup

自回归解码逐步串行，延迟随生成长度涨；需要在 **不牺牲目标模型采样分布** 的前提下减少墙钟时间。

## Method / Mechanism

```text
draft 自回归生成 γ 个候选 token
target 对这 γ 步做（可并行的）验证 / 接受-拒绝
接受前缀写入输出；拒绝点由 target 纠正，再继续
```

- **Lossless 设定**：接受规则使最终分布 = 目标模型分布。
- 加速直觉：一次 target 前向「消化」多个 draft token；α 高才划算。
- 与「直接换小模型」不同：目标是 **保大模型质量、换时间**。

## How it changes the Agent Loop

| 层 | 作用 |
|----|------|
| Agent Loop | **无契约变更**；仍是同一次「目标质量」的 LLM 调用 |
| Model Optimization | 是否启用、draft–target 配对、质量一致性要求 |
| LLM Inference | draft/target 与 continuous batching 的共存、观测 TTFT/TPOT |

## Experiments that matter

- 论文在翻译等序列任务上展示相对朴素自回归的加速，并分析接受率与草稿质量关系（**α 与加速比必须在你的流量上重测**）。
- Agent 工程：开放文本 α 高、工具/JSON 步 α 可能低——要分桶。

## Engineering takeaways

1. 上线前监控 **分桶 α**、端到端延迟、额外算力；α 低则关投机或换草稿。
2. 绑定 **draft–target 版本对**，纳入模型指纹与回归矩阵。
3. 工具调用密集步可单独禁用投机，避免「平均 α 好看、关键步变慢」。
4. 机制选型归 [model-optimization](../../../runtime/model-optimization/model-optimization.md)；服务实现归 [llm-inference](../../../runtime/llm-inference/llm-inference.md)。

## Limitations & failure modes

- α 低 → 延迟不降反升。
- 近似/有损接受 → 分布偏移，Agent 工具错误可能被放大。
- 双模型运维复杂度上升。

## When you should / shouldn't use it

| 应该 | 不应该 |
|------|--------|
| 必须保留大模型质量，但 TPOT/交互延迟痛 | 可接受质量降时——优先路由/小模型可能更简单 |
| 已有目标模型服务，能挂 draft | 无 α 观测就全局默认开启 |

## Related handbook modules

- [model-optimization](../../../runtime/model-optimization/model-optimization.md)（机制主）
- [llm-inference](../../../runtime/llm-inference/llm-inference.md)（调度落地）

## References

- [paper-speculative-decoding-2023](../../../references/papers.md#paper-speculative-decoding-2023)
- [paper-frugalgpt-2023](../../../references/papers.md#paper-frugalgpt-2023)
