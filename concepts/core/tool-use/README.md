# Tool Use

> 职责：工具调用机制 — schema、选择、执行、结果回灌、失败处理。

## Canonical 文档

**主文（Source of Truth）**：[tool-use.md](./tool-use.md)

Phase 1 计划拆成的多篇主题已合并进上述 Canonical 文档，避免浅层重复。后续仅在需要独立深潜（例如某一类工具威胁模型）时再拆子文，并回链主文。

## 受众

要实现或加固 Agent 工具面的工程师。

## 前置依赖

`foundations/agent`；建议同时具备 `core/prompt-engineering` 入门。

## 非目标

- MCP 协议细节与生态 → [`protocols/mcp`](../../protocols/mcp/README.md)
- 沙箱实现手册 → [`runtime/sandbox/sandbox.md`](../../runtime/sandbox/sandbox.md)
- 完整威胁模型 → [`production/security`](../../production/security/README.md)
- 控制回路主体 → [`core/agent-loop`](../agent-loop/agent-loop.md)

## 边界

- **不是** MCP：协议与生态在 `protocols/mcp`
- **不是** Sandbox：隔离执行在 [`runtime/sandbox/sandbox.md`](../../runtime/sandbox/sandbox.md)
- **Tool Use = 机制**；**MCP = 互操作协议**

## 上游 / 下游

- 上游：`foundations/agent`
- 下游：[`agent-loop`](../agent-loop/agent-loop.md)、`protocols/mcp`、所有 `agent-types/`
