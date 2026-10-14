# Evergreen Review — Structured Outputs / Caching / Seed（2026-10-07）

> 接续 [evergreen-2026-10-tool-docs.md](./evergreen-2026-10-tool-docs.md)。  
> 范围：LLM 结构化输出、前缀缓存、`seed` 可复现；登记新 references 并回写 Foundations / Prompt / Tool Use。

## 新增 references

| ID | URL |
|----|-----|
| [doc-openai-structured-outputs](../references/docs.md#doc-openai-structured-outputs) | developers.openai.com/.../structured-outputs |
| [doc-openai-prompt-caching](../references/docs.md#doc-openai-prompt-caching) | developers.openai.com/.../prompt-caching |
| [doc-openai-advanced-usage](../references/docs.md#doc-openai-advanced-usage) | developers.openai.com/.../advanced-usage |
| [doc-anthropic-structured-outputs](../references/docs.md#doc-anthropic-structured-outputs) | docs.anthropic.com/.../structured-outputs |
| [doc-anthropic-prompt-caching](../references/docs.md#doc-anthropic-prompt-caching) | docs.anthropic.com/.../prompt-caching |

## 机制级结论（可写入 Canonical）

1. **Structured Outputs ≠ JSON mode**：前者 schema 遵从；后者仅合法 JSON。OpenAI 推荐优先 Structured Outputs。  
2. **Tool `strict`**：OpenAI function tools 与 Anthropic custom tools 均可 `strict: true` 约束参数。  
3. **Prompt caching**：稳定前缀在前；OpenAI 自动/显式 breakpoint；Anthropic `cache_control`，序 tools→system→messages。  
4. **`seed`**：OpenAI = mostly deterministic + `system_fingerprint`；非比特级保证。

## TTL / 最低 token / Gemini seed（扩展）

> 本轮仅登记前缀策略与 OpenAI seed。**TTL、最低可缓存 token、Gemini seed** 的机制复核见后续交付 **[evergreen-2026-10-caching-seed.md](./evergreen-2026-10-caching-seed.md)**（2026-10-07）。

## 故意不写入的数字

- 缓存读写**倍率**、多模态价目 — 标价目 **TODO: Verify**（变化快）。  
- ~~精确 TTL / 最低 token~~ → 已迁至 [caching-seed](./evergreen-2026-10-caching-seed.md) 机制核验。

## Canonical 同步

- [llm.md](../foundations/llm/llm.md) §2/§3/§5 + self-review  
- [prompt-engineering.md](../core/prompt-engineering/prompt-engineering.md)  
- [tool-use.md](../core/tool-use/tool-use.md) self-review  
- [llm-inference.md](../runtime/llm-inference/llm-inference.md) / [context-engineering.md](../core/context-engineering/context-engineering.md) caching 指针  

## 其它本轮交付

- 仓库 [LICENSE](../LICENSE) 定为 **MIT**；根 README License 节已更新。

## 下次建议

- 价目表**倍率**专项（caching / vision / compaction）  
- MCP Host UX / SDK 落地矩阵  

## 后续已交付

- OTel GenAI conventions 字段落地 → [evergreen-2026-10-otel.md](./evergreen-2026-10-otel.md)  
- Streaming / strict · refusal · HITL · compaction 机制 → [evergreen-2026-10-agents-stream.md](./evergreen-2026-10-agents-stream.md)  
- Caching TTL / 最低 token · Gemini seed → [evergreen-2026-10-caching-seed.md](./evergreen-2026-10-caching-seed.md)  
