# Document & Spec References

> ID 前缀：`doc-` / `std-`  
> **Evergreen**：工具文档 → [evergreen-2026-10-tool-docs.md](../docs/evergreen-2026-10-tool-docs.md)；LLM API → [evergreen-2026-10-llm-api.md](../docs/evergreen-2026-10-llm-api.md)；caching TTL / seed → [evergreen-2026-10-caching-seed.md](../docs/evergreen-2026-10-caching-seed.md)；streaming / HITL / compaction → [evergreen-2026-10-agents-stream.md](../docs/evergreen-2026-10-agents-stream.md)；MCP → [evergreen-2026-10-mcp.md](../docs/evergreen-2026-10-mcp.md)；OpenTelemetry → [evergreen-2026-10-otel.md](../docs/evergreen-2026-10-otel.md)。

### doc-openai-function-calling
- Title: Function calling
- Type: doc
- Authors/Org: OpenAI
- Year: 2026
- URL: https://developers.openai.com/api/docs/guides/function-calling
- Topics: core/tool-use, core/agent-loop
- Summary: Function/tool calling：应用声明 tools（JSON Schema 函数工具 + free-form custom tools），模型产出 tool call，应用侧执行并回传 tool output；另有平台内置工具。新模型可能要求 Responses API；大量工具时可与 tool_search 联用。
- Added: 2026-10-07
- Last checked: 2026-10-07
- Evergreen notes (2026-10-07): 核心契约未变（模型不执行你的函数）。文档强调 function tools vs custom tools vs built-in tools；`strict` 仍出现在函数 schema 示例中。Agents API / Responses / Chat Completions 集成路径分流——实现时钉 API 面，勿混抄示例。

### doc-openai-tools
- Title: Using tools
- Type: doc
- Authors/Org: OpenAI
- Year: 2026
- URL: https://developers.openai.com/api/docs/guides/tools
- Topics: core/tool-use
- Summary: 工具总览：内置工具、function calling、Programmatic Tool Calling、tool search、远程 MCP 等；能力随 runtime（Responses / Agents API / Agents SDK）而异。
- Added: 2026-10-07
- Last checked: 2026-10-07
- Evergreen notes (2026-10-07): 仍覆盖 remote MCP 与 tool search。官方写明 **仅 `gpt-5.4` 及以后**支持 `tool_search`。出现 Programmatic Tool Calling 作为扩展面——手册不绑死实现细节。

### doc-anthropic-tool-use
- Title: Tool use with Claude
- Type: doc
- Authors/Org: Anthropic
- Year: 2026
- URL: https://platform.claude.com/docs/en/agents-and-tools/tool-use/overview
- Topics: core/agent-loop, core/tool-use
- Summary: Tool use（亦称 function calling）：client tools 由应用执行并回传 `tool_result`；server tools（如 web_search / web_fetch / code_execution / tool_search）由 Anthropic 执行。另有 Anthropic-schema 的 client tools（bash、text_editor、computer、browser、memory 等）。
- Added: 2026-10-07
- Last checked: 2026-10-07
- Evergreen notes (2026-10-07): 总览仍强调 client vs server 执行位置。Server tool 列表示例含 `web_search`（带版本化 type 字符串，易变）。部分 server 工具可能有额外用量计费。

### doc-anthropic-how-tool-use-works
- Title: How tool use works
- Type: doc
- Authors/Org: Anthropic
- Year: 2026
- URL: https://platform.claude.com/docs/en/agents-and-tools/tool-use/how-tool-use-works
- Topics: core/tool-use
- Summary: 工具契约：模型只发结构化请求，不自行执行。三类执行位——(1) 用户定义 client tools；(2) Anthropic-schema client tools（应用执行、schema 由 Anthropic 提供）；(3) server-executed tools。Client 路径用 `stop_reason=tool_use` 驱动应用侧循环；server 路径可有服务端内环与 `pause_turn`。
- Added: 2026-10-07
- Last checked: 2026-10-07
- Evergreen notes (2026-10-07): URL 去掉历史 `.md` 后缀（现行 docs 路由）。三类分桶比「仅 client/server」更精确；并行混合 client+server 时可能先把控制交回应用。Anthropic-schema 示例：`memory`、`bash`、`text_editor`、`computer`、`browser`。Server 示例：`web_search`、`web_fetch`、`code_execution`、`tool_search`。

