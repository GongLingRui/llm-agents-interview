# P2 — Memory + Context Budget

## Goal

加入短记忆与窗口预算：摘要/滑动窗，关键状态可恢复。

## Non-goals

- 完整向量知识库
- 跨组织记忆合规产品

## Depends on modules

- `P1`
- `core/memory`
- `core/context-engineering`

## Architecture sketch

```text
State store + optional memory write gate + ContextBuilder(token budget)
```

## Acceptance criteria

- 长对话不会无脑塞满上下文
- 硬约束在压缩后仍可测保留
- 重启后能从 checkpoint 恢复目标与未决项

## Failure modes observed

（练习时填写实际观测；下列为预期高发）

- 摘要丢掉禁止项
- Observation 原样回灌爆炸
- 把窗口当数据库

## What I'd improve next

- （练习后填写）

## References

- 路线：[docs/practice-paths.md](../../../docs/practice-paths.md)
- 清单：[../README.md](../README.md)
