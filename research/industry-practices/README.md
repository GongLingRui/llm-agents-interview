# Industry Practices

> 职责：整理 Anthropic / OpenAI / Cursor 等公司的工程文章与实践，并映射到本手册模块。  
> 规范：[standards/article-notes.md](../../standards/article-notes.md)  
> **策略**：不写第二套机制正文；深文在 Core/Production Canonical。引用源：[`references/articles.md`](../../references/articles.md)。

## 已登记文章 → 模块映射

| Ref ID | 主映射模块 | 笔记 |
|--------|------------|------|
| [article-anthropic-building-effective-agents-2024](../../references/articles.md#article-anthropic-building-effective-agents-2024) | agent-loop, architecture, design-patterns, tool-use | [anthropic/building-effective-agents.md](./anthropic/building-effective-agents.md) |
| [article-anthropic-writing-tools-for-agents-2025](../../references/articles.md#article-anthropic-writing-tools-for-agents-2025) | tool-use, evaluation | [anthropic/writing-tools-for-agents.md](./anthropic/writing-tools-for-agents.md) |
| [article-anthropic-context-engineering-2025](../../references/articles.md#article-anthropic-context-engineering-2025) | context-engineering, agent-loop, memory | [anthropic/effective-context-engineering.md](./anthropic/effective-context-engineering.md) |
| [article-anthropic-multi-agent-research-2025](../../references/articles.md#article-anthropic-multi-agent-research-2025) | multi-agent, evaluation, observability | [anthropic/multi-agent-research-system.md](./anthropic/multi-agent-research-system.md) |
| [article-cursor-dynamic-context-discovery-2026](../../references/articles.md#article-cursor-dynamic-context-discovery-2026) | context-engineering, coding-agent, mcp | [cursor/dynamic-context-discovery.md](./cursor/dynamic-context-discovery.md) |
| [article-openai-skills-shell-compaction-2026](../../references/articles.md#article-openai-skills-shell-compaction-2026) | coding-agent, sandbox, context-engineering | [openai/skills-shell-compaction.md](./openai/skills-shell-compaction.md) |

## 笔记增量规则

1. 新文章先登记 `references/articles.md`（禁止无 URL 条目）。  
2. 再在本目录按厂商增加短笔记（机制映射表 + 可迁移点）。  
3. 不在此复述 Tool Use / Context 全文。

## 与 Case Studies / OSS 的区别

| 类型 | 焦点 |
|------|------|
| Industry Practices（本目录） | 厂商方法论与工程文 |
| [Case Studies](../case-studies/README.md) | 端到端产品/系统案例 |
| [OSS Analysis](../open-source-analysis/README.md) | 开源实现拆解 |
