# How we built our multi-agent research system（Anthropic）

## Metadata

- Source / Author / Date: Anthropic Engineering · 2025-06-13
- URL: https://www.anthropic.com/engineering/multi-agent-research-system
- Mapped modules: multi-agent, agent-architecture, evaluation, observability, context-engineering, tool-use, memory
- Last reviewed: 2026-10-07
- Reference: [article-anthropic-multi-agent-research-2025](../../../references/articles.md#article-anthropic-multi-agent-research-2025)

## Claim（作者主张）

1. Research 类开放任务适合 **orchestrator–worker** 多 Agent：Lead 规划并并行派生子 Agent 搜索，再汇总；另有 CitationAgent 做引用定位。
2. 多 Agent 主要通过 **并行上下文窗口** 扩大有效 token 预算；成本显著更高（文：Agent ~4× chat，Multi-Agent ~15× chat）。
3. 协调复杂度爆炸；早期失败包括滥生子 Agent、无效穷搜、互相干扰——主要靠提示启发式 + 工具 ACI + Eval/观测补齐。

## Mechanism（可验证的机制）

```text
User query
  → LeadResearcher（计划写入 Memory，防截断丢计划）
  → 并行 Subagents（独立上下文 + 搜索工具）
  → Lead 综合 / 决定是否再派
  → CitationAgent（声明↔来源位置）
  → 用户结果
```

与静态 RAG 对比：多步动态搜索，随发现调整，而非一次性 top-k。

提示原则摘要：像 Agent 一样调试、委派要带目标/格式/边界、按复杂度缩放努力、工具选择启发式、宽搜再收窄、并行工具调用、过程可观测。

生产：可恢复检查点、全量 tracing（隐私约束下）、rainbow 部署、同步子 Agent 的瓶颈与异步化权衡。

## Evidence（注明来源）

- 内部 research eval：Opus4 Lead + Sonnet4 子 Agent 相对单 Opus4 **+90.2%**（Anthropic 内部集，非公开复现基准）。
- BrowseComp 分析：token 用量解释约 80% 方差；token+工具次数+模型共解释约 95%（文内分析）。
- 工具描述自改进 Agent：任务完成时间约 **−40%**（文内；工具集特定）。

## Engineering translation

| 教训 | 本手册 |
|------|--------|
| 默认仍单 Agent；并行探索有评测证据再拆 | [multi-agent](../../../agent-types/multi-agent/multi-agent.md) |
| Orchestrator-workers | [design-patterns](../../../production/design-patterns/design-patterns.md) · [architecture](../../../core/agent-architecture/agent-architecture.md) |
| 计划进 Memory 防截断 | [memory](../../../core/memory/memory.md) · Context compaction |
| 结果导向 Eval + 小样本起步 | [evaluation](../../../production/evaluation/evaluation.md) |
| Tracing 诊断动态失败 | [observability](../../../production/observability/observability.md) |
| ACI / 工具描述 | [tool-use](../../../core/tool-use/tool-use.md) |

## Disagreement / Caveats

- +90.2% / 15× token 均为 Anthropic 内部数据，迁移时必须自建 eval。
- 文承认多数 coding 任务并行度低于 research——与本仓库「Coding 默认单 Agent」一致。
- 同步子 Agent 简化协调但制造瓶颈。

## Action items

1. 仅在「高价值 + 真并行」任务试点 Multi-Agent。
2. Lead 委派模板强制：目标、输出格式、工具/源、边界、预算。
3. 上线前具备 resume/checkpoint + trace。

## References

- [article-anthropic-multi-agent-research-2025](../../../references/articles.md#article-anthropic-multi-agent-research-2025)
