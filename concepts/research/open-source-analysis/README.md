# Open Source Project Analysis

> 职责：拆解代表性开源 Agent / 协议 / 推理系统，提取可迁移设计。  
> 规范：[standards/open-source-analysis.md](../../standards/open-source-analysis.md)

## 当前分析

| 项目 | 焦点 | 笔记 |
|------|------|------|
| vLLM | 推理数据面与 Agent 控制面边界 | [vllm/vllm.md](./vllm/vllm.md) |
| MCP | 互操作协议 vs Tool Use 机制 | [mcp/mcp-protocol.md](./mcp/mcp-protocol.md) |
| OpenAI Agents SDK | Loop / handoffs / guardrails / tracing 映射 | [openai-agents-sdk/openai-agents-sdk.md](./openai-agents-sdk/openai-agents-sdk.md) |

## 分析模板

见本仓采用：Metadata → Scope / Non-goals → Agent Loop 映射 → Tool surface → State / Memory / Context → Eval / Obs hooks → Reusable designs → Anti-patterns → Mapped handbook modules → References。

## 引用登记

开源仓库条目放 [`references/repos.md`](../../references/repos.md)，分析文只链 ID。