### std-json-schema-2020-12
- Title: JSON Schema Specification (2020-12)
- Type: std
- Authors/Org: JSON Schema
- Year: 2022
- URL: https://json-schema.org/specification
- Topics: core/tool-use
- Summary: JSON Schema 现行草稿 2020-12；工具参数 schema 的工业标准基础。
- Added: 2026-10-07
- Last checked: 2026-10-07

### std-mcp-specification
- Title: Model Context Protocol Specification
- Type: std
- Authors/Org: Model Context Protocol
- Year: 2025–2026
- URL: https://modelcontextprotocol.io/specification/
- Topics: protocols/mcp, core/tool-use
- Summary: MCP 开放协议规范（schema.ts 为真相源）。标准传输：stdio 与 Streamable HTTP；HTTP+SSE（2024-11-05）已由 Streamable HTTP 取代并提供兼容指引。Client features：`sampling/createMessage`、`elicitation/create`（form/url）；Utilities：`notifications/cancelled`；Tasks（`2025-11-25` experimental：`tasks/get|result|list|cancel`）。官方扩展另见 Skills / MCP Apps 条目。
- Added: 2026-10-07
- Last checked: 2026-10-07
- Notes: 版本锚点 `2025-11-25`。传输 https://modelcontextprotocol.io/specification/2025-11-25/basic/transports ；Sampling https://modelcontextprotocol.io/specification/2025-11-25/client/sampling ；Elicitation https://modelcontextprotocol.io/specification/2025-11-25/client/elicitation ；Cancellation https://modelcontextprotocol.io/specification/2025-11-25/basic/utilities/cancellation ；Tasks https://modelcontextprotocol.io/specification/2025-11-25/basic/utilities/tasks
- Evergreen notes (2026-10-07): 机制方法名与 capability 已核实；Host 默认开关 / SDK 落地矩阵仍属产品层 Verify。

### doc-mcp-skills-extension
- Title: Skills (MCP Extension)
- Type: doc
- Authors/Org: Model Context Protocol / Skills Over MCP Working Group
- Year: 2026
- URL: https://modelcontextprotocol.io/extensions/skills/overview
- Topics: protocols/mcp, core/tool-use
- Summary: 官方扩展 `io.modelcontextprotocol/skills`（SEP-2640 Final）。Server 须声明 `resources` + 该扩展；实现 `skills/list`、`skills/get`；内容经 `resources/read`；可选 `resources/directory/read`（`directoryRead: true`）。Skill 格式委派 Agent Skills 规格；Host 决定如何加载进上下文。SDK/Host 支持仍在落地。
- Added: 2026-10-07
- Last checked: 2026-10-07
- Notes: 完整规格 https://skills.extensions.modelcontextprotocol.io/ ；SEP https://modelcontextprotocol.io/seps/2640-skills-extension

### doc-mcp-apps-extension
- Title: MCP Apps (Extension)
- Type: doc
- Authors/Org: Model Context Protocol
- Year: 2026
- URL: https://modelcontextprotocol.io/extensions/apps/overview
- Topics: protocols/mcp
- Summary: 可选扩展：工具经 `_meta.ui.resourceUri` 指向 `ui://` HTML Resource；Host 在沙箱 iframe 渲染；App↔Host 使用 `ui/` 前缀 JSON-RPC（如 `ui/initialize`），多为 postMessage。Host 支持因客户端而异。
- Added: 2026-10-07
- Last checked: 2026-10-07
- Notes: 构建指南 https://modelcontextprotocol.io/extensions/apps/build ；API/SDK https://apps.extensions.modelcontextprotocol.io/

### doc-openai-code-interpreter
- Title: Code Interpreter
- Type: doc
- Authors/Org: OpenAI
- Year: 2026
- URL: https://developers.openai.com/api/docs/guides/tools-code-interpreter
- Topics: runtime/sandbox, core/tool-use
- Summary: Responses API 的 `code_interpreter` 工具：在沙箱容器中跑 Python（模型侧常称 “python tool”）。容器 auto/显式创建；内存档 `1g`（默认）/`4g`/`16g`/`64g`；闲置约 20 分钟过期且数据不可恢复，产物须主动导出。
- Added: 2026-10-07
- Last checked: 2026-10-07
- Evergreen notes (2026-10-07): 闲置 **20 minutes** 过期仍成立；更高内存档按内置工具价目计费（本手册不抄绝对价格）。示例模型名随文档演示变更——勿当唯一支持矩阵。

