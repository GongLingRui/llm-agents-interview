# Case: Anthropic Coding Agent（SWE-bench 路径）

## Problem

从「代码补全」走到能基于 PR/Issue 描述在多文件仓库中自主改代码并通过测试；路径开放、需环境反馈，固定 Workflow 难以覆盖。

## Constraints

- 解可被自动化测试验证（相对客服类任务更可测）
- 多文件编辑，子任务结构随 Issue 变化
- 必须控制工具格式开销（diff vs 整文件重写、路径约定等 ACI）
- 人工评审仍需覆盖「是否符合更广系统要求」（测试绿 ≠ 可合并）

## Architecture（链 Canonical）

```text
Task (issue / failing tests)
  → Agent Loop（plan ↔ edit ↔ run tests）
  → Tools: read/edit/search/shell（Sandbox）
  → Observation: test logs / diffs
  → Terminate on tests pass or budget
```

| 组件 | 手册映射 |
|------|----------|
| Loop + 工具反馈 | [agent-loop](../../core/agent-loop/agent-loop.md) · [tool-use](../../core/tool-use/tool-use.md) |
| 多文件不确定子任务 | [design-patterns](../../production/design-patterns/design-patterns.md) Orchestrator-workers（可选）；默认仍可单 Agent |
| 执行隔离 | [sandbox](../../runtime/sandbox/sandbox.md) |
| 上下文不整仓塞入 | [context-engineering](../../core/context-engineering/context-engineering.md) · [coding-agent](../../agent-types/coding-agent/coding-agent.md) |
| 成功定义 | [evaluation](../../production/evaluation/evaluation.md)（测试/基准） |

来源文章将 Coding Agent 作为 Agent（非纯 Workflow）的典型域，并强调 ACI：例如强制绝对路径以避免相对路径漂移错误。

## What worked

- **可验证反馈环**：测试结果作 ground truth，支撑迭代修复
- **工具投入 ≥ 系统提示**：SWE-bench 路径上大量时间花在工具格式与描述
- **Poka-yoke 参数**：降低模型易错格式（路径、diff 头等）
- **沙箱评测**：高自主度下先隔离环境再扩大权限

## What failed / 风险

- 只看基准通过率，忽略风格/架构/安全回归
- 工具结果与仓库上下文爆炸 → 无 compaction/句柄策略则空转或截断丢关键日志
- 相对路径、错误 diff 格式导致无效动作循环
- 把「Agent 能改代码」当成可无人值守合并

## Metrics（注明来源）

| 指标 | 说明 | 来源 |
|------|------|------|
| SWE-bench Verified 类任务 | 文称其实现可基于 PR 描述解决真实 GitHub issues | [article-anthropic-building-effective-agents-2024](../../references/articles.md#article-anthropic-building-effective-agents-2024)（定性；**非**本手册复现数字） |
| 工具优化占比 | 「花在工具上的时间多于整体 prompt」 | 同上 Appendix 2（工程经验陈述） |

> 不把未核实的公开榜单分数写入本案例；落地用内部失败用例集 + 沙箱回归。

## Mapped modules

coding-agent · agent-loop · tool-use · sandbox · context-engineering · evaluation · design-patterns

## References

- [article-anthropic-building-effective-agents-2024](../../references/articles.md#article-anthropic-building-effective-agents-2024)
- 机制笔记：[../industry-practices/anthropic/building-effective-agents.md](../industry-practices/anthropic/building-effective-agents.md)
- 对照：[coding-agent.md](../../agent-types/coding-agent/coding-agent.md)
