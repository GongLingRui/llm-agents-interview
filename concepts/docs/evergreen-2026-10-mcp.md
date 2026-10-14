# Evergreen Review — MCP 协议扩展（机制层 · 2026-10-07）

> 范围：`protocols/mcp/mcp.md` 中 Tasks / sampling / elicitation / Skills-over-MCP / MCP Apps / 取消方法；对照公开规范，**不计价、不臆造产品默认**。  
> 方法：抓取 modelcontextprotocol.io（规范锚点 **`2025-11-25`**）与官方扩展页；只钉可复核的 capability / 方法名。  
> Canonical 同步：[mcp.md](../protocols/mcp/mcp.md) §5.5–§5.6、Q28/Q39、Self-review。

## 已核实（机制）

| 主题 | 钉死事实 | 来源 |
|------|----------|------|
| Sampling | Client capability `sampling`（可选 `sampling.tools`）；方法 **`sampling/createMessage`**；规范要求用户可控同意；**不强制**具体 UX | [client/sampling](https://modelcontextprotocol.io/specification/2025-11-25/client/sampling) |
| Elicitation | Capability `elicitation`（`form` / `url`；`{}` ≡ 仅 form）；方法 **`elicitation/create`**；敏感凭据 MUST 用 URL mode，禁止 form 索取密钥 | [client/elicitation](https://modelcontextprotocol.io/specification/2025-11-25/client/elicitation) |
| 普通取消 | **`notifications/cancelled`**（`requestId` + 可选 `reason`）；`initialize` 不可取消；断连 ≠ 取消 | [utilities/cancellation](https://modelcontextprotocol.io/specification/2025-11-25/basic/utilities/cancellation) |
| Task 取消 | Task 增强请求 **MUST** 用 **`tasks/cancel`**，不得用 `notifications/cancelled` | 同上 + Tasks 页 |
| Tasks | **`2025-11-25` 引入且标 experimental**；capability `tasks`；方法 `tasks/get` / `tasks/result` / `tasks/list` / `tasks/cancel`；工具级 `execution.taskSupport` | [utilities/tasks](https://modelcontextprotocol.io/specification/2025-11-25/basic/utilities/tasks) |
| Skills over MCP | 扩展 ID `io.modelcontextprotocol/skills`；SEP-2640 **Final**；`skills/list` + `skills/get` + `resources/read`；可选 directory read | [doc-mcp-skills-extension](../references/docs.md#doc-mcp-skills-extension) |
| MCP Apps | `_meta.ui.resourceUri` → `ui://`；沙箱 iframe；`ui/` JSON-RPC；**Host 支持因客户端而异** | [doc-mcp-apps-extension](../references/docs.md#doc-mcp-apps-extension) |
| 传输（既有） | stdio + Streamable HTTP；HTTP+SSE 为历史兼容 | 传输页 `2025-11-25`（此前已刷新） |

未改机制立场：MCP = 互操作协议层；不替代 Tool Use / Agent Loop；扩展均为 **opt-in / 能力协商**。

## 仍保持 TODO: Verify

- 各 Host 对 **sampling / elicitation / 远程 MCP 授权** 的默认开关与审批 UX
- 目标 Host / SDK 对 **Tasks（experimental）**、**Skills**、**MCP Apps** 的版本钉扎可用性（规范有 ≠ 运行时已实现）
- Skills / Apps **客户端支持矩阵**上的具体产品版本号（扩展页会变，本手册不抄易变名单）

## References 变更

| Ref ID | 动作 |
|--------|------|
| [std-mcp-specification](../references/docs.md#std-mcp-specification) | Summary / Notes 补 Client features、Cancellation、Tasks 锚点 |
| [doc-mcp-skills-extension](../references/docs.md#doc-mcp-skills-extension) | **新增** |
| [doc-mcp-apps-extension](../references/docs.md#doc-mcp-apps-extension) | **新增** |

## 下次建议

- 日历：≤90 天或 MCP 发布新协议日期版本后重跑  
- Tasks 若摘 experimental 标签，更新 §5.6 与 Q39  
- Host UX / 支持矩阵继续产品文档周期复核（勿写入绝对「全 Host 默认开」）