### doc-openai-sandbox-agents
- Title: Sandbox Agents
- Type: doc
- Authors/Org: OpenAI
- Year: 2026
- URL: https://developers.openai.com/api/docs/guides/agents/sandboxes
- Topics: runtime/sandbox, core/agent-loop, core/tool-use
- Summary: Agents SDK 沙箱：区分 harness（控制面：loop/工具路由/审批/tracing）与 sandbox execution plane（FS、shell、端口、快照、隔离）。覆盖 Manifest、挂载、provider 选型与 resume。
- Added: 2026-10-07
- Last checked: 2026-10-07
- Evergreen notes (2026-10-07): harness vs compute 切分仍是核心。偶发 shell 建议先用 hosted shell tool；需要工作区隔离/快照/多 provider 再用 SandboxAgent。托管 harness 另见 Agents API 文档分流。

### doc-openai-structured-outputs
- Title: Structured model outputs
- Type: doc
- Authors/Org: OpenAI
- Year: 2026
- URL: https://developers.openai.com/api/docs/guides/structured-outputs
- Topics: foundations/llm, core/prompt-engineering, core/tool-use
- Summary: Structured Outputs 保证响应遵从所供 JSON Schema；相对 JSON mode（仅合法 JSON）。Chat Completions 用 `response_format.json_schema`+`strict`；Responses 用 `text.format`。支持模型自 gpt-4o-2024-08-06 / gpt-4o-mini 起；refusal 可程序化检测（`refusal` 字段 / content part）。
- Added: 2026-10-07
- Last checked: 2026-10-07
- Evergreen notes (2026-10-07): refusal 与 incomplete（如 max tokens）路径机制已核；跨模型矩阵仍 Verify。

### doc-openai-prompt-caching
- Title: Prompt caching
- Type: doc
- Authors/Org: OpenAI
- Year: 2026
- URL: https://developers.openai.com/api/docs/guides/prompt-caching
- Topics: foundations/llm, core/context-engineering, runtime/llm-inference
- Summary: 对够长且精确匹配的前缀自动缓存以降延迟/成本；稳定内容置前。较新模型支持 `prompt_cache_options`（implicit/explicit）与 breakpoint。机制（Evergreen 2026-10-07）：`prompt_cache_options.ttl` 唯一支持且默认值为 `30m`（最近一次 write/reuse 后至少 30 分钟可复用，可能更长）；GPT-5.6+ 最低可缓存前缀 **1024** visible input tokens（不含 hidden system）；更早模型最低长度随请求设置而变。计费倍率仍 Verify。
- Added: 2026-10-07
- Last checked: 2026-10-07
- Evergreen notes (2026-10-07): TTL/最低 token 机制已核；读写价目倍率不抄入 Canonical。扩展见 [docs/evergreen-2026-10-caching-seed.md](../docs/evergreen-2026-10-caching-seed.md)。

### doc-openai-advanced-usage
- Title: Advanced usage
- Type: doc
- Authors/Org: OpenAI
- Year: 2026
- URL: https://developers.openai.com/api/docs/guides/advanced-usage
- Topics: foundations/llm
- Summary: `seed` + 固定参数追求 mostly 可复现；用 `system_fingerprint` 监测后端变更；不保证绝对确定性。
- Added: 2026-10-07
- Last checked: 2026-10-07

### doc-gemini-seed
- Title: Content generation parameters — Seed (Gemini)
- Type: doc
- Authors/Org: Google
- Year: 2026
- URL: https://docs.cloud.google.com/gemini-enterprise-agent-platform/models/capabilities/content-generation-parameters
- Topics: foundations/llm
- Summary: `GenerationConfig.seed` 固定后模型对同 prompt/参数做 **best effort** 复现；文档明确 **Deterministic output isn't guaranteed**。改模型或其它采样参数即使同 seed 也可变；默认随机 seed。API 字段亦见 ai.google.dev `generateContent` / `GenerationConfig.seed`。
- Added: 2026-10-07
- Last checked: 2026-10-07
- Notes: REST 字段说明（mostly deterministic / not absolute）https://docs.cloud.google.com/gemini-enterprise-agent-platform/reference/rest/v1beta1/GenerationConfig ；Gemini API https://ai.google.dev/api/generate-content#GenerationConfig
- Evergreen notes (2026-10-07): 与 OpenAI 同属 mostly/best-effort；绝对比特级一致非保证。见 [docs/evergreen-2026-10-caching-seed.md](../docs/evergreen-2026-10-caching-seed.md)。

