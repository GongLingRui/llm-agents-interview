# ReAct: Synergizing Reasoning and Acting in Language Models

## Metadata

- Authors / Year / Venue: Yao et al. · 2022 · arXiv:2210.03629
- Links: https://arxiv.org/abs/2210.03629
- Tags: agent-loop, reasoning-planning, tool-use
- Last reviewed: 2026-10-07
- Reference: [paper-react-2022](../../../references/papers.md#paper-react-2022)

## TL;DR

交错生成 **Thought（推理）** 与 **Action（工具/环境动作）**，用 Observation 驱动下一步；把「只推理」与「只行动」合成同一轨迹。是现代 Agent Loop「边想边做」的重要思想源头，但论文范式 ≠ 生产控制面。

## Problem setup

知识密集型 / 交互式任务上，纯 CoT 易幻觉且无法更新知识；纯行动轨迹缺少显式中间推理，难对齐与纠错。

## Method / Mechanism

```text
Thought_t → Action_t → Observation_t → Thought_{t+1} → …
```

- Thought：自由文本中间推理（目标分解、信息需求、策略）
- Action：API / Wikipedia / 环境动作等
- Observation：外部环境返回，进入下一轮上下文

与仅 CoT、仅 Act、以及先全部想再行动的基线对比。

## How it changes the Agent Loop

| Loop 环节 | ReAct 贡献 |
|-----------|------------|
| Decide | 显式 thought 再选动作（可观测的策略痕迹） |
| Act | 工具调用成为一等公民 |
| Observe | 观测进入轨迹，支撑修正 |
| Terminate | 论文侧重任务成功；生产还需预算/HITL（本仓库补） |

## Experiments that matter

- 问答与决策类任务上，交错推理+行动优于单一范式的若干设置（见原文表；部署时重跑自己的域）。
- 人类可读轨迹便于事中干预与事后归因——这比刷榜数字对工程更重要。

## Engineering takeaways

1. 默认 Loop 形态：`reason ↔ act ↔ observe`，而不是「一次 CoT 写完再调工具」。
2. Thought 是可选可观测策略，不是必须持久化的 Plan 对象；显式 Plan 见 Reasoning Canonical。
3. Observation 必须结构化/裁剪，否则轨迹膨胀（Context Engineering）。
4. 生产要加：max_steps、工具鉴权、可复现评测——论文未当作控制面展开。

## Limitations & failure modes

- Thought 本身可错、可冗长、可泄漏敏感推理。
- 无硬预算时易空转或工具刷屏。
- 任务成功 ≠ 工具选择最优或成本可接受。

## When you should / shouldn't use it

| 应该 | 不应该 |
|------|--------|
| 需要环境反馈才能推进的任务 | 单次分类/抽取（Workflow 或单次 LLM 足够） |
| 需要可审计中间理由 | 把「打印 CoT」当成安全边界 |

## Related handbook modules

- [agent-loop](../../../core/agent-loop/agent-loop.md)
- [reasoning-planning](../../../core/reasoning-planning/reasoning-planning.md)
- [tool-use](../../../core/tool-use/tool-use.md)

## References

- [paper-react-2022](../../../references/papers.md#paper-react-2022)
- [paper-cot-2022](../../../references/papers.md#paper-cot-2022)
