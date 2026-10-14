# RAG

> 职责：检索增强生成作为 Agent 的外部知识子系统 — 索引、检索、重排、引用、与行动结合。

## Canonical 文档

**主文（Source of Truth）**：[rag.md](./rag.md)

核心立场：

- **RAG ≠ Memory ≠ Search ≠ Database ≠ Context Engineering**
- Agentic RAG 是可选形态，**不是**唯一正确形式
- **retrieval quality ≠ generation quality**

## 受众

构建知识密集型 Agent / grounding 系统的工程师。

## 前置依赖

[Agent Loop](../agent-loop/agent-loop.md)、[Tool Use](../tool-use/tool-use.md)  
边界对照：[Memory](../memory/memory.md)、[`context-engineering`](../context-engineering/context-engineering.md)

## 非目标

- Memory 写闸/遗忘 → [memory.md](../memory/memory.md)
- 窗口总装配 → [`context-engineering`](../context-engineering/context-engineering.md)
- MCP → [`protocols/mcp`](../../protocols/mcp/README.md)
- 向量库运维百科 → 不写

## 上游 / 下游

- 上游：Loop、Tool Use、Architecture
- 下游：知识型 Agent、Eval 检索切片、Context 注入
