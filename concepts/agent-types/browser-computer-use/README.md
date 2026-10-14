# Browser / Computer Use

> 职责：GUI / 浏览器 / 桌面操控类 Agent — 感知、动作空间、反馈与安全边界。

## Canonical 文档

**主文（Source of Truth）**：[browser-computer-use.md](./browser-computer-use.md)

核心立场：

- **Layer1（界面）变了 ≠ Layer2（业务副作用）成功了**
- 感知默认 **过滤 + hybrid**；坐标动作为降级
- 高危副作用必须 **策略 + HITL + 沙箱**；完整威胁模型链 Security

## 受众

设计或加固 GUI / 浏览器 / Computer Use Agent 的工程师。

## 前置依赖

硬前置：[Agent Loop](../../core/agent-loop/agent-loop.md)、[Tool Use](../../core/tool-use/tool-use.md)  
强烈建议：[Context Engineering](../../core/context-engineering/context-engineering.md)、[`observability`](../../production/observability/README.md)、[`security`](../../production/security/README.md)、[`sandbox`](../../runtime/sandbox/README.md)

## 主题覆盖（已并入 Canonical；下表仅作可选深潜拆分索引）

| 主题 | Canonical 落点 |
|------|----------------|
| 动作空间设计 | [browser-computer-use.md](./browser-computer-use.md) |
| 感知与 grounding | 同上 |
| 反馈环 / 可靠性 | 同上 |
| 安全边界 | 同上；威胁模型总论 → [security](../../production/security/security.md) |
| 评测切片 | 同上；方法总论 → [evaluation](../../production/evaluation/evaluation.md) |

## 非目标

- 通用 Tool Calling 契约 → [Tool Use](../../core/tool-use/tool-use.md)
- 沙箱实现手册 → [`runtime/sandbox`](../../runtime/sandbox/README.md)
- 威胁模型总论 → [`production/security`](../../production/security/README.md)
- Coding Agent 深文 → [`coding-agent`](../coding-agent/README.md)
- 厂商产品操作手册 / 软文

## 边界

| 主题 | 去哪里 |
|------|--------|
| 通用沙箱机制 | `runtime/sandbox` |
| 威胁模型总论 | `production/security` |
| 评测体系全文 | `production/evaluation` |
| 与 Coding 对照 | 主文 §9；深文见 coding-agent |

## 上游 / 下游

- 上游：Agent Loop、Tool Use、Context Engineering
- 下游：Observability、Security、Evaluation、Computer-use 案例与实践项目
