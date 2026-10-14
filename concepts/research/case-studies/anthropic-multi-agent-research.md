# Case: Anthropic Multi-Agent Research（产品 Research 功能）

## Problem

开放域「研究」任务路径不可预知：需要多源搜索、随发现转向、最终给出带引用的综合答案。单 Agent 单窗口在广度优先查询上易陷入串行慢搜或上下文塞满。

## Constraints

- 必须对接 Web / Workspace / 集成工具等多源
- 长程运行：状态持久、失败可恢复，不能「整段重来」
- 成本敏感：多 Agent token 消耗显著高于 chat
- 输出需可归因（引用位置），不仅「听起来像研究」
- 隐私：观测需避免不当暴露会话内容（文述高层 tracing）

## Architecture（链 Canonical，不重写机制）

```text
User
  → LeadResearcher（Plan → Memory；委派）
  → Subagents × N（并行独立 Context + 搜索工具）
  → Lead 综合 / 再派
  → CitationAgent（声明↔来源）
  → Answer
```

| 组件 | 手册映射 |
|------|----------|
| Lead / Worker | [multi-agent](../../agent-types/multi-agent/multi-agent.md) · [design-patterns](../../production/design-patterns/design-patterns.md) Orchestrator-workers |
| 每 Agent 仍是 Loop | [agent-loop](../../core/agent-loop/agent-loop.md) |
| 计划防截断 | [memory](../../core/memory/memory.md) · [context-engineering](../../core/context-engineering/context-engineering.md) |
| 工具 ACI | [tool-use](../../core/tool-use/tool-use.md) |
| 结果导向评测 | [evaluation](../../production/evaluation/evaluation.md) |
| Tracing / 部署 | [observability](../../production/observability/observability.md) · [infrastructure](../../production/infrastructure/infrastructure.md) |

相对静态 RAG：动态多步搜索，而非一次性相似度取片。

## What worked

- **真并行探索**：子 Agent 各自上下文，压缩后回传 Lead（广度查询受益）
- **委派合同**：目标、输出格式、工具/源、边界写清，减少重复劳动
- **努力缩放启发式**：简单查询少派、复杂查询多派，抑制「50 个子 Agent」
- **小样本 Eval 先行** + LLM-judge rubric（事实/引用/完整/源质量/工具效率）
- **生产工程**：checkpoint/resume、rainbow 部署、全链路 trace

## What failed（早期 / 持续风险）

- 滥生子 Agent、无效穷搜、子 Agent 互相干扰
- 委派过短 → 任务误解与重复搜索
- 同步等待子 Agent → 信息流瓶颈；异步化引入状态一致性难题
- SEO 内容农场偏好（人工评测发现后靠源质量启发式修正）
- Token 成本：文称 Multi-Agent ~15× chat——任务价值不够则不经济

## Metrics（注明来源；不可核实则标注）

| 指标 | 数值 / 说明 | 来源 |
|------|-------------|------|
| 内部 research eval 相对单 Agent | +90.2%（Opus4 Lead + Sonnet4 workers vs 单 Opus4） | Anthropic 工程文；**内部集，不可公开复现** |
| Token 相对 chat | Agent ~4×；Multi-Agent ~15× | 同上，内部数据 |
| BrowseComp 方差解释 | token 用量 ~80%；+工具次数+模型 ≈95% | 同上分析 |
| 工具描述自改进后任务时间 | ~−40% | 同上，特定工具集 |

> 本案例 **不**把上述数字当作行业 SLA；落地必须用自有任务集重测。

## Mapped modules

multi-agent · agent-architecture · agent-loop · tool-use · memory · context-engineering · evaluation · observability · infrastructure · design-patterns

## References

- [article-anthropic-multi-agent-research-2025](../../references/articles.md#article-anthropic-multi-agent-research-2025)
- 机制笔记：[../industry-practices/anthropic/multi-agent-research-system.md](../industry-practices/anthropic/multi-agent-research-system.md)
- 对照立场：[multi-agent.md](../../agent-types/multi-agent/multi-agent.md)（默认单 Agent）
