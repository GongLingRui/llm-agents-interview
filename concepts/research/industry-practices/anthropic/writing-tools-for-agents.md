# Writing effective tools for agents（Anthropic）

## Metadata

- Source / Author / Date: Anthropic Engineering · 2025-09-11
- URL: https://www.anthropic.com/engineering/writing-tools-for-agents
- Mapped modules: tool-use, evaluation, prompt-engineering, mcp, coding-agent
- Last reviewed: 2026-10-07
- Reference: [article-anthropic-writing-tools-for-agents-2025](../../../references/articles.md#article-anthropic-writing-tools-for-agents-2025)

## Claim（作者主张）

1. 工具是 **确定性系统与非确定性 Agent** 之间的契约，不能按「给开发者写 API」的方式设计。
2. 有效路径：原型 → **工具评测** → 与 Agent 协作迭代工具描述/实现。
3. 原则：选对工具（而非越多越好）、命名空间划界、返回有意义上下文、为 token 效率优化响应、Prompt-engineer 工具描述。

## Mechanism（可验证的机制）

```text
Prototype tools (local MCP / API)
        ↓
Eval tasks（真实复杂度，多步工具）
        ↓
Agent loop per task + metrics
  (准确率 / 工具次数 / token / 错误类型)
        ↓
读 transcript + CoT / feedback → 改 schema/描述/合并工具
        ↓
Held-out 防过拟合
```

强评测任务特征：多工具、真实数据源、可验证结果；弱任务：单次搜索/单参数调度。

指标除顶层正确率外：单工具时延、总工具次数、token、错误码分布（冗余调用 → 分页/限额；非法参数 → 描述/示例不足）。

## Evidence

- 内部 Slack / Asana 工具 held-out 提升曲线（文中图示；具体数字随内部集变化，不抄绝对值作真理）。
- Web search 案例：模型无意义追加 `2025` 到 query → 靠改工具描述纠正。
- 强调 held-out，避免用「训练评测」过拟合描述。

## Engineering translation（我们怎么用）

| 原则 | 本手册落点 |
|------|------------|
| 工具 = Agent 契约 | [tool-use](../../../core/tool-use/tool-use.md) schema / 错误编码 / 结果回灌 |
| 少而清晰的工具面 | Tool Use 路由与「工具爆炸」；[design-patterns](../../../production/design-patterns/design-patterns.md) 工具网关 |
| 响应 token 效率 | [context-engineering](../../../core/context-engineering/context-engineering.md) Observation 管道 |
| 描述即 Prompt | [prompt-engineering](../../../core/prompt-engineering/prompt-engineering.md) ACI |
| 评测驱动改工具 | [evaluation](../../../production/evaluation/evaluation.md)（工具正确性维度） |
| MCP 承载工具 | [mcp](../../../protocols/mcp/mcp.md)；协议正确 ≠ ACI 合格 |

**可执行下一步**：为每个生产工具集建「真实任务」评测包；改描述/合并工具必须过 held-out。

## Disagreement / Caveats

- 「用 Claude 优化工具」是加速手段，不能替代人类对权限与副作用的政策审查。
- 文中 MCP「数百工具」是能力面，不是推荐默认工具数；本仓库默认 **收敛工具面**。
- LLM-as-judge 验证器需金标标定（见 Evaluation Canonical）。

## Action items

1. 新建工具 PR 模板：职责边界、反例、示例、错误可行动性、token 预算。
2. Eval 任务禁止「单 API 包装」式玩具题。
3. 定期扫工具错误分布，驱动描述与参数 poka-yoke。

## References

- [article-anthropic-writing-tools-for-agents-2025](../../../references/articles.md#article-anthropic-writing-tools-for-agents-2025)
- 对照：[tool-use](../../../core/tool-use/tool-use.md) · [evaluation](../../../production/evaluation/evaluation.md)
