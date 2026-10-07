# Dynamic context discovery（Cursor）

## Metadata

- Source / Author / Date: Cursor Blog · Jediah Katz · 2026-01-06
- URL: https://cursor.com/blog/dynamic-context-discovery
- Mapped modules: context-engineering, coding-agent, mcp, tool-use, agent-loop
- Last reviewed: 2026-10-07
- Reference: [article-cursor-dynamic-context-discovery-2026](../../../references/articles.md#article-cursor-dynamic-context-discovery-2026)

## Claim（作者主张）

1. 随模型变强，成功模式是 **少预置、多按需拉取**（dynamic context discovery），相对「始终塞入」的 static context。
2. **文件**是动态发现的简单原语：长工具结果、聊天历史、Skills、MCP 工具描述、终端会话均可落盘再检索。
3. Host（Coding Agent）应承担 MCP 工具面的上下文削减责任，不能指望每个 Server 自行优化。

## Mechanism（可验证的机制）

| 做法 | 机制 |
|------|------|
| 长工具结果 → 文件 | 不截断丢信息；Agent `tail`/分段读 |
| 摘要后保留 history 文件句柄 | Compaction 有损时可用搜索恢复细节 |
| Skills | 名称/描述静态；正文按需发现 |
| MCP 工具描述同步到按 Server 分文件夹 | 静态只留工具名；需要时再 lookup |
| 集成终端 → 文件 | 用户不必粘贴；可 grep 长日志 |

文中 A/B：在「调用了 MCP 工具」的 run 上，动态 MCP 策略使总 agent tokens 约 **−46.9%**（显著，方差随安装 MCP 数变化）。

## Evidence

- Cursor 产品 harness 工程博文 + 内部 A/B（MCP 子集）。
- 明确对比「截断」vs「落盘再读」的失败模式差异。

## Engineering translation（我们怎么用）

| 决策 | 本手册 |
|------|--------|
| Observation → 句柄而非全文 | [context-engineering](../../../core/context-engineering/context-engineering.md) |
| MCP 工具爆炸 | [mcp](../../../protocols/mcp/mcp.md) + Host 侧目录/检索，而非全量 schema 常驻 |
| Skills / 程序化 playbook | [coding-agent](../../../agent-types/coding-agent/coding-agent.md) · Prompt 分区 |
| Compaction + 可恢复历史 | Context Canonical；Memory 写闸仍独立 |

**可执行下一步**：第三方工具默认 `store + pointer`；MCP 工具表做「名录静态 / 详情动态」。

## Disagreement / Caveats

- 「文件原语」是 Coding Agent 友好抽象，不等于所有产品形态都必须落本地 FS。
- −46.9% 是 Cursor 特定 A/B，不可当跨产品常数。
- 动态发现失败时可能漏工具——需可观测「未 lookup」信号。

## Action items

1. 审计当前工具结果是否整页进窗口。
2. MCP：实现工具名录与详情二级加载。
3. Compaction 后保留可检索历史句柄。

## References

- [article-cursor-dynamic-context-discovery-2026](../../../references/articles.md#article-cursor-dynamic-context-discovery-2026)
- 对照：[context-engineering](../../../core/context-engineering/context-engineering.md) · [mcp](../../../protocols/mcp/mcp.md)
