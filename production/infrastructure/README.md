# Infrastructure

> 职责：运行 Production Agent 所需的平台能力 — 服务化、队列、配置、密钥、多环境、限流配额、多租户，以及与 Inference Gateway 的衔接。

## Canonical 文档

**主文（Source of Truth）**：[infrastructure.md](./infrastructure.md)

覆盖：服务化参考拓扑、异步 Run/队列/租约、配置与密钥、多环境、限流与配额、多租户隔离与计量钩子、与 [`runtime/llm-inference`](../../runtime/llm-inference/README.md) 的 Gateway 契约。Phase 1 多文件计划已合并进主文。

## 受众

设计 Agent 平台拓扑、容量、发布与多租户运行面的工程师。

## 前置依赖

硬前置：[Agent Loop](../../core/agent-loop/agent-loop.md)、[Agent Architecture](../../core/agent-architecture/agent-architecture.md)  
软前置：[LLM Inference](../../runtime/llm-inference/README.md)、[Observability](../observability/README.md)；建议 [Sandbox](../../runtime/sandbox/README.md)

## 非目标

- 通用 Kubernetes / 某云控制台教程
- 评测方法论正文 → [`evaluation`](../evaluation/README.md)
- Security 威胁模型正文 → [`security`](../security/README.md)
- 推理批处理 / KV cache / 路由算法深文 → [`llm-inference`](../../runtime/llm-inference/README.md)
- 架构组件职责定义正文 → [`agent-architecture`](../../core/agent-architecture/agent-architecture.md)

## 边界

| 易混主题 | 放哪里 |
|----------|--------|
| Runtime / 控制面职责切分 | `core/agent-architecture` |
| 单次 Run 内 Loop | `core/agent-loop` |
| TTFT、批处理、前缀缓存 | `runtime/llm-inference` |
| 轨迹与 SLO 遥测模型 | `production/observability` |
| 威胁模型与注入 | `production/security` |
| 沙箱实现 | `runtime/sandbox` |

## 上游 / 下游

- 上游：`core/agent-loop`、`core/agent-architecture`、`runtime/llm-inference`
- 下游：真实部署与 System Design 练习、`production/design-patterns`、企业多租户 Agent 产品
