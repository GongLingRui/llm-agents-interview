# OSS / Spec: Model Context Protocol（MCP）

## Metadata

- Repo / License / Language: [repo-modelcontextprotocol-specification](../../../references/repos.md#repo-modelcontextprotocol-specification) · License 见上游仓库 · protocol spec（多语言 Server 生态）
- Last reviewed: 2026-10-07
- Canonical modules mapped: mcp, tool-use, security, context-engineering

## Scope / Non-goals

**范围**：把 MCP 作为 **互操作协议** 拆解——Host / Client / Server、Tools·Resources·Prompts、传输与信任边界；映射本手册模块。

**非目标**：

- 复写 [mcp.md](../../../protocols/mcp/mcp.md) Canonical 全文
- 把 MCP 写成「又一种 Tool Calling 教程」
- 枚举全部生态 Server 清单

**核实基线**：[std-mcp-specification](../../../references/docs.md#std-mcp-specification) · [repo-modelcontextprotocol-specification](../../../references/repos.md#repo-modelcontextprotocol-specification) · 产品页 https://modelcontextprotocol.io/introduction

## Agent Loop 映射

| Agent 概念 | MCP 角色 |
|------------|----------|
| Tool surface | Server 暴露的 tools（schema + 执行在 Server/Host 约定侧） |
| Context 注入 | resources / prompts 可作为策展素材，仍由 Host Assembler 决定是否进窗 |
| Loop 控制 | **不在** MCP 内；Host 的 Agent Loop 决定何时 list/call |
| 终止 / 预算 | Host 职责 |

```text
Agent Host（Loop · Policy · Assembler）
    ↕ MCP Client
MCP Server（tools / resources / prompts）
```

## Tool surface

- 协议统一「发现 + 调用」形状，降低一次性私有插件协议成本。
- **ACI 质量仍决定成败**：描述差 → 错工具；见 Tool Use / Anthropic 工具文。
- 工具爆炸：Host 应做动态发现（Cursor：工具描述落盘按需读），不是全量 schema 常驻。

## State / Memory / Context

- MCP 不提供跨会话 Memory 语义。
- Resource 内容默认 **不可信**（间接注入面）→ Security / Context 分区。
- 采样/elicitation 等能力以钉死的协议版本为准（TODO: Verify 产品默认开关）。

## Eval / Obs hooks

- 记录：server 标识、tool name、args 摘要、latency、错误码、协议版本
- Eval：按 Server/工具切片成功率与误用率
- 鉴权失败应可观测（勿静默「忘掉工具」）

## Reusable designs

1. **协议层 vs 机制层**：MCP 解决互操作；Tool Use 解决选择/执行/回灌。
2. **信任域**：远程 Server ≠ 本地进程；网络与密钥政策独立评审。
3. **Host 责任**：上下文预算、工具名录二级加载、HITL 闸门。

## Anti-patterns

- 「接了 MCP 就是 Agent」
- 数百工具全量进 system prompt
- 把 Server 返回当可信指令执行
- 不钉协议版本的生产集成

## Mapped handbook modules

- [mcp](../../../protocols/mcp/mcp.md)（主）
- [tool-use](../../../core/tool-use/tool-use.md)
- [security](../../../production/security/security.md)
- [context-engineering](../../../core/context-engineering/context-engineering.md)

## References

- [std-mcp-specification](../../../references/docs.md#std-mcp-specification)
- [repo-modelcontextprotocol-specification](../../../references/repos.md#repo-modelcontextprotocol-specification)
- [article-cursor-dynamic-context-discovery-2026](../../../references/articles.md#article-cursor-dynamic-context-discovery-2026)
