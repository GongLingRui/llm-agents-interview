# P1 — Robust Loop Agent

## Goal

在 P0 上加固：重试、超时、最大步数、人类中断点。

## Non-goals

- 长期记忆
- 检索
- 多租户

## Depends on modules

- `P0`
- `core/agent-loop`
- `core/tool-use`

## Architecture sketch

```text
Loop + budgets(max_steps, timeouts) + retry policy + optional HITL pause
```

## Acceptance criteria

- 工具瞬时失败可重试且有上限
- 达 max_steps 必停并返回部分结果/原因
- 可注入「用户取消」终止

## Failure modes observed

（练习时填写实际观测；下列为预期高发）

- 无限重试烧钱
- 把业务错误当可重试
- 终止后状态不可解释

## What I'd improve next

- （练习后填写）

## References

- 路线：[docs/practice-paths.md](../../../docs/practice-paths.md)
- 清单：[../README.md](../README.md)
