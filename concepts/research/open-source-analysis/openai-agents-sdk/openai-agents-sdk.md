# OSS: OpenAI Agents SDK（Python）

## Metadata

- Repo / License / Language: [repo-openai-agents-python](../../../references/repos.md#repo-openai-agents-python) · License 见上游仓库 · Python
- Last reviewed: 2026-10-07
- Canonical modules mapped: agent-loop, multi-agent, tool-use, security, observability, mcp

## Scope / Non-goals

**范围**：以公开仓库/文档为据，把 Agents SDK 的 **Agent、Runner、tools、handoffs、guardrails、tracing** 映射到本手册词汇；标明 implementation example。

**非目标**：

- SDK API 教程或版本绑死的代码抄写
- 把 handoffs 默认成「应该上 Multi-Agent」
- 替代 [agent-loop](../../../core/agent-loop/agent-loop.md) / [multi-agent](../../../agent-types/multi-agent/multi-agent.md) Canonical

**核实基线**：[repo-openai-agents-python](../../../references/repos.md#repo-openai-agents-python) · 文档：Orchestration/Handoffs、Guardrails、Observability（developers.openai.com/api/docs/guides/agents/*）

## Agent Loop 映射

| 手册概念 | SDK 近似 |
|----------|----------|
| Agent Loop / Runner | Runner 驱动模型↔工具轮次直至终止 |
| Tool Use | function tools / hosted tools / MCP 接入 |
| Multi-Agent | handoffs（移交会话所有权）vs agents-as-tools（经理保留终答） |
| HITL | approval interruptions + resume state |
| Observability | 内置 tracing（model/tool/handoff/guardrail spans） |
| Security 政策 | input/output/tool guardrails |

官方建议：**能单 Agent 就单 Agent**；仅当能力隔离、政策隔离、提示清晰或 trace 可读性实质提升时再拆专家。

## Tool surface

- 工具仍是「模型产出调用意图 → 运行时执行」。
- Manager 模式：`agent.asTool()` 把专家当有界能力，避免过早移交用户对话所有权。
- MCP：hosted vs 本地/私有连接，信任与审批边界不同。

## State / Memory / Context

- SDK 管理 run state / 可恢复 interruptions；**不等于**长期 Memory 产品。
- 长程：需自管 compaction / 外置笔记（见 Context / OpenAI Skills+Compaction 文）。
- Handoff 时可过滤历史——属于 Context 策展，不是免费「全量共享大脑」。

## Eval / Obs hooks

- Trace 先于 Eval：先看清 run，再固化评分。
- Guardrail 失败、approval 拒绝应进失败分类。
- 勿把「有 dashboard」当成任务成功率。

## Reusable designs

1. **Handoff vs asTool** 决策表（所有权是否转移）
2. **Guardrail 分层**：输入/输出/工具旁路；高危副作用用人审
3. **默认单 Agent** 与本仓库 Multi-Agent 立场一致
4. Trace 字段对齐 `agent.step` / `agent.tool` 语义

## Anti-patterns

- 为展示架构过早拆多个 Agent
- 只靠 agent-level guardrail、工具副作用旁路无校验
- 把 SDK 类名写进理论定义
- 无 resume 策略的长跑 HITL

## Mapped handbook modules

- [agent-loop](../../../core/agent-loop/agent-loop.md)
- [multi-agent](../../../agent-types/multi-agent/multi-agent.md)
- [tool-use](../../../core/tool-use/tool-use.md)
- [security](../../../production/security/security.md)
- [observability](../../../production/observability/observability.md)
- [mcp](../../../protocols/mcp/mcp.md)

## References

- [repo-openai-agents-python](../../../references/repos.md#repo-openai-agents-python)
- [article-openai-skills-shell-compaction-2026](../../../references/articles.md#article-openai-skills-shell-compaction-2026)
- OpenAI Docs: Orchestration and handoffs；Guardrails and human review；Integrations and observability
