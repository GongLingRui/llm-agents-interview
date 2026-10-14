# MemGPT: Towards LLMs as Operating Systems

## Metadata

- Authors / Year / Venue: Packer et al. · 2023 · arXiv:2310.08560
- Links: https://arxiv.org/abs/2310.08560
- Tags: memory, context-engineering
- Last reviewed: 2026-10-07
- Reference: [paper-memgpt-2023](../../../references/papers.md#paper-memgpt-2023)

## TL;DR

借鉴 OS **分层存储 / 分页**：在有限上下文窗口与外置存储之间调度读写，支撑长会话与文档分析。启发「热/温/冷」记忆与虚拟上下文；**不能只靠模型自己管记忆而省略写入政策与 ACL**。

## Problem setup

固定窗口装不下长期对话与大文档；单纯截断或摘要会丢关键状态。需要把「主存（窗口）」与「外存（向量库/档案）」显式编排。

## Method / Mechanism

```text
Main Context（工作集，受窗口限制）
  ↕ 模型通过记忆管理动作（读/写/分页式换入换出）
External Context（归档对话、文档块等）
控制：函数调用式 memory ops + 系统提示规定层级职责
```

思想：LLM 像 OS 进程，主动决定何进何出，而非一次塞满窗口。

## How it changes the Agent Loop

| 环节 | 变化 |
|------|------|
| Context 装配 | 工作集由调度策略填充，非整史粘贴 |
| Act | 增加 memory 读写类「工具」 |
| Memory | 外置存储成为一等公民；窗口只是缓存 |

与 Context Engineering：分页/换出是 **装配策略**；与 Memory Canonical：层级思想可借鉴，政策与审计仍外置。

## Experiments that matter

- 长对话一致性与文档问答类设定上，相对朴素截断基线的机制优势（见原文；生产用自有语料与窗口重测）。
- 工程关键：换入换出次数、工作集命中率、错误驱逐导致的「假失忆」。

## Engineering takeaways

1. 分层：热（当前 State）/ 温（可检索情节）/ 冷（归档）；TTL 与容量显式配置。
2. Memory ops 必须鉴权、审计、可回放；勿把「模型决定删除」当唯一真相。
3. 换出前摘要有损——关键约束应钉在系统区，不参与随意驱逐。
4. 与 RAG 分工：MemGPT 管会话/工作集调度；语料事实检索仍走 RAG 管道。

## Limitations & failure modes

- 模型错误换出关键事实 → 静默能力退化。
- 自我管理记忆增加工具步数与延迟。
- 论文原型 ≠ 多租户安全产品。

## When you should / shouldn't use it

| 应该 | 不应该 |
|------|--------|
| 长会话、大文档分析的架构灵感 | 无写闸地「全权交给模型管记忆」 |
| 设计虚拟上下文/工作集策略 | 替代权限、遗忘权、合规删除 |

## Related handbook modules

- [memory](../../../core/memory/memory.md)
- [context-engineering](../../../core/context-engineering/context-engineering.md)
- [agent-loop](../../../core/agent-loop/agent-loop.md)

## References

- [paper-memgpt-2023](../../../references/papers.md#paper-memgpt-2023)
- [paper-generative-agents-2023](../../../references/papers.md#paper-generative-agents-2023)
- [paper-lost-in-the-middle-2023](../../../references/papers.md#paper-lost-in-the-middle-2023)
