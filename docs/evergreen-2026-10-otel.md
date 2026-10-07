# Evergreen Review — OpenTelemetry GenAI Semantic Conventions（2026-10-07）

> 接续 [evergreen-2026-10-llm-api.md](./evergreen-2026-10-llm-api.md)。  
> 范围：OTel GenAI 语义约定的**机制与字段名**（model / tokens / tool）；不含厂商价目。  
> 回写：[observability.md](../production/observability/observability.md) §4.2。

## 官方源

| 条目 | URL |
|------|-----|
| [std-otel-genai-semantic-conventions](../references/docs.md#std-otel-genai-semantic-conventions) | github.com/open-telemetry/semantic-conventions-genai |
| 人类可读入口 | `docs/gen-ai/README.md`（Status: **Development**） |
| 属性登记 | `docs/registry/attributes/gen-ai.md` |
| 迁移说明（旧站） | https://opentelemetry.io/docs/specs/semconv/gen-ai/ （Moved） |

## 机制级结论

1. **仓库拆分**：GenAI 约定已离开核心 `semantic-conventions`；以 GenAI 仓库 `docs/` 为准，勿死记旧站镜像。  
2. **稳定性**：约定与 `gen_ai.*` 属性均为 **Development**；尚无 Stable 的 GenAI 专属属性。可叠用核心 **Stable** 键如 `error.type`、`server.address`。  
3. **操作名**：`gen_ai.operation.name` — Agent 相关预定义值含 `chat`、`invoke_agent`、`execute_tool`、`plan`、`embeddings`、`retrieval` 等。  
4. **模型**：`gen_ai.provider.name` + `gen_ai.request.model` / `gen_ai.response.model`；弃用 `gen_ai.system`。  
5. **Token**：`gen_ai.usage.input_tokens` / `output_tokens`（弃用 `prompt_tokens` / `completion_tokens`）；缓存用 `cache_read` / `cache_write` 的 `*.input_tokens`，且计入 input 总量；可选 `reasoning.output_tokens`。  
6. **工具**：`execute_tool` span — `gen_ai.tool.name` Required；`tool.call.id` / `tool.type` Recommended；`tool.call.arguments` / `result` 为 **Opt-In**（敏感）。  
7. **正文默认不上**：`input.messages` / `output.messages` / `system_instructions` / `tool.definitions` 均为 Opt-In；与手册隐私红线一致。  
8. **落地策略**：内部 schema 为真相；导出适配器映射 `gen_ai.*`；约定仍会变，勿在业务代码硬编码社区键。

## 故意不写入

- 各厂商 usage 字段与多模态/缓存**计价倍率** — 另标 **TODO: Verify**（价目变化快）。

## Canonical 同步

- [observability.md](../production/observability/observability.md) §4.2、Q25、§17–18  
- [references/docs.md](../references/docs.md) `#std-otel-genai-semantic-conventions`

## 下次建议

- 价目表数字专项（caching / vision / reasoning tokens）  
- GenAI 仓库若出现 Stable 属性或正式 schema URL，再钉版本复核  
- MCP Tasks / Apps 扩展可用性  
