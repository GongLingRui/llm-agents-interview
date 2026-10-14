# Effective context engineering for AI agents（Anthropic）

## Metadata

- Source / Author / Date: Anthropic Engineering · 2025-09-29
- URL: https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents
- Mapped modules: context-engineering, agent-loop, memory, prompt-engineering, coding-agent, multi-agent
- Last reviewed: 2026-10-07
- Reference: [article-anthropic-context-engineering-2025](../../../references/articles.md#article-anthropic-context-engineering-2025)

## Claim（作者主张）

1. Context = 采样时进入模型的 **token 集合**；工程问题是在 LLM 固有约束下最大化这些 token 的效用。
2. **Context Engineering** 是 Prompt Engineering 的自然延伸：从「怎么写指令」扩展到「每次推理策展什么信息」。
3. Context 是有限资源（注意力预算）；存在 **context rot**——窗口变大时精确召回与长程推理能力下降。
4. 长程任务三招：**compaction**、**结构化笔记（agentic memory）**、**sub-agent**（子 Agent 只回浓缩摘要）。

## Mechanism（可验证的机制）

```text
可能信息宇宙（历史 / 工具结果 / 文件 / RAG / MCP…）
        ↓ 每次模型调用策展
最小高信号 token 集 → LLM
        ↓ 循环产生更多观测
再策展（含 JIT 检索、清理、压缩）
```

| 组件 | 文中口径 | 本仓库对齐 |
|------|----------|------------|
| System prompt | 正确「海拔」：非脆弱 if-else，亦非空泛假设 | Prompt = 指令合同 |
| Tools | 少重叠、返回 token 高效 | Tool Use + Context |
| Few-shot | 多样典范例，非边角规则清单 | Prompt Engineering |
| JIT / progressive disclosure | 句柄（路径、查询）运行时加载 | Context Engineering |
| Compaction | 摘要续窗；可先清深历史 tool result | Context |
| Note-taking | 窗外持久笔记再拉回 | Memory（写闸仍要） |
| Sub-agents | 隔离探索上下文，主 Agent 只收摘要 | Multi-Agent（可选） |

混合策略：部分预取（如 `CLAUDE.md`）+ 运行时 glob/grep 类探索。

## Evidence

- Needle-in-a-haystack / context rot 现象（跨模型存在，衰减斜率不同）。
- Transformer 全对注意力 → \(n^2\) 关系；训练分布偏短序列 → 长上下文经验更少。
- Claude Code：compaction + 最近文件；工具结果清理；子 Agent 研究系统另文。

## Engineering translation（我们怎么用）

| 决策 | 映射 |
|------|------|
| Prompt vs Context | [prompt-engineering](../../../core/prompt-engineering/prompt-engineering.md) · [context-engineering](../../../core/context-engineering/context-engineering.md) |
| Observation Explosion | Context Canonical 管道：normalize→filter→store→inject |
| Compaction / 前缀稳定 | Context + [llm-inference](../../../runtime/llm-inference/llm-inference.md) 缓存直觉 |
| 笔记写入 | [memory](../../../core/memory/memory.md)（必须写闸，非全文外挂） |
| 子 Agent | [multi-agent](../../../agent-types/multi-agent/multi-agent.md)；默认单 Agent |
| 「更大窗口」不是解药 | [llm](../../../foundations/llm/llm.md) 硬预算 + Lost-in-the-middle |

**可执行下一步**：为长任务定义 compaction 保留清单（决策/未决 bug/句柄）与丢弃清单（冗余 tool dump）；先调 recall 再压 precision。

## Disagreement / Caveats

- 厂商称 Context Engineering 为 Prompt 的「演进」——本仓库仍 **分模块**：指令合同 ≠ 窗口策展。
- Sub-agent 有通信与成本税；无评测证据勿默认拆分。
- Memory tool / 平台 API 字段易变 → TODO: Verify 产品细节；机制用本仓库 Memory/Context 词汇。

## Action items

1. 每次模型调用前回答：「这批 token 各自服务哪个决策？」
2. 工具结果默认摘要+句柄，禁止整页 dump。
3. Compaction prompt 用真实失败轨迹调参，并配回归集。

## References

- [article-anthropic-context-engineering-2025](../../../references/articles.md#article-anthropic-context-engineering-2025)
- [paper-lost-in-the-middle-2023](../../../references/papers.md#paper-lost-in-the-middle-2023)
- 对照：[context-engineering](../../../core/context-engineering/context-engineering.md)
