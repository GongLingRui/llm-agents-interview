# P8a — Optional Multi-Agent

## Goal

仅当单 Agent 职责清晰仍不够时，做最小分工编排。

## Non-goals

- 默认上多 Agent
- 复杂辩论系统

## Depends on modules

- `P2+`
- `agent-types/multi-agent`
- `core/agent-architecture`

## Architecture sketch

```text
Supervisor 或 pipeline；子 Agent 只回摘要+句柄
```

## Acceptance criteria

- 书面说明为何单 Agent 不够
- 子轨迹不整段灌入主上下文
- 冲突/失败有归属

## Failure modes observed

（练习时填写实际观测；下列为预期高发）

- 为炫技拆 Agent
- 共享可变状态无锁
- 成本翻倍无收益

## What I'd improve next

- （练习后填写）

## References

- 路线：[docs/practice-paths.md](../../../docs/practice-paths.md)
- 清单：[../README.md](../README.md)
