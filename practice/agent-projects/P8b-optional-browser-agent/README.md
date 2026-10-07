# P8b — Optional Browser Agent

## Goal

在 Tool/Obs/Security 就绪后，做最小浏览器操控任务。

## Non-goals

- 通用计算机控制产品
- 绕过站点条款的爬虫

## Depends on modules

- `P6`
- `agent-types/browser-computer-use`
- `runtime/sandbox`
- `production/security`

## Architecture sketch

```text
Perceive(DOM/screenshot) → Act → Observe；HITL for risky actions
```

## Acceptance criteria

- 完成 1 个受控页面的多步任务
- 高危动作需确认
- 有轨迹可回放

## Failure modes observed

（练习时填写实际观测；下列为预期高发）

- 选择器脆弱
- 截图塞爆上下文
- 未隔离凭证

## What I'd improve next

- （练习后填写）

## References

- 路线：[docs/practice-paths.md](../../../docs/practice-paths.md)
- 清单：[../README.md](../README.md)
