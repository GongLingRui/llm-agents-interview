# Prompt Caching In Agents（Earendil / Pi）

## Metadata

- Source / Author / Date: Earendil Engineering · 2026-07-22  
- URL: https://earendil.com/posts/prompt-caching/  
- 中文整理：小墨同学 · https://x.com/xiaomovps/status/2107296424291701131（CC BY 4.0 适配；歧义以英文原文为准）  
- Mapped modules: llm-inference, context-engineering, tool-use, coding-agent, observability  
- Last reviewed: 2026-10-08  
- Reference: [article-earendil-prompt-caching-in-agents-2026](../../../references/articles.md#article-earendil-prompt-caching-in-agents-2026)

## Claim（作者主张）

1. Coding Agent 的输入**绝大部分与上一轮相同**（末尾追加）；提示缓存让这种工作方式在经济上勉强可行，但**十分脆弱**。  
2. 缓存行为不是边角优化——它塑造**延迟、成本、工具加载、会话/分支设计与产品功能**。  
3. **会话 ID ≠ 可复用前缀**；决定命中的是 token 前缀同一性 + 服务端路由/TTL。  
4. 客户端应优先 **稳定、以追加为主的转录**；剪枝/压缩省 token，但会改前缀并产生一次性重算成本。  
5. 缓存健康必须**可观测**（读/写/命中率/显著未命中），不能只靠账单事后推断。

## Mechanism（可验证的机制）

```text
Prefill：为输入算注意力 KV
Decode：逐步生成
跨请求 Prompt Cache：相同 token 前缀 → 复用已有 KV，只 Prefill 新后缀
```

| 概念 | 机制要点 | 本仓库对齐 |
|------|----------|------------|
| KV / 前缀 | 匹配是 **token 级连续前缀**；中间改一处，其后全部失效 | [llm-inference §5](../../../runtime/llm-inference/llm-inference.md) |
| 存放 | **会话亲和**（回原 GPU）vs **分布式 KV 块** | Inference 调度；客户端不可单独决定 |
| API 形态 | Anthropic 显式 `cache_control`；OpenAI 等自动前缀缓存（+ 可选 key/breakpoint） | [doc-openai-prompt-caching](../../../references/docs.md#doc-openai-prompt-caching) · [doc-anthropic-prompt-caching](../../../references/docs.md#doc-anthropic-prompt-caching) |
| 工具在前缀前 | 增删改/重排工具 schema ≈ 在提示早期打断前缀 → 旧对话整段落空 | [tool-use §9](../../../core/tool-use/tool-use.md) |
| 增量工具 | 支持时：新工具挂到**消息位置**（deferred / tool_reference / tool_search），不改最初工具列表 | Tool Use 动态工具；厂商能力矩阵仍 Verify |
| 会话树 | 同 session 多分支共享部分前缀；fork 新 ID 可能带相同内容却因路由 key 隔离而打不中 | Context / Coding Agent 会话模型 |
| TTL | 会话「还开着」≠ 缓存还在；空闲（测跑/喝咖啡）可超短 TTL → 短「继续」按全量输入计费 | Evergreen TTL；价目 Verify |
| 剪枝 vs 缓存 | 中间删除改变前缀；一次性重写成本 ≈ 编辑点后保留 token ×（未缓存价 − 缓存读价）；每轮节省 ≈ 删掉 token × 缓存读价 | [context-engineering](../../../core/context-engineering/context-engineering.md) |
| Compaction | 有意重建前缀 → 应记为**缓存重置**，不是「神秘失败」 | Context compaction |
| 激励 | 用户与 GPU 运营商通常希望命中；转售/网关若按未缓存输入抽成，激励可能错位 → 更要可观测 | Observability / 成本归因 |

**常见未命中原因（排障清单）**：空闲超 TTL · 换模型/厂商 · 分支/回退/fork · 压缩或手改历史 · 工具/推理级别变化 · 动态系统提示（时间戳等）· Extension 改载荷 · 路由/淘汰。

## Evidence

- 文中以 Pi 产品行为为例：稳定会话 ID/缓存提示、显式断点、增量工具、`R`/`W`/`CH` 状态栏、`/session` 累计命中与「缓存重复计费」估算、可选 `showCacheMissNotices`。  
- 示例数字（如 10 万 token 历史、`/session` 费用）为**说明场景**，非本仓库实测；**计费倍率勿抄入 Canonical**。  
- Anthropic 默认短 TTL（文中强调 5 分钟量级）与编程空闲冲突——与本仓库 Evergreen「Anthropic 默认 5m / 可选 1h」一致；具体以官方文档为准。

## Engineering translation（我们怎么用）

| 决策 | 做法 |
|------|------|
| 前缀纪律 | system + 工具 schema + 政策置顶且版本化；禁止每轮时间戳/随机盐打头 |
| 工具加载 | 默认固定工具集；按需加载优先用**消息锚定增量**（厂商支持时）；避免中途重排/替换整表 |
| 转录策略 | 默认追加；接近窗口上限再 compaction；剪枝前做「重写成本 vs 后续节省」估算 |
| 会话/分支 | 产品上 trellis/回退时，预期命中率下降；必要时向用户解释「换前缀」 |
| 可观测 | 记录 cache_read / cache_write / hit_rate；显著未命中告警（空闲、换模可归因） |
| 网关 | 亲和路由与负载均衡权衡显式化；勿假设转售层利益与命中一致 |

## Disagreement / Caveats

- 文从 **Pi + 特定时期厂商行为** 出发；其他 Harness 细节（`PI_CACHE_RETENTION`、UI 字段）不是通用标准。  
- 「Claude Code 订阅侧 1h TTL」等产品对照为原文观察，**TODO: Verify** 当前产品政策。  
- 不把「缓存命中」当成扩大上下文窗口或答案复用；复用的是 **KV 计算状态**。  
- 压缩 ≠ 缓存；二者配合：压缩腾窗口，缓存降重复 Prefill。

## Action items

1. Canonical 交叉引用：Inference / Context / Tool Use / Coding Agent（本笔记落地时完成）。  
2. 自建 Agent：加 cache 用量指标与未命中告警（对齐 Observability）。  
3. 评测门禁：回归用例包含「改工具顺序 / 注入时间戳 / 空闲超 TTL」三类故意 miss。

## References

- [article-earendil-prompt-caching-in-agents-2026](../../../references/articles.md#article-earendil-prompt-caching-in-agents-2026)  
- [doc-openai-prompt-caching](../../../references/docs.md#doc-openai-prompt-caching) · [doc-anthropic-prompt-caching](../../../references/docs.md#doc-anthropic-prompt-caching)  
- [evergreen-2026-10-caching-seed.md](../../../docs/evergreen-2026-10-caching-seed.md)  
- 机制正文：[llm-inference](../../../runtime/llm-inference/llm-inference.md) · [context-engineering](../../../core/context-engineering/context-engineering.md) · [tool-use](../../../core/tool-use/tool-use.md)
