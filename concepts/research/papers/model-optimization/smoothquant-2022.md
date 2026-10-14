# SmoothQuant: Accurate and Efficient Post-Training Quantization for Large Language Models

## Metadata

- Authors / Year / Venue: Xiao, Lin, Seznec, Wu, Demouth, Han · 2022 · arXiv:2211.10438（后刊 ICML 等）
- Links: https://arxiv.org/abs/2211.10438
- Tags: model-optimization
- Last reviewed: 2026-10-07
- Reference: [paper-smoothquant-2022](../../../references/papers.md#paper-smoothquant-2022)

## TL;DR

通过 **按通道平滑缩放**，把激活上的量化难度迁移到权重侧，使 **W8A8（权重+激活）** 等低精度矩阵乘更可行。处理的是激活离群值问题；属推理压缩，**不是** Agent 策略层。

## Problem setup

LLM 激活常有 **离群大值**：激活量化难、权重量化相对易。只做权重量化省显存，但要进一步压算力时常卡在激活 INT8。

## Method / Mechanism

```text
对通道 j：X̂_j = X_j / s_j ， Ŵ_j = W_j · s_j
选择 s，使「难量化的激活」变平滑、「易量化的权重」略变难
→ 再对平滑后的激活与权重做量化（典型目标：W8A8）
```

- 数学上等价格式变换 + 两侧量化；关键是 **迁移难度** 而非改网络结构。
- 与 GPTQ/AWQ（多为权重量化为主）互补：SmoothQuant 面向 **激活也可低比特** 的路径。

## How it changes the Agent Loop

- Loop **不变**；改变的是 GEMM 数值路径与硬件利用率。
- Agent 侧风险：激活量化对 **异常 token / 长上下文 / 结构化约束** 更敏感 → 需轨迹级回归。
- 映射：[model-optimization](../../../runtime/model-optimization/model-optimization.md)；算子与批调度属 [llm-inference](../../../runtime/llm-inference/llm-inference.md)。

## Experiments that matter

- 论文展示在多种 LLM 上，平滑后 W8A8 相对「直接量化激活」更保精度，并讨论推理效率（**勿抄数字当容量规划**）。
- 工程：确认你的引擎是否真正跑 W8A8 路径，而非「权重 INT8、激活仍 FP」。

## Engineering takeaways

1. 先分清目标：只压显存（权重）vs 还要压算力（权重+激活）。
2. 激活离群是 W8A8 成败关键；平滑是机制选项之一。
3. Agent 验收仍盯工具合规与多步成功，尤其长上下文步。
4. 与 Inference 联调：内核支持、精度配置进模型指纹。

## Limitations & failure modes

- 平滑超参/通道策略不当 → 两边都难量化。
- 并非所有硬件/框架对 W8A8 同等友好。
- 不能替代路由、减步、减上下文等任务级优化。

## When you should / shouldn't use it

| 应该 | 不应该 |
|------|--------|
| 需要激活也量化以换吞吐，且引擎支持 | 尚未有 BF16/FP16 质量基线就上 W8A8 |
| 与权重量化方案对照选型 | 当作 Agent Loop 或 Memory 方案 |

## Related handbook modules

- [model-optimization](../../../runtime/model-optimization/model-optimization.md)（主）
- [llm-inference](../../../runtime/llm-inference/llm-inference.md)

## References

- [paper-smoothquant-2022](../../../references/papers.md#paper-smoothquant-2022)
- [paper-gptq-2022](../../../references/papers.md#paper-gptq-2022)
- [paper-awq-2023](../../../references/papers.md#paper-awq-2023)
