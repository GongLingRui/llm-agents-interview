# Case: Cursor Agent Harness — Dynamic Context Discovery

## Problem

长程 Coding Agent 轨迹中，工具结果、MCP 工具描述、终端日志与历史不断膨胀；静态把一切塞进窗口既贵又吵，截断又丢信息。

## Constraints

- 多模型共用 harness，需模型特异指令 + 跨模型上下文策略
- 第三方 MCP / shell 输出不受第一方工具「最小返回」控制
- Compaction 有损，摘要后仍可能需要找回细节
- 用户期望终端失败可被 Agent 理解，无需手动粘贴

## Architecture（链 Canonical）

```text
Static minimal context（系统要点 · 工具名录）
  + Dynamic discovery via files:
      long tool output · chat history · skills · MCP descs · terminal
  → Agent tools: read/tail/grep/jq
  → Compaction with history file handle
```

| 组件 | 手册映射 |
|------|----------|
| 策展 / 句柄 | [context-engineering](../../core/context-engineering/context-engineering.md) |
| MCP 工具面 | [mcp](../../protocols/mcp/mcp.md) · Host 侧二级加载 |
| Coding 场景 | [coding-agent](../../agent-types/coding-agent/coding-agent.md) |
| 观测成本 | [observability](../../production/observability/observability.md)（token 图） |

## What worked

- **落盘再读** 替代盲目截断，减少过早 summarization
- **MCP 工具描述按 Server 分文件夹**，静态只留名称；文称调用 MCP 的 run 上总 tokens 约 −46.9%（Cursor A/B）
- Skills：描述静态、正文按需
- 摘要后保留 history 文件句柄，支持搜索恢复

## What failed / 风险

- 动态发现可能「漏 lookup」工具 → 需可观测
- −46.9% 不可当跨产品常数
- 文件原语适合 IDE/本地 Agent，不等于所有托管形态

## Metrics（注明来源）

| 指标 | 数值 | 来源 |
|------|------|------|
| MCP 动态加载 A/B | 调用 MCP 的 run 上总 agent tokens ≈ −46.9%（显著，方差大） | [article-cursor-dynamic-context-discovery-2026](../../references/articles.md#article-cursor-dynamic-context-discovery-2026) |

## Mapped modules

context-engineering · coding-agent · mcp · tool-use · observability

## References

- [article-cursor-dynamic-context-discovery-2026](../../references/articles.md#article-cursor-dynamic-context-discovery-2026)
- 机制笔记：[../industry-practices/cursor/dynamic-context-discovery.md](../industry-practices/cursor/dynamic-context-discovery.md)