### doc-anthropic-structured-outputs
- Title: Structured outputs
- Type: doc
- Authors/Org: Anthropic
- Year: 2026
- URL: https://docs.anthropic.com/en/docs/build-with-claude/structured-outputs
- Topics: foundations/llm, core/prompt-engineering, core/tool-use
- Summary: JSON outputs（`output_config.format` / json_schema）与 strict tool use（`strict: true`）可独立或同用；保证可解析结构。旧 `output_format` beta 字段处于过渡期。拒绝时 `stop_reason: "refusal"`（200，计费）；`max_tokens` 可导致半截非 schema 输出。
- Added: 2026-10-07
- Last checked: 2026-10-07
- Evergreen notes (2026-10-07): refusal / max_tokens 非 schema 路径已写入 Canonical；完整模型矩阵仍 Verify。

### doc-anthropic-prompt-caching
- Title: Prompt caching
- Type: doc
- Authors/Org: Anthropic
- Year: 2026
- URL: https://docs.anthropic.com/en/docs/build-with-claude/prompt-caching
- Topics: foundations/llm, core/context-engineering, runtime/llm-inference
- Summary: 用 `cache_control`（`type: ephemeral`）标记可复用前缀；缓存构造顺序 tools → system → messages。机制（Evergreen 2026-10-07）：默认 TTL **5 分钟**（命中刷新）；可选 `ttl: "1h"`。最低可缓存长度按模型：512 / 1024 / 2048 / 4096 tokens 不等（官方按型号表列；未达阈值则不缓存、不报错）。计费倍率仍 Verify。
- Added: 2026-10-07
- Last checked: 2026-10-07
- Evergreen notes (2026-10-07): 默认 5m、可选 1h、按模型最低 token 已核；读写价目倍率不抄入 Canonical。扩展见 [docs/evergreen-2026-10-caching-seed.md](../docs/evergreen-2026-10-caching-seed.md)。

### doc-openai-streaming
- Title: Streaming API responses
- Type: doc
- Authors/Org: OpenAI
- Year: 2026
- URL: https://developers.openai.com/api/docs/guides/streaming-responses
- Topics: core/tool-use, foundations/llm
- Summary: Responses API 语义化 SSE 事件（含 `response.function_call_arguments.delta` / `.done`、`response.refusal.delta` 等）；Chat Completions 用 delta chunk。工具参数流式细节亦见 Function calling 指南。
- Added: 2026-10-07
- Last checked: 2026-10-07
- Evergreen notes (2026-10-07): 与 function-calling 页交叉引用；执行仍应以 arguments done / 完整聚合为准。

### doc-anthropic-streaming
- Title: Streaming Messages
- Type: doc
- Authors/Org: Anthropic
- Year: 2026
- URL: https://platform.claude.com/docs/en/build-with-claude/streaming
- Topics: core/tool-use, foundations/llm
- Summary: Messages 流式；`tool_use` 的 input 以 `input_json_delta.partial_json` 分片到达，最终 `tool_use.input` 为对象；宜在 `content_block_stop` 后解析。
- Added: 2026-10-07
- Last checked: 2026-10-07

### doc-anthropic-strict-tool-use
- Title: Strict tool use
- Type: doc
- Authors/Org: Anthropic
- Year: 2026
- URL: https://platform.claude.com/docs/en/agents-and-tools/tool-use/strict-tool-use
- Topics: core/tool-use
- Summary: 工具定义 `strict: true` 以 grammar-constrained sampling 保证 input 贴合 schema；与 Structured Outputs 同源管线。部分 computer/browser toolset 条目拒绝 `strict`。
- Added: 2026-10-07
- Last checked: 2026-10-07

### doc-anthropic-fine-grained-tool-streaming
- Title: Fine-grained tool streaming
- Type: doc
- Authors/Org: Anthropic
- Year: 2026
- URL: https://platform.claude.com/docs/en/agents-and-tools/tool-use/fine-grained-tool-streaming
- Topics: core/tool-use
- Summary: 工具级 `eager_input_streaming`：跳过服务端缓冲/校验以降首包延迟；累积串可能非法 JSON，解析须 guard；`max_tokens` 中断需重试/修复。
- Added: 2026-10-07
- Last checked: 2026-10-07

