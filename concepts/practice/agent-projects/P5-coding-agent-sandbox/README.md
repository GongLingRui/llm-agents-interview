# P5 — Coding Agent + Sandbox

## Goal

文件/命令工具在隔离环境改代码并跑测试。

## Non-goals

- 完整 IDE 产品
- 无沙箱直接宿主机 root

## Depends on modules

- `P1`
- `agent-types/coding-agent`
- `runtime/sandbox`
- `production/security`

## Architecture sketch

```text
Coding tools → Sandbox exec → test feedback → repair loop
```

## Acceptance criteria

- 默认无宿主机敏感路径
- 能对小仓库完成「修测试」闭环至少一次
- 网络/写权限有策略可关

## Failure modes observed

（练习时填写实际观测；下列为预期高发）

- 整仓塞进上下文
- 沙箱逃逸面未评估
- 测试红了仍宣称成功

## What I'd improve next

- （练习后填写）

## References

- 路线：[docs/practice-paths.md](../../../docs/practice-paths.md)
- 清单：[../README.md](../README.md)
