# Lost in the Middle: How Language Models Use Long Contexts

## Metadata

- Authors / Year / Venue: Liu et al. · 2023 · arXiv:2307.03172
- Links: https://arxiv.org/abs/2307.03172
- Tags: context-engineering
- Last reviewed: 2026-10-07
- Reference: [paper-lost-in-the-middle-2023](../../../references/papers.md#paper-lost-in-the-middle-2023)

## TL;DR

多文档问答等设置下，相关信息放在上下文 **开头或结尾** 时表现更好，放在 **中间** 时显著下降——呈 U 形。直接否定「窗口越大、随便塞就越好」。

## Problem setup

长上下文模型宣称能用满窗口，但实际是否均匀使用全部位置上的证据？

## Method / Mechanism

控制相关段落在多文档输入中的 **位置**，保持任务与干扰文档集合，测量准确率随位置变化 → 观察到 U-shaped 性能曲线。

## How it changes the Agent Loop

- 影响 **Context 装配顺序**：关键约束、当前目标、高信号证据应避开「中间死亡区」或通过压缩减少中间噪声。
- 不改变 Tool/Memory 定义，但改变「检索命中后如何摆放」。

## Experiments that matter

- 多文档 QA：位置敏感性跨多种模型出现（幅度因模型而异）。
- 工程含义：内部评测应刷新「你们主力模型」的位置偏差曲线（标为定期 Verify）。

## Engineering takeaways

1. Assembler 策略：系统硬约束与当前子目标靠前；最新观测可靠后；中间放可丢摘要。
2. 降 k、重排、去重比盲目扩窗更有效。
3. Compaction 后重排仍然重要——摘要也可能被埋在中间。
4. 与 Anthropic「注意力预算 / context rot」叙事一致，本仓库以 Context Engineering 为 Canonical。

## Limitations & failure modes

- 曲线随模型/任务变化；不可把 2023 绝对位置当永恒常数。
- 仅优化位置不能修复检索本身错误。

## When you should / shouldn't use it

| 应该 | 不应该 |
|------|--------|
| 设计 Context 布局与评测切片 | 当作「永远不要用长上下文」 |
| RAG top-k 摆放策略 | 替代权限与注入防护 |

## Related handbook modules

- [context-engineering](../../../core/context-engineering/context-engineering.md)
- [rag](../../../core/rag/rag.md)
- [llm](../../../foundations/llm/llm.md)

## References

- [paper-lost-in-the-middle-2023](../../../references/papers.md#paper-lost-in-the-middle-2023)
- [article-anthropic-context-engineering-2025](../../../references/articles.md#article-anthropic-context-engineering-2025)
