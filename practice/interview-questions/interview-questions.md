# Interview Questions（Practice Index）

> **一句话主题**：按模块索引各 Canonical 面试题，并提供跨模块综合追问——练的是机制与权衡，不是背诵。
>
> **状态**：Practice Index · **Last reviewed**: 2026-10  
> **受众**：面试准备与面试官出题  
> **前置**：对应 Canonical 至少读完定义与边界节

---

## 使用方式

1. 按岗位选 Track（见 [interview-paths.md](../../docs/interview-paths.md)）  
2. 模块深挖 → 跳到该 Canonical 的 **Interview Questions** 节（各约 45 题）  
3. 用本文 **综合题** 练串联与系统设计口述  

**禁止**：把各 Canonical 题库整篇粘贴到此处（一主题一位置）。

---

## 1. 模块题库索引

### Foundations

| 模块 | Canonical 题库 |
|------|----------------|
| LLM | [foundations/llm/llm.md](../../foundations/llm/llm.md) §Interview |
| Agent | [foundations/agent/agent.md](../../foundations/agent/agent.md) §Interview |

### Core

| 模块 | Canonical 题库 |
|------|----------------|
| Tool Use | [tool-use.md](../../core/tool-use/tool-use.md) |
| Agent Loop | [agent-loop.md](../../core/agent-loop/agent-loop.md) |
| Reasoning & Planning | [reasoning-planning.md](../../core/reasoning-planning/reasoning-planning.md) |
| Agent Architecture | [agent-architecture.md](../../core/agent-architecture/agent-architecture.md) |
| Memory | [memory.md](../../core/memory/memory.md) |
| RAG | [rag.md](../../core/rag/rag.md) |
| Context Engineering | [context-engineering.md](../../core/context-engineering/context-engineering.md) |
| Prompt Engineering | [prompt-engineering.md](../../core/prompt-engineering/prompt-engineering.md) |

### Protocols / Agent Types / Runtime / Production

| 模块 | Canonical |
|------|-----------|
| MCP | [mcp.md](../../protocols/mcp/mcp.md) |
| Coding Agent | [coding-agent.md](../../agent-types/coding-agent/coding-agent.md) |
| Browser / Computer Use | [browser-computer-use.md](../../agent-types/browser-computer-use/browser-computer-use.md) |
| Multi-Agent | [multi-agent.md](../../agent-types/multi-agent/multi-agent.md) |
| Sandbox | [sandbox.md](../../runtime/sandbox/sandbox.md) |
| LLM Inference | [llm-inference.md](../../runtime/llm-inference/llm-inference.md) |
| Model Optimization | [model-optimization.md](../../runtime/model-optimization/model-optimization.md) |
| Fine-tuning | [fine-tuning.md](../../runtime/fine-tuning/fine-tuning.md) |
| RL for Agents | [reinforcement-learning.md](../../runtime/reinforcement-learning/reinforcement-learning.md) |
| Evaluation | [evaluation.md](../../production/evaluation/evaluation.md) |
| Observability | [observability.md](../../production/observability/observability.md) |
| Security | [security.md](../../production/security/security.md) |
| Infrastructure | [infrastructure.md](../../production/infrastructure/infrastructure.md) |
| Design Patterns | [design-patterns.md](../../production/design-patterns/design-patterns.md) |

系统设计专项：[system-design.md](../system-design/system-design.md)

---

## 2. 跨模块综合题（25）

**C1.** 用一张图解释：State、Memory、RAG、Context、Prompt 在一次 Loop 迭代中的数据流。  
**期望**：Context 是投影；Memory/RAG 是可选源；Prompt 是指令分区。

**C2.** 为什么「更大 context window」不能替代 Context Engineering？  
**期望**：注意力/噪声/成本/Lost in the Middle。

**C3.** Tool Use 与 MCP 差在哪一层？何时本地工具足够？  
**期望**：机制 vs 互操作协议。

**C4.** Observation Explosion 的检测信号与默认管道？  
**期望**：normalize→filter→store→inject portion。

**C5.** 设计写闸：哪些东西永远不进长期 Memory？  
**期望**：密钥、一次性工具 dump、未验证用户声明。

**C6.** RAG 命中与 Memory 偏好冲突时如何决策？  
**期望**：权限与政策优先；冲突显式；统一 rank。

**C7.** Plan-and-Execute 何时比纯 ReAct 值得？成本在哪？  
**期望**：长任务/可验证里程碑；计划过时需 replan。

**C8.** 单 Agent 何时拆 Multi-Agent？给反例。  
**期望**：职责/权限/并行真需求；反例：为炫技拆。

**C9.** Coding Agent 如何避免「整仓进窗口」？  
**期望**：JIT 句柄、diff、测试摘要。

**C10.** Browser Agent 与 Coding Agent 的感知-动作差异？  
**期望**：DOM/截图 vs 文件/AST；脆弱选择器。

**C11.** Sandbox 与 Security 如何分工？  
**期望**：隔离机制 vs 威胁模型与政策。

**C12.** 提示注入从检索文档进来时，哪一层必须硬防？  
**期望**：Assembler 分区 + Tool authz，不靠模型服从。

**C13.** Offline eval 过了，线上仍差：可能原因三条。  
**期望**：分布漂移、工具环境差、延迟导致早停。

**C14.** Trace 里最少要有哪些字段才能复盘一次错误购票？  
**期望**：run_id、tool args/results、policy decision、context token map。

**C15.** 前缀缓存如何反向约束 Prompt/Context 设计？  
**期望**：稳定 system 前缀；变动放大后。

**C16.** 何时上微调而不是改 Prompt/加 RAG？  
**期望**：稳定行为缺口、数据充足、延迟/格式硬约束。

**C17.** 为何说 Eval 先于 RL？  
**期望**：无可靠奖励/任务集则 RL 优化噪声。

**C18.** Design pattern「workflow vs agent」如何选型？  
**期望**：可预知路径用 workflow；需自主探索用 agent。

**C19.** 成本防火墙：除了 RPM 还限什么？  
**期望**：context tokens、tool 调用、sandbox 时长、每 run 预算。

**C20.** 如何向面试官证明你的 Agent「变好了」？  
**期望**：固定集、消融、失败分类、成本/延迟伴随指标。

**C21.** Control Plane 与 Data Plane 在 Architecture 里如何落地到 infra？  
**期望**：编排/政策 vs 工具 IO/检索语料。

**C22.** Compaction 与 Memory 巩固的职责边界？  
**期望**：本 run 续航 vs 跨 run；巩固需写闸。

**C23.** 多租户下 Memory 隔离失败的后果与检测？  
**期望**：串数据；tenant_id 强制、红队。

**C24.** 给你一个「会调用 40 个工具」的 Agent，你先砍什么？  
**期望**：工具面收敛、路由、schema 质量、评测驱动。

**C25.** 口述从 P0 做到 P7 你学到的最大失败。  
**期望**：结合 practice 项目；机制而非框架名。

---

## 3. 面试官评分维度（简表）

| 维度 | 强 | 弱 |
|------|----|----|
| 边界意识 | 主动划 Memory/RAG/Context | 混为一谈 |
| 失败模式 | 能举检测信号 | 只讲 happy path |
| 权衡 | 成本/延迟/安全 | 只追准确率 |
| 生产感 | 提 trace/预算/权限 | 停留 demo |

---

## 4. Cross References

- [interview-paths.md](../../docs/interview-paths.md)  
- [system-design.md](../system-design/system-design.md)  
- [practice-paths.md](../../docs/practice-paths.md)
