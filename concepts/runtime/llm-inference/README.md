# LLM Inference

> 职责：推理侧工程 — 批处理、KV/前缀缓存、流式、限流、多租户与延迟结构（TTFT/TPOT），以及它们如何放大 Agent Loop 的预填成本。

## Canonical 文档

**主文（Source of Truth）**：[llm-inference.md](./llm-inference.md)

Phase 1 计划拆成的多篇主题已合并进上述 Canonical 文档。量化/蒸馏/投机解码仍在 [`model-optimization`](../model-optimization/README.md)。

## 受众

实现 / 选型 LLM 网关、做容量与多租户配额、为 Agent Runtime 解释延迟与账单的工程师。

## 前置依赖

- [`foundations/llm`](../../foundations/llm/README.md)（token / 窗口直觉）  
- 强烈建议：[Agent Loop](../../core/agent-loop/agent-loop.md)、[Context Engineering](../../core/context-engineering/context-engineering.md)

## 非目标

- 量化、蒸馏、投机解码算法深文 → [`model-optimization`](../model-optimization/README.md)
- CUDA / 算子手册 → 不写
- 集群拓扑百科 → [`production/infrastructure`](../../production/infrastructure/README.md)
- Loop 控制流 / Context 策展正文 → 各自 Canonical

## 边界

| 易混模块 | 边界 |
|----------|------|
| `model-optimization` | 模型侧压缩与加速；本文只保留接口级链接 |
| `core/agent-loop` | 为何多步调用；本文解释每步如何被服务与计费 |
| `core/context-engineering` | 放什么进窗口；本文解释窗口长度如何变成 Prefill/缓存命中 |
| `production/infrastructure` | 部署拓扑；本文聚焦调度与延迟结构 |

## 上游 / 下游

- 上游：`foundations/llm`、Agent Loop、Context Engineering  
- 下游：`model-optimization`、Production（Infrastructure / Observability）、各 Agent Types 的容量实践
