# 论文分析规范

> 主题：如何把 Agent / LLM Agent 论文转成工程可消费的知识。

## 目标

论文笔记必须能回答：

1. 论文提出的 **机制** 是什么？
2. 它改变了 Agent 的哪一段（Loop / Memory / Tool / Eval…）？
3. 工程上可落地的部分与不可直接落地的部分？

落点：`research/papers/`

## 选题标准

优先选择直接作用于下列主题的论文：

Agent Loop、Reasoning/Planning、Tool Use、Memory、RAG/Context、Multi-Agent、Evaluation、Computer Use、Coding Agent、RL for Agents

一般不收：与 Agent 无关的纯 NLP 任务刷榜论文（除非明确映射到上述模块）。

## 文档模板（强制结构）

```markdown
# <Paper Title>

## Metadata
- Authors / Year / Venue
- Links (pdf / code)
- Tags: (handbook modules)
- Last reviewed

## TL;DR (5 行内)
机制一句话 + 为何与 Agent Engineering 相关

## Problem setup
## Method / Mechanism
## How it changes the Agent Loop
## Experiments that matter (not all charts)
## Engineering takeaways
## Limitations & failure modes
## When you should / shouldn't use it
## Related handbook modules
## References
```

## 质量要求

- **机制优先于结果表**：先讲清楚算法/流程，再讲指标
- **映射强制**：至少映射到 1 个 `core/` 或 `production/` 模块
- **可执行 takeaway**：至少 3 条工程师能用的决策建议
- **限制必写**：适用前提、成本、不稳定点

## 推荐阅读顺序（写笔记时）

1. Abstract + Intro（问题）
2. Method / Figure（机制）
3. 与本仓库知识图对照
4. Experiments（只取关键消融）
5. Limitations / 相关工作中的竞品机制

## 禁止

- 中英混排的逐段翻译
- 只写「SOTA」不写机制
- 把论文笔记写成博客感想
- 同一论文在 `papers/` 与某个 core 模块各写一份完整笔记（core 只保留短引用链接）
