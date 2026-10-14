# 阅读路线

> 主题：Reading Paths — 在内容逐渐填充后，如何按目的高效阅读，而不是从目录扫到尾。

## 原则

1. **先机制后场景**：先读 Core，再读 Agent Types / Industry。
2. **先规范后笔记**：读论文/OSS 前先看 `standards/`。
3. **带着问题读**：每个阅读单元结束时回答一个工程问题（见各路径「验收问题」）。
4. **交叉引用优先跳转**：遇到 Memory/RAG/Context 三角，按知识图跳，不平行开三篇硬读。

---

## Route R1 — 周末速通（机制骨架）

适合：只有 1–2 天，想建立正确心智模型。

1. [Agent 基础（Canonical）](../foundations/agent/agent.md)
2. [Tool Use](../core/tool-use/tool-use.md)
3. [Agent Loop](../core/agent-loop/agent-loop.md)
4. [Agent Architecture](../core/agent-architecture/agent-architecture.md)
5. [知识依赖关系](./knowledge-graph.md)

**验收问题**：用不超过 10 句话解释「Agent 与 Chatbot 的本质区别」，并画出 Loop。

---

## Route R2 — Core 深潜（系统理解）

1. [Tool Use](../core/tool-use/tool-use.md) → [Agent Loop](../core/agent-loop/agent-loop.md) → [Reasoning/Planning](../core/reasoning-planning/reasoning-planning.md)
2. [Memory](../core/memory/memory.md) → [RAG](../core/rag/rag.md) → [Context Engineering](../core/context-engineering/context-engineering.md)
3. [Prompt Engineering](../core/prompt-engineering/prompt-engineering.md)（回看：指令合同，不是 Context 中心）
4. [MCP](../protocols/mcp/mcp.md)（在 Tool Use 之后；协议 ≠ 调用策略）

**验收问题**：给定一个长任务 Agent，你会把状态放 Memory、检索放 RAG、还是每次重算进 Context？为什么？

---

## Route R3 — 垂直 Agent 阅读

### Coding Agent

[coding-agent.md](../agent-types/coding-agent/coding-agent.md) → [tool-use](../core/tool-use/tool-use.md) / [context-engineering](../core/context-engineering/context-engineering.md) / [sandbox](../runtime/sandbox/sandbox.md) → [industry-practices](../research/industry-practices/README.md) → [open-source-analysis](../research/open-source-analysis/README.md)

### Browser / Computer Use

[browser-computer-use.md](../agent-types/browser-computer-use/browser-computer-use.md) → [tool-use](../core/tool-use/tool-use.md) → [observability](../production/observability/observability.md) → [security](../production/security/security.md) → 相关 papers

### Multi-Agent

[agent-architecture](../core/agent-architecture/agent-architecture.md) → [reasoning-planning](../core/reasoning-planning/reasoning-planning.md) → [multi-agent](../agent-types/multi-agent/multi-agent.md) → [evaluation](../production/evaluation/evaluation.md) → [design-patterns](../production/design-patterns/design-patterns.md)

---

## Route R4 — Production 阅读

顺序固定：

[evaluation](../production/evaluation/evaluation.md) → [observability](../production/observability/observability.md) → [security](../production/security/security.md)（含 [sandbox](../runtime/sandbox/sandbox.md)）→ [infrastructure](../production/infrastructure/infrastructure.md) → [design-patterns](../production/design-patterns/design-patterns.md)

**验收问题**：如果线上 Agent 突然变贵且变慢，你先看哪三类信号？

---

## Route R5 — 论文 / 开源对照读

1. 选一个机制主题（例如 Tool Calling / ReAct）
2. 读 [paper-analysis.md](../standards/paper-analysis.md) + 对应笔记（见 [research/papers](../research/papers/README.md)）
3. 读对应 `core/` Canonical
4. 读 1 篇 [open-source-analysis](../research/open-source-analysis/README.md)
5. 写「论文主张 vs 实现取舍」对照表

---

## Route R6 — 厂商实践阅读（防迷失）

顺序：

1. 先读相关 Core Canonical（建立批判框架）
2. 再读 [industry-practices](../research/industry-practices/README.md) 下已有笔记
3. 把文章主张映射回：Loop / Context / Eval / Security 中的哪一层
4. 拒绝只收藏链接不提炼机制

---

## 阅读与学习的区别

| | 学习路线 | 阅读路线 |
|--|----------|----------|
| 目标 | 获得能力与产出 | 高效吸收已有文档 |
| 是否要求动手 | 是 | 可选 |
| 退出条件 | 项目/设计产出 | 能回答验收问题 |
