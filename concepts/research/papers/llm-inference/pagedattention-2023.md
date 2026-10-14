# Efficient Memory Management for Large Language Model Serving with PagedAttention

## Metadata

- Authors / Year / Venue: Kwon et al. · 2023 · arXiv:2309.06180
- Links: https://arxiv.org/abs/2309.06180
- Tags: llm-inference
- Reference: [paper-pagedattention-2023](../../../references/papers.md#paper-pagedattention-2023)
- Last reviewed: 2026-10-07

## TL;DR

用分页管理 KV Cache（PagedAttention）降低显存碎片并提高并发；工程含义：**并发上限常由 KV 池决定**。vLLM 等实现的核心思想来源。

## Problem setup

连续批下 KV 预留导致显存碎片与浪费，限制 batch。

## Method / Mechanism

类似 OS 虚拟内存：KV 分块按需分配/回收，注意力按页读取。

## How it changes the Agent Loop

- 仍在推理数据面。
- 长上下文 Agent 的隐成本中心是 KV，不只是「prompt 字符数」。

## Experiments that matter

- 相对连续预留 KV：分页降低碎片与浪费，可支撑更高并发 batch（见原文 / vLLM 报告；硬件与模型相关）。
- 吞吐提升主要来自「同样显存塞进更多序列」，而非单次 attention 魔法加速。
- 工程取用：容量规划按 KV 池占用来做；上线前用真实前缀长度与并发重跑。

## Engineering takeaways

1. 容量规划按 KV 池而非仅 GPU FLOPs。
2. 前缀缓存与分页协同时注意多租户隔离。
3. 见 [vllm OSS 笔记](../../open-source-analysis/vllm/vllm.md)。

## Limitations & failure modes

- 实现复杂度；页大小调参。
- 不解决 Context 内容质量问题。

## When you should / shouldn't use it

| 应该 | 不应该 |
|------|--------|
| 高并发 LLM serving | 用论文数字当业务绝对延迟承诺 |

## Related handbook modules

- [llm-inference](../../../runtime/llm-inference/llm-inference.md)
- [model-optimization](../../../runtime/model-optimization/model-optimization.md)

## References

- [paper-pagedattention-2023](../../../references/papers.md#paper-pagedattention-2023)
- [repo-vllm](../../../references/repos.md#repo-vllm)
