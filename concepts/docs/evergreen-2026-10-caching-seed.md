# Evergreen Review — Prompt Caching TTL / Min Tokens · Gemini Seed（2026-10-07）

> 接续 [evergreen-2026-10-llm-api.md](./evergreen-2026-10-llm-api.md)（Structured Outputs / caching 前缀策略 / OpenAI seed）。  
> 范围：**机制数字**（TTL、最低可缓存 token、seed 语义）；**不编造价目倍率 / 美元价**。

## A — Gemini `seed`

| 来源 | 结论 |
|------|------|
| [doc-gemini-seed](../references/docs.md#doc-gemini-seed)（Content generation parameters） | 固定 `seed` → 模型对同 prompt/参数做 **best effort** 复现；**Deterministic output isn't guaranteed** |
| GenerationConfig REST | 表述为 **mostly** deterministic；**非**绝对确定性保证 |
| ai.google.dev `GenerationConfig.seed` | 字段存在；未设则随机 seed |

与 OpenAI（mostly + `system_fingerprint`）同属工程可复现预算：**绝对比特级一致非保证**。Anthropic Messages 公开 API 仍 **无 seed**（见 [evergreen-2026-10-agents-stream.md](./evergreen-2026-10-agents-stream.md)）。

## B — Prompt caching：TTL + 最低 token

### OpenAI（[doc-openai-prompt-caching](../references/docs.md#doc-openai-prompt-caching)）

| 机制 | 官方口径（2026-10-07） |
|------|------------------------|
| TTL | `prompt_cache_options.ttl` **唯一支持且默认 `30m`**；最近一次 write/reuse 后至少 30 分钟可复用（可能更长） |
| 最低可缓存前缀 | GPT-5.6+：**1024** visible input tokens（不含 hidden system）；更早模型随 tools/images/schema 等请求设置而变 |
| 计费倍率 | **TODO: Verify**（本轮不抄） |

注：更早模型另有 `prompt_cache_retention`（`in_memory` / `24h`）语义，与 GPT-5.6+ 的 `ttl` 控制面不同——实现时钉模型代际。

### Anthropic（[doc-anthropic-prompt-caching](../references/docs.md#doc-anthropic-prompt-caching)）

| 机制 | 官方口径（2026-10-07） |
|------|------------------------|
| 默认 TTL | **5 分钟**（`ephemeral`）；命中可刷新 |
| 可选 TTL | **`ttl: "1h"`** |
| 最低可缓存长度 | 按模型：**512 / 1024 / 2048 / 4096** tokens（官方表；未达则静默不缓存、不报错） |
| 计费倍率 | **TODO: Verify**（本轮不抄） |

## 故意不写入

- 缓存读写倍率、美元价目、多模态折算价  
- 完整「型号 × 最低 token」逐行抄表（以官方页为准；Canonical 只记档位）

## Canonical / references 同步

- [llm.md](../foundations/llm/llm.md) §2 缓存机制；§5 seed（含 Gemini）；Self-review  
- [llm-inference.md](../runtime/llm-inference/llm-inference.md) §5.4  
- [prompt-engineering.md](../core/prompt-engineering/prompt-engineering.md) §7.3  
- [references/docs.md](../references/docs.md)：`doc-openai-prompt-caching` / `doc-anthropic-prompt-caching` Summary；新增 `doc-gemini-seed`

## 下次建议

- caching / vision / compaction **计费倍率**专项  
- 其它厂商 seed（若有）  
