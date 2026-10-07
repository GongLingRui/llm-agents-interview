# Building effective agents（Anthropic）

## Metadata

- Source / Author / Date: Anthropic Engineering · Erik S. & Barry Zhang · 2024-12-19
- URL: https://www.anthropic.com/engineering/building-effective-agents
- Mapped modules: agent-loop, agent-architecture, design-patterns, tool-use, multi-agent, coding-agent
- Last reviewed: 2026-10-07
- Reference: [article-anthropic-building-effective-agents-2024](../../../references/articles.md#article-anthropic-building-effective-agents-2024)

## Claim（作者主张）

1. 成功实现多用 **简单可组合模式**，而非复杂框架。
2. 区分 **workflows**（预定义代码路径编排 LLM/工具）与 **agents**（模型动态决定过程与工具）。
3. 默认找最简方案；Agentic 系统用延迟与成本换任务表现，应显式做权衡。
4. 构建 Agent 三原则：保持简单、规划步骤透明、认真做 **ACI（Agent-Computer Interface）**。

## Mechanism（可验证的机制）

```text
Augmented LLM（检索 / 工具 / 记忆）
        ↓
Workflows（固定图）          Agents（模型控环）
  · Prompt chaining             · LLM + tools in a loop
  · Routing                     · 环境观测作 ground truth
  · Parallelization             · 检查点 / max iterations
  · Orchestrator-workers
  · Evaluator-optimizer
```

| 模式 | 控制权在谁 | 何时适合 |
|------|------------|----------|
| Prompt chaining | 代码 + 门控 | 任务可干净拆成固定子步 |
| Routing | 分类器/LLM → 专用路径 | 输入类别清晰、分治收益大 |
| Parallelization | 代码聚合 | 可并行子任务，或投票提置信 |
| Orchestrator-workers | 编排 LLM 动态委派 | 子任务结构不可预知（如多文件改码） |
| Evaluator-optimizer | 生成↔评价环 | 有清晰评价标准且迭代可量化改进 |
| Agent loop | 模型选工具与下一步 | 步数不可硬编码、需环境反馈 |

附录 ACI：工具格式要贴近模型自然文本、减少格式开销（如绝对路径优于易错相对路径）；工具定义投入 ≈ HCI 投入。

## Evidence

- 文中案例域：客服（对话+行动+可测成功）、Coding Agent（测试可验证、SWE-bench 类任务）。
- 明确警告：框架易遮挡底层 prompt/response，诱使过度复杂化；建议先从 LLM API 直写理解后再用框架。

## Engineering translation（我们怎么用）

| 决策 | 映射到本手册 |
|------|----------------|
| Workflow vs Agent 选型 | [agent.md](../../../foundations/agent/agent.md) · [agent-architecture](../../../core/agent-architecture/agent-architecture.md) |
| 「LLM + tools in a loop」 | [agent-loop](../../../core/agent-loop/agent-loop.md) |
| Orchestrator-workers / 路由 | [design-patterns](../../../production/design-patterns/design-patterns.md)；默认仍单 Agent |
| ACI / 工具文档 | [tool-use](../../../core/tool-use/tool-use.md) · [prompt-engineering](../../../core/prompt-engineering/prompt-engineering.md) |
| 沙箱测 Agent | [sandbox](../../../runtime/sandbox/sandbox.md) |
| MCP 作为增强 LLM 的一种集成面 | [mcp](../../../protocols/mcp/mcp.md)（协议层，不是 Loop 替代品） |

**可执行下一步**：新项目先画「能否用单次 LLM + 检索/示例解决」；再 Workflow；最后才开开放 Loop，并配 max_steps + 评测。

## Disagreement / Caveats

- 文首已注明 2024-12 工具景观有变；Managed Agents 产品语义需单独 Verify，不把营销名当理论。
- 「简单」不等于「无控制面」：预算、鉴权、观测仍是生产硬要求（本仓库 Production 层）。
- Orchestrator-workers ≠ 默认上 Multi-Agent：先证明单 Agent + 工具面不够。

## Action items

1. 团队词汇统一：workflow / agent / augmented LLM。
2. 每个新工具写 ACI 检查表（描述、边界、反例、绝对路径等）。
3. 复杂度升级必须绑定 offline eval 增量，而不是框架功能清单。

## References

- [article-anthropic-building-effective-agents-2024](../../../references/articles.md#article-anthropic-building-effective-agents-2024)
- 对照 Canonical：[agent-loop](../../../core/agent-loop/agent-loop.md) · [design-patterns](../../../production/design-patterns/design-patterns.md)