### doc-openai-agents-hitl
- Title: Human-in-the-loop (OpenAI Agents SDK)
- Type: doc
- Authors/Org: OpenAI
- Year: 2026
- URL: https://openai.github.io/openai-agents-python/human_in_the_loop/
- Topics: foundations/agent, core/agent-loop
- Summary: `needs_approval` 等声明敏感工具；暂停后 `interruptions` + `RunState.approve`/`reject`；用原顶层 agent + state 恢复。嵌套 `as_tool` / handoff 审批仍浮到外层 run。
- Added: 2026-10-07
- Last checked: 2026-10-07

### doc-openai-agents-guardrails
- Title: Guardrails and human review
- Type: doc
- Authors/Org: OpenAI
- Year: 2026
- URL: https://developers.openai.com/api/docs/guides/agents/guardrails-approvals
- Topics: foundations/agent, production/security
- Summary: Guardrails（输入/输出/工具自动校验）与 HITL 审批暂停的分工；审批是暂停的同一 run，不是新用户回合。
- Added: 2026-10-07
- Last checked: 2026-10-07

### doc-anthropic-managed-agents-permissions
- Title: Permission policies (Claude Managed Agents)
- Type: doc
- Authors/Org: Anthropic
- Year: 2026
- URL: https://platform.claude.com/docs/en/managed-agents/permission-policies
- Topics: foundations/agent
- Summary: `always_allow` / `always_ask` / `auto`；agent toolset 默认 allow，MCP toolset 默认 ask；`auto` 不是人工检查点。always_ask 时会话暂停等待确认。
- Added: 2026-10-07
- Last checked: 2026-10-07

### doc-openai-compaction
- Title: Compaction
- Type: doc
- Authors/Org: OpenAI
- Year: 2026
- URL: https://developers.openai.com/api/docs/guides/compaction
- Topics: core/context-engineering
- Summary: Responses 服务端 compaction（`context_management` + `compact_threshold`）与独立 `/responses/compact`；返回加密 compaction item，opaque；compact 输出应原样作为下一窗。
- Added: 2026-10-07
- Last checked: 2026-10-07

### doc-anthropic-compaction
- Title: Compaction
- Type: doc
- Authors/Org: Anthropic
- Year: 2026
- URL: https://docs.anthropic.com/en/docs/build-with-claude/compaction
- Topics: core/context-engineering
- Summary: `compact_20260112` 策略；input_tokens trigger（文档默认示例 150k，下限 50k）；产出 `compaction` block；回传后忽略块前内容；可选 `pause_after_compaction` / `instructions`。
- Added: 2026-10-07
- Last checked: 2026-10-07

### doc-anthropic-context-editing
- Title: Context editing
- Type: doc
- Authors/Org: Anthropic
- Year: 2026
- URL: https://docs.anthropic.com/en/docs/build-with-claude/context-editing
- Topics: core/context-engineering
- Summary: 服务端 edits：`clear_tool_uses_20250919`（清旧 tool result，可配 keep / clear_tool_inputs）与 thinking clearing；与 client-side / compact 策略并列说明。
- Added: 2026-10-07
- Last checked: 2026-10-07

### std-otel-genai-semantic-conventions
- Title: OpenTelemetry GenAI Semantic Conventions
- Type: std
- Authors/Org: OpenTelemetry
- Year: 2026
- URL: https://github.com/open-telemetry/semantic-conventions-genai
- Topics: production/observability
- Summary: GenAI（LLM/agent/embeddings/retrieval）与 MCP 的 span/metrics/events 语义约定；从核心 semantic-conventions 拆出。人类可读文档在 `docs/gen-ai/`，属性登记在 `docs/registry/attributes/gen-ai.md`。
- Added: 2026-10-07
- Last checked: 2026-10-07
- Evergreen notes (2026-10-07): 文档状态 **Development**；全部 `gen_ai.*` 属性均为 Development。原 https://opentelemetry.io/docs/specs/semconv/gen-ai/ 已迁出，仅保留 Moved 说明。Agent 轨迹关键键：`gen_ai.operation.name`、`gen_ai.provider.name`、`gen_ai.request.model` / `gen_ai.response.model`、`gen_ai.usage.input_tokens` / `output_tokens`（及 cache_read/cache_write）、`gen_ai.tool.name` / `tool.call.id`；消息与 tool args/result 为 Opt-In。弃用：`gen_ai.system`→`provider.name`，`prompt_tokens`/`completion_tokens`→`input_tokens`/`output_tokens`。核心侧 `error.type` 等可为 Stable。复核见 [docs/evergreen-2026-10-otel.md](../docs/evergreen-2026-10-otel.md)。
