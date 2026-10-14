# P0 — Minimal Tool Agent

## Goal

实现最小可运行 Tool-using Agent：LLM + 2–3 个工具 + while loop。

## Non-goals

- Memory/RAG
- MCP
- 多 Agent
- 生产鉴权

## Depends on modules

- `foundations/llm`
- `foundations/agent`
- `core/tool-use`
- `core/agent-loop`

## Architecture sketch

```text
User → Loop(build_messages → model → tool_calls? → execute → append) → Final
```

## Acceptance criteria

- 能完成至少 1 个必须调工具的任务
- 每轮可打印：消息数、是否 tool_call、终止原因
- 工具失败时有可见错误回灌（哪怕暂不重试）

## Failure modes observed

（练习时填写实际观测；下列为预期高发）

- 模型不调工具却瞎答
- 参数 schema 不符导致执行崩溃
- 无 max_steps 空转

## What I'd improve next

- （练习后填写）

## References

- 路线：[docs/practice-paths.md](../../../docs/practice-paths.md)
- 清单：[../README.md](../README.md)
