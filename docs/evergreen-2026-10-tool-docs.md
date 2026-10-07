# Evergreen Review — OpenAI / Anthropic 工具文档（2026-10-07）

> 范围：`references/docs.md` 中 OpenAI/Anthropic 工具与沙箱相关条目 + 依赖它们的 Canonical 硬事实。  
> 方法：抓取现行官方页，对照条目 Summary / Canonical 断言；不改机制理论，只刷新事实与 Verify 状态。

## 复核条目

| Ref ID | URL 可达 | Summary 需改？ | 结论 |
|--------|----------|----------------|------|
| [doc-openai-function-calling](../references/docs.md#doc-openai-function-calling) | ✅ | 小幅增强 | 契约未变；补充 custom tools / API 面分流 |
| [doc-openai-tools](../references/docs.md#doc-openai-tools) | ✅ | 是 | 钉死 `tool_search` ≥ `gpt-5.4`；提及 Programmatic Tool Calling |
| [doc-anthropic-tool-use](../references/docs.md#doc-anthropic-tool-use) | ✅ | 是 | 明确 server 工具举例与 Anthropic-schema client tools |
| [doc-anthropic-how-tool-use-works](../references/docs.md#doc-anthropic-how-tool-use-works) | ✅ | 是 | **三类执行位**；URL 去 `.md`；`pause_turn` |
| [doc-openai-code-interpreter](../references/docs.md#doc-openai-code-interpreter) | ✅ | 是 | 核实闲置 **20 min**；内存档 `1g/4g/16g/64g` |
| [doc-openai-sandbox-agents](../references/docs.md#doc-openai-sandbox-agents) | ✅ | 小幅 | harness vs execution 仍成立 |

未改机制立场：模型（对你的函数）不执行；执行在应用或托管运行时。

## Canonical 同步

| 文件 | 变更 |
|------|------|
| [tool-use.md](../core/tool-use/tool-use.md) | tool_search 支持面；Anthropic 三类执行位短注；Q1 追问补全 |
| [sandbox.md](../runtime/sandbox/sandbox.md) | Code Interpreter 闲置 20 min + 内存档写死为已核实 |

## 仍保持 TODO: Verify

- `strict` / streaming tool args **跨模型矩阵**（机制已落盘 → [agents-stream](./evergreen-2026-10-agents-stream.md)；矩阵仍随快照变）
- `tool_search` **计费与配额**
- Anthropic server 工具 **版本化 type 字符串**（如 `web_search_20260209`）与价目
- 托管 Sandbox Agents **快照语义细节**、microVM 冷启动数量级

## 下次建议

- 日历：≤90 天或重大模型发布后重跑本清单  
- 价目数字 / 跨模型矩阵继续周期复核  
- 后续机制轮 → [evergreen-2026-10-llm-api.md](./evergreen-2026-10-llm-api.md) · [evergreen-2026-10-agents-stream.md](./evergreen-2026-10-agents-stream.md)
