# LLM 基础

> 职责：讲清语言模型在 Agent 系统中真正被依赖的能力与限制。

## Canonical 文档

**主文（Source of Truth）**：[llm.md](./llm.md)

核心立场：

- **LLM 是决策器，不是执行器**——产出文本 / tool_calls / structured；副作用由运行时完成  
- **输出是概率的**——流畅 ≠ 正确；「已完成」以 Executor / 状态机为准  
- **上下文窗口是硬预算**——More Context ≠ Better Context  
- Token / 接口形态 / 采样 / 幻觉与截止是 Agent 设计的前置约束，不是 ML 教材

## 受众

准备进入 Agent 的工程师；不要求 ML 研究背景。

## 前置依赖

无（硬前置）。强烈建议读完后进入 [`foundations/agent`](../agent/README.md)。

## 非目标

- 预训练细节大全
- 对齐算法深水 → [`runtime/reinforcement-learning`](../../runtime/reinforcement-learning/README.md)
- Inference 系统实现 → [`runtime/llm-inference`](../../runtime/llm-inference/README.md)
- Prompt 措辞配方深文 → [`core/prompt-engineering`](../../core/prompt-engineering/README.md)
- Context 装配 / Tool 执行 / Loop 控制 → 各自 Canonical（主文只链不重写）

## 上游 / 下游

- 上游：无
- 下游：[`foundations/agent`](../agent/README.md)、[`core/prompt-engineering`](../../core/prompt-engineering/README.md)、[`core/tool-use`](../../core/tool-use/tool-use.md)、[`runtime/llm-inference`](../../runtime/llm-inference/README.md)
