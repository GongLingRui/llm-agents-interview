# P6 — Eval + Observability Harness

## Goal

任务集 + 轨迹记录 + 基础指标，改 prompt/工具能量化对比。

## Non-goals

- 完整在线实验平台
- RL 训练

## Depends on modules

- `P1+`
- `production/evaluation`
- `production/observability`

## Architecture sketch

```text
Dataset → Runner → Traces → Scorer → Diff report
```

## Acceptance criteria

- 至少 20 条任务可离线跑
- 产出 success/cost/steps 与失败分类
- 两次配置对比有报告

## Failure modes observed

（练习时填写实际观测；下列为预期高发）

- 只看 demo 三条
- 指标与业务成功脱节
- 轨迹缺 tool 参数

## What I'd improve next

- （练习后填写）

## References

- 路线：[docs/practice-paths.md](../../../docs/practice-paths.md)
- 清单：[../README.md](../README.md)
