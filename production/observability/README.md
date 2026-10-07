# Observability

> 职责：让 Agent 行为可追踪、可回放、可诊断 — traces、spans、工具审计、上下文 token 图、用户反馈与失败归因。

## Canonical 文档

**主文（Source of Truth）**：[observability.md](./observability.md)

核心立场：

- **Evaluation = 判好坏**；**Observability = 可观测**（轨迹 / 审计 / 回放 / 诊断）
- 传统 APM 不够：需要 Agent 语义层（run / step / model / tool / termination_reason）
- **默认可观测 ≠ 默认可明文存储**（隐私分级）
- Obs 产出可被 Eval 消费；本文不定义任务成功指标方法论

## 受众

实现 Agent Runtime / 平台可观测、负责线上排障与合规审计的工程师。

## 前置依赖

硬前置：[Agent Loop](../../core/agent-loop/agent-loop.md)、[Tool Use](../../core/tool-use/tool-use.md)  
建议：[Context Engineering](../../core/context-engineering/context-engineering.md)、[Agent Architecture](../../core/agent-architecture/agent-architecture.md)  
边界对照：[`evaluation`](../evaluation/README.md)

## 主题覆盖（已并入 Canonical；可选深潜不再平行维护正文）

| 主题 | 说明 |
|------|------|
| Telemetry 模型 / Trace·Replay | 主文已覆盖；细节扩展再拆子文并回链 |
| 排障剧本 / 反馈信号 / 日志隐私 / SLO | 同上；勿与 [evaluation](../evaluation/evaluation.md) 指标方法论重复 |

## 非目标

- 「好不好」的指标体系、数据集、rubric、LLM-as-judge 方法论 → [`evaluation`](../evaluation/README.md)
- 基础设施组件清单 → [`infrastructure`](../infrastructure/README.md)
- 威胁模型与沙箱硬化正文 → [`security`](../security/README.md)
- 具体 APM/日志厂商安利 → 不写

## 上游 / 下游

- 上游：Agent Loop、Tool Use、Context Engineering、Architecture  
- 下游：线上排障、合规举证、Eval 轨迹导出、Security 审计协作

## 与易混模块的边界

| 模块 | 边界 |
|------|------|
| Evaluation | Eval 定义并测量好坏；Obs 提供证据与诊断，不定义成功 |
| Security | Security 定威胁与防护；Obs 保障审计与脱敏落地 |
| Infrastructure | Infra 提供存储/采集底座；Obs 定 Agent 语义与挂钩 |
| Context Engineering | Context 定装配策略；Obs 度量 token 图与膨胀信号 |
