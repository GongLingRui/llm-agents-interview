# P7 — Production Slice

## Goal

最小上线清单：鉴权、限流、密钥、追踪导出。

## Non-goals

- 完整 SaaS
- 多区域容灾

## Depends on modules

- `P6`
- `production/infrastructure`
- `production/security`
- `production/observability`

## Architecture sketch

```text
API gateway → authn/z → queue/run worker → model gateway → traces
```

## Acceptance criteria

- 无密钥进仓库/日志明文
- 有 per-user/rpm 限流
- 一次失败 run 可凭 trace 复盘

## Failure modes observed

（练习时填写实际观测；下列为预期高发）

- 密钥泄漏
- 无预算失控
- 无法关联 user→run→tool

## What I'd improve next

- （练习后填写）

## References

- 路线：[docs/practice-paths.md](../../../docs/practice-paths.md)
- 清单：[../README.md](../README.md)
