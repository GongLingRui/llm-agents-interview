# Plan-and-Solve Prompting: Improving Zero-Shot Chain-of-Thought Reasoning by Large Language Models

## Metadata

- Authors / Year / Venue: Wang et al. · 2023 · arXiv:2305.04091
- Links: https://arxiv.org/abs/2305.04091
- Tags: reasoning-planning
- Last reviewed: 2026-10-07
- Reference: [paper-plan-and-solve-2023](../../../references/papers.md#paper-plan-and-solve-2023)

## TL;DR

在 Zero-shot-CoT 上改为 **先写计划再按计划求解**（PS / PS+），缓解缺步、计算错误等。提供「plan-then-solve」直觉；**仍是 prompting 范式，不等于生产里的持久 Plan 状态机**。

## Problem setup

Zero-shot-CoT（如 “Let’s think step by step”）常缺步、算错或计划含糊；需要更明确的「先规划后执行」提示结构，且尽量少依赖 few-shot 示例。

## Method / Mechanism

```text
PS:  指令 → 先输出整体计划 → 再逐步求解 → 答案
PS+: 强化计划质量（更细步骤、变量抽取、计算自检等提示要素）
```

相对「一步写出推理+答案」，显式拆成 **计划痕迹** 与 **执行痕迹**。无外部工具、无搜索树；核心是提示形态。

## How it changes the Agent Loop

- **Decide 前**：可诱导一段「计划」文本再行动。
- **不自动**创建可校验、可回滚的 Plan 对象；无 Act/Observe 契约变更。
- 与生产 Planner-Executor：论文 ≈ 提示层直觉；Canonical 仍要求结构化状态、预算与失败重规划。

## Experiments that matter

- 相对 Zero-shot-CoT 基线的多步推理改进（见原文）；迁移时以你们任务与模型重测为准。
- 工程含义：结构化「先列步骤」有时比盲目加长 CoT 更稳，但仍属提示技巧。

## Engineering takeaways

1. Plan-and-Solve ≠ 持久 Plan：生产需 ID、步骤状态、权限与重规划钩子。
2. 可把「先输出 bullet 计划」当作廉价默认；复杂域再升级状态机。
3. PS+ 中的自检提示可与 Verifier 节点组合，勿只靠模型自觉。
4. 有工具时：计划步骤应映射到允许的 tool 集合，禁止自由发挥危险动作。

## Limitations & failure modes

- 计划本身可错、可空泛；执行阶段照错计划走。
- 无环境反馈时无法发现计划过时。
- Token 略增；对简单任务可能过度规划。

## When you should / shouldn't use it

| 应该 | 不应该 |
|------|--------|
| Zero-shot 多步算术/逻辑类提示基线 | 当作完整 Agent Planning 架构 |
| 诱导可读中间计划供人工抽查 | 替代 ToT 搜索或 Tool 闭环 |

## Related handbook modules

- [reasoning-planning](../../../core/reasoning-planning/reasoning-planning.md)
- [prompt-engineering](../../../core/prompt-engineering/prompt-engineering.md)

## References

- [paper-plan-and-solve-2023](../../../references/papers.md#paper-plan-and-solve-2023)
- [paper-cot-2022](../../../references/papers.md#paper-cot-2022)
