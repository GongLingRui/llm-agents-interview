# Memory

> 职责：Agent 的记忆机制 — 短期状态、长期记忆、写入/读取/遗忘策略。

## Canonical 文档

**主文（Source of Truth）**：[memory.md](./memory.md)

核心立场：**默认不该全部写入**。Memory ≠ State ≠ Context ≠ RAG；现实系统允许 overlap，但所有权与政策必须清晰。

## 受众

设计长程 Agent、会话连续性与隐私合规的工程师。

## 前置依赖

硬前置：[Agent Loop](../agent-loop/agent-loop.md)  
建议：[Agent Architecture](../agent-architecture/agent-architecture.md)

## 非目标

- RAG 检索流水线深文 → [rag.md](../rag/rag.md)
- 窗口装配深文 → [`context-engineering`](../context-engineering/context-engineering.md)
- Run checkpoint / 状态机 → [`agent-loop`](../agent-loop/agent-loop.md)

## 边界

| 概念 | 主责 |
|------|------|
| State | 当前 run 真相 |
| Memory | 跨步骤/任务/会话的可召回资产 |
| Context | 当前调用可见投影 |
| RAG | 外部知识语料检索 |

## 上游 / 下游

- 上游：`agent-loop`、`agent-architecture`
- 下游：长程 Agent、Multi-Agent 共享记忆（深文另述）、Privacy/Security
