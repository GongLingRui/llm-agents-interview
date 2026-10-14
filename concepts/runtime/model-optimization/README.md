# Model Optimization

> 职责：在可接受质量下降低推理成本与延迟 — 量化、蒸馏、投机解码、路由到小模型、质量–成本权衡。

## Canonical 文档

**主文（Source of Truth）**：[model-optimization.md](./model-optimization.md)

覆盖：量化 / 蒸馏 / 投机解码 / 路由小模型 / 质量–成本多目标；与 Inference 的清晰边界；与 Evaluation 的强制门禁链接；Agent 多步特有影响与失败模式。Phase 1 多文件计划已合并进主文。

## 受众

要压低 Token/GPU 成本、控制交互延迟、设计多模型策略的工程师。

## 前置依赖

硬前置：[LLM Inference](../llm-inference/README.md)  
建议：[Evaluation](../../production/evaluation/README.md)、[foundations/llm](../../foundations/llm/README.md)

## 非目标

- 连续批处理、KV Cache、前缀缓存、流式、限流背压 → [`llm-inference`](../llm-inference/README.md)
- CUDA / kernel 手册 → 非本手册范围
- SFT / LoRA 训练配方 → [`fine-tuning`](../fine-tuning/README.md)
- RLHF / Agent RL → [`reinforcement-learning`](../reinforcement-learning/README.md)
- 评测体系全文 → [`production/evaluation`](../../production/evaluation/README.md)

## 边界

- **Model Optimization ≠ Inference**：前者改模型表示与选择策略；后者改同一资产的服务效率
- **投机解码**：机制与选型在本文；引擎调度落地在 Inference
- **路由**：质量策略在本文；超时/队列/fallback 执行在 Inference
- **蒸馏 ≠ Fine-tuning**：动机分别为压缩成本 vs 行为对齐（实现可重叠）

## 上游 / 下游

- 上游：`runtime/llm-inference`、`foundations/llm`
- 下游：`production/evaluation`、多模型生产部署、成本治理；与 `fine-tuning` / `reinforcement-learning` 可组合但门禁分离
