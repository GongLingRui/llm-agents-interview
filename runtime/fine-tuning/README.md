# Fine-tuning

> 职责：通过监督微调（及 PEFT）改变模型行为先验，以服务 Agent 的工具使用、风格、领域流程等；并明确与 Prompt / RAG / RL 的边界。

## Canonical 文档

**主文（Source of Truth）**：[fine-tuning.md](./fine-tuning.md)

覆盖：何时微调 vs Prompt/RAG；数据与飞轮；SFT 工程；工具使用行为微调直觉；灾难遗忘与工具过拟合等风险；与 RL 边界；评测门禁与面试题。Phase 1 多文件计划已合并进主文。

## 受众

为 Agent 选型训练路径、构造 SFT 数据、评估回归风险的工程师。

## 前置依赖

硬前置：[`foundations/llm`](../../foundations/llm/README.md)  
强烈建议：[Tool Use](../../core/tool-use/tool-use.md)、[`production/evaluation`](../../production/evaluation/README.md)

## 非目标

- RLHF / DPO / 轨迹级 RL / 奖励设计全文 → [`reinforcement-learning`](../reinforcement-learning/README.md)
- 纯知识注入与检索子系统 → [`core/rag`](../../core/rag/rag.md)
- 工具执行、鉴权、schema 契约正文 → [`core/tool-use`](../../core/tool-use/tool-use.md)
- 推理服务性能 → [`llm-inference`](../llm-inference/README.md)
- 量化 / 蒸馏部署优化 → [`model-optimization`](../model-optimization/README.md)
- 任务评测体系全文 → [`evaluation`](../../production/evaluation/README.md)

## 边界

- **Fine-tuning ≠ RAG ≠ longer prompt**
- **SFT ≠ 任务奖励优化**；**Tool-use SFT ≠ 工具执行**
- **LoRA ≠ 无灾难遗忘**；生产成功日志 ≠ 干净金标
- 偏好优化与 Agent RL → `runtime/reinforcement-learning`

## 上游 / 下游

- 上游：`foundations/llm`；运行时理解依赖 `core/tool-use`、`core/agent-loop`
- 下游：`runtime/reinforcement-learning`、生产评测门禁、领域 / Coding Agent
