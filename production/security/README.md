# Security

> 职责：Agent 系统威胁模型与防护 — 注入族、工具权限、数据泄漏、供应链、Sandbox/MCP 信任边界。

## 受众

负责 Agent 上线、权限策略、红队与合规的工程师 / 安全架构师。

## 前置依赖

- 硬前置：`core/tool-use`、`runtime/sandbox`
- 建议：`core/agent-loop`、`protocols/mcp`、`core/context-engineering`、`core/memory`、`core/rag`

## Canonical 主文

| 文件 | 主题 | 状态 |
|------|------|------|
| [`security.md`](./security.md) | 威胁模型；Prompt/Retrieved/Tool/Memory/Context 注入；工具权限；数据泄漏；供应链；Sandbox 隔离机制链；MCP 信任域钩子；45 面试题 | Canonical |

## 主题覆盖（已并入 Canonical）

威胁模型、注入族、工具权限、数据泄漏、供应链与信任边界均在 [security.md](./security.md)。红队语料库、政策引擎 UX 细则等运营深文可另拆，但不得复写沙箱实现（→ [sandbox.md](../../runtime/sandbox/sandbox.md)）。

## 非目标

- 沙箱隔离**实现**手册（进程/FS/网络策略旋钮）→ [`runtime/sandbox/sandbox.md`](../../runtime/sandbox/sandbox.md)
- **容器 / 微 VM / cgroup 等实现细节**（主文禁止展开）
- MCP 协议字段与传输全文 → `protocols/mcp`
- Tool Schema / 执行管线正文 → `core/tool-use`
- 通用 Web OWASP 百科、密码学教程、合规条文逐条释义

## 上游 / 下游

| 方向 | 模块 |
|------|------|
| 上游 | Tool Use、Agent Loop、Context / Memory / RAG、MCP |
| 执行隔离 | [`runtime/sandbox/sandbox.md`](../../runtime/sandbox/sandbox.md)（机制链挂钩，实现不在此） |
| 下游 | Evaluation（安全题包）、Observability（审计信号）、Infrastructure、Design Patterns |
| 场景 | Coding Agent、Browser/Computer Use、Multi-Agent |

## 边界（易混）

| 主题 | 落点 |
|------|------|
| 是否允许调用 / 为何允许 / 信任域 | **本模块** |
| 允许之后的隔离半径与残余逃逸 | `runtime/sandbox` |
| 工具契约与 observation 管线 | `core/tool-use` |
| 协议互操作 | `protocols/mcp`（本模块只写信任域钩子） |
| 检索质量 vs 语料信任标记 | RAG vs **本模块** |
| 记忆写什么 vs 写闸安全政策 | Memory vs **本模块** |
