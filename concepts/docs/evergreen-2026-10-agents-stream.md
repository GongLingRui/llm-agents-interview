# Evergreen Review — Streaming / Strict · Refusal · HITL · Compaction（2026-10-07）

> 接续 [evergreen-2026-10-llm-api.md](./evergreen-2026-10-llm-api.md)。  
> 范围：高价值 Verify 项的 **机制级** 落盘；**不编造价目表**；跨模型矩阵 / 产品默认 UX 仍标 Verify。

## 新增 / 增补 references

| ID | URL |
|----|-----|
| [doc-openai-streaming](../references/docs.md#doc-openai-streaming) | developers.openai.com/.../streaming-responses |
| [doc-anthropic-streaming](../references/docs.md#doc-anthropic-streaming) | platform.claude.com/.../streaming |
| [doc-anthropic-strict-tool-use](../references/docs.md#doc-anthropic-strict-tool-use) | platform.claude.com/.../strict-tool-use |
| [doc-anthropic-fine-grained-tool-streaming](../references/docs.md#doc-anthropic-fine-grained-tool-streaming) | platform.claude.com/.../fine-grained-tool-streaming |
| [doc-openai-agents-hitl](../references/docs.md#doc-openai-agents-hitl) | openai.github.io/.../human_in_the_loop |
| [doc-openai-agents-guardrails](../references/docs.md#doc-openai-agents-guardrails) | developers.openai.com/.../guardrails-approvals |
| [doc-anthropic-managed-agents-permissions](../references/docs.md#doc-anthropic-managed-agents-permissions) | platform.claude.com/.../permission-policies |
| [doc-openai-compaction](../references/docs.md#doc-openai-compaction) | developers.openai.com/.../compaction |
| [doc-anthropic-compaction](../references/docs.md#doc-anthropic-compaction) | docs.anthropic.com/.../compaction |
| [doc-anthropic-context-editing](../references/docs.md#doc-anthropic-context-editing) | docs.anthropic.com/.../context-editing |

（Structured Outputs / Function calling 既有条目 Summary 已补 refusal / streaming 交叉说明。）

## 机制级结论（可写入 Canonical）

### A — Streaming tool args + strict

1. **OpenAI**：Responses 流式聚合 `function_call_arguments.delta` → **`done` 为权威完整 arguments**；Chat Completions 按 `tool_calls[].index` 累积。`strict: true` 借 Structured Outputs；Responses 省略时可能规范化或回落 non-strict。  
2. **Anthropic**：`input_json_delta.partial_json` 拼串；`content_block_stop` 后再 parse。`strict: true` = grammar 约束。`eager_input_streaming` 可未校验碎片。  
3. **不变契约**：半截 JSON **不执行**。

### B — Structured Outputs refusal / seed

1. **OpenAI refusal**：可检测 `refusal`（及 Responses refusal part / 流式 delta）；incomplete（如 max tokens）亦可能非 schema。  
2. **Anthropic**：`stop_reason: "refusal"`（200、计费）；`max_tokens` 可半截。  
3. **seed**：OpenAI = mostly + `system_fingerprint`；**Anthropic Messages 公开 API 无 seed**；**Gemini = best-effort/mostly**（见 [caching-seed](./evergreen-2026-10-caching-seed.md)）；其它厂商继续 Verify。

### C — Managed Agents HITL

1. **OpenAI Agents SDK**：`needs_approval` → `interruptions` → `RunState` approve/reject → **同 run 恢复**。  
2. **Anthropic Managed Agents**：`permission_policy` allow/ask/auto；文档默认 **agent toolset allow、MCP ask**；`auto` ≠ 人工门。  
3. **产品开箱 UX / 默认工具矩阵**：仍 Verify。

### D — Agent README

- 前置 LLM 链接去掉「若尚未成文」措辞（`llm.md` 已存在）。

### E — Compaction / clearing

1. **OpenAI**：`context_management` + `compact_threshold`，或 `/responses/compact`；加密 compaction item；compact 输出勿手裁。  
2. **Anthropic**：`compact_20260112`；`clear_tool_uses_20250919` 清旧 tool result。  
3. **价目与精确默认阈值/模型矩阵**：仍 Verify。

## 故意不写入的数字

- caching / compaction / vision / tool_search **价目与配额**  
- 示例中的具体 token 阈值仅作文档量级提示，不固化为价目真理  

## Canonical 同步

| 文件 | 变更 |
|------|------|
| [tool-use.md](../core/tool-use/tool-use.md) | §3.2 strict；§8.3 streaming；self-review |
| [llm.md](../foundations/llm/llm.md) | refusal 路径；Anthropic 无 seed；self-review |
| [agent.md](../foundations/agent/agent.md) | §5.2 HITL 产品原语；self-review |
| [agent/README.md](../foundations/agent/README.md) | 前置措辞 |
| [context-engineering.md](../core/context-engineering/context-engineering.md) | §6 compaction/clearing 机制 |

## 仍保持 TODO: Verify

- streaming / strict / Structured Outputs **完整模型矩阵**  
- Managed Agents **产品默认 UX**  
- compaction / caching / 多模态 **价目**  
- 其它非 OpenAI/Anthropic/Gemini 厂商 seed（Anthropic 无 seed；Gemini best-effort 见 [caching-seed](./evergreen-2026-10-caching-seed.md)）  

## 下次建议

- 价目表数字专项（caching / vision / compaction）  
- 完整模型矩阵与 Managed 产品默认 UX  
- MCP Host UX / SDK 落地（机制见 [evergreen-2026-10-mcp.md](./evergreen-2026-10-mcp.md)）  
