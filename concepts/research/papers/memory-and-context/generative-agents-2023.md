# Generative Agents: Interactive Simulacra of Human Behavior

## Metadata

- Authors / Year / Venue: Park et al. · 2023 · arXiv:2304.03442
- Links: https://arxiv.org/abs/2304.03442
- Tags: memory, reasoning-planning
- Last reviewed: 2026-10-07
- Reference: [paper-generative-agents-2023](../../../references/papers.md#paper-generative-agents-2023)

## TL;DR

用自然语言 **memory stream** 记录完整经历，经检索与 **reflection** 合成高层记忆，再驱动规划与行为。展示 memory + retrieval + reflection 的可映射架构；沙盒仿真 ≠ 生产 Memory 控制面（ACL、遗忘、审计仍需工程补齐）。

## Problem setup

要让 LLM 角色在长期交互中表现连贯、可信的行为，仅靠当前对话窗口不够：需要积累经历、提取抽象信念，并据此做日程级规划。

## Method / Mechanism

```text
观察/行动 → 写入 memory stream（自然语言记忆条目）
检索：按相关性、近因、重要性等对记忆打分取子集
反思：周期性从记忆合成更抽象的高层记忆并写回
规划：基于检索/反思结果生成或修订计划 → 行动
```

闭环：**感知 → 记忆 → 检索/反思 → 计划 → 行动 → 新观察**。记忆是一等存储，不只是 prompt 拼接。

## How it changes the Agent Loop

| 环节 | 变化 |
|------|------|
| Observe | 观测持久化进 stream，而非用完即弃 |
| Memory | 检索条件化 Decide；反思做巩固/抽象 |
| Decide | 计划受长期记忆与当前处境共同约束 |

与 RAG：此处记忆是 **主体经历**，不是外部知识库语料。与 Context：每次调用只注入检索出的投影。

## Experiments that matter

- 沙盒社会仿真中，架构支持涌现式社会行为与可访谈的「记忆一致性」（见原文评测设计）。
- 工程迁移重点：检索打分维度与反思频率，而非复刻小镇剧情。

## Engineering takeaways

1. 拆分：原始事件日志 vs 反思摘要；写入前过写闸（PII、密钥、可遗忘）。
2. 检索公式（相关/近因/重要）可产品化，但阈值需可调、可观测。
3. Reflection 是巩固策略，不是免费真理——错误反思会污染长期行为。
4. 生产必须叠加：租户 ACL、用户删除、出处、容量淘汰；论文未当控制面展开。

## Limitations & failure modes

- 仿真环境假设强；真实产品有合规与多租户约束。
- 记忆膨胀与错误巩固。
- 规划依赖检索质量；漏检关键事实 → 行为漂移。

## When you should / shouldn't use it

| 应该 | 不应该 |
|------|--------|
| 设计长期角色/助手记忆架构参考 | 照搬无政策的「全量自然语言日记」 |
| 需要跨会话连贯人设/目标 | 替代业务数据库或权限系统 |

## Related handbook modules

- [memory](../../../core/memory/memory.md)
- [reasoning-planning](../../../core/reasoning-planning/reasoning-planning.md)
- [context-engineering](../../../core/context-engineering/context-engineering.md)

## References

- [paper-generative-agents-2023](../../../references/papers.md#paper-generative-agents-2023)
- [paper-reflexion-2023](../../../references/papers.md#paper-reflexion-2023)
- [paper-memgpt-2023](../../../references/papers.md#paper-memgpt-2023)
