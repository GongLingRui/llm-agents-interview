# P4 — MCP-connected Agent

## Goal

同一 Agent Loop 通过 MCP 消费工具，可替换工具后端。

## Non-goals

- 自研 MCP 规范
- 无限接入不可信 server

## Depends on modules

- `P1`
- `protocols/mcp`
- `core/tool-use`
- `core/agent-loop`

## Architecture sketch

```text
Host/Client → MCP Server tools → normalize to Tool Use envelope → Loop
```

## Acceptance criteria

- 本地工具与 MCP 工具对 Loop 接口一致
- 切换 server 无需改 Loop 内核
- 权限/超时失败可观测

## Failure modes observed

（练习时填写实际观测；下列为预期高发）

- 把 MCP 当唯一工具课
- 过度权限 server
- 发现失败导致静默无工具

## What I'd improve next

- （练习后填写）

## References

- 路线：[docs/practice-paths.md](../../../docs/practice-paths.md)
- 清单：[../README.md](../README.md)
