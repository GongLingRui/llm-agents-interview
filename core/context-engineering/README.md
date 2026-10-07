# Context Engineering

> 职责：决定「什么信息、以什么形式、在什么时候」进入模型上下文窗口。

## Canonical 文档

**主文（Source of Truth）**：[context-engineering.md](./context-engineering.md)

核心立场：

- **不是** Prompt Engineering 的改名升级版  
- 属于 **Agent Runtime / Context Management**  
- **More Context ≠ Better Context**；**Longer Context ≠ Better Agent**  
- Memory / RAG / Tool Results 是 **Context Sources**，深文在各自 Canonical

## 受众

实现 Context Builder、治理长程窗口与观察膨胀的工程师。

## 前置依赖

硬前置：[Agent Loop](../agent-loop/agent-loop.md)、[Tool Use](../tool-use/tool-use.md)  
强烈建议：[Memory](../memory/memory.md)、[RAG](../rag/rag.md)

## 非目标

- 指令措辞配方深文 → [`prompt-engineering`](../prompt-engineering/README.md)
- 检索管道 → [rag.md](../rag/rag.md)
- 记忆存储/写闸 → [memory.md](../memory/memory.md)
- MCP → [`protocols/mcp`](../../protocols/mcp/README.md)

## 上游 / 下游

- 上游：Loop、Tool Use、Memory、RAG、Architecture  
- 下游：所有生产 Agent、Coding Agent、评测中的上下文消融
