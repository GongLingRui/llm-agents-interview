# 学习路线

> 主题：Learning Paths — 按读者起点给出有依赖顺序的学习路径。

## 路径总览

| 路径 | 适合谁 | 终点能力 |
|------|--------|----------|
| A. Agent Zero to One | 会一点编程，LLM 新手 | 能实现并解释最小 Agent Loop |
| B. Agent Systems Engineer | 已会 Tool Calling | 能设计带 Memory/Context/Eval 的系统 |
| C. Production Agent Engineer | 已做过 Demo Agent | 能上线、观测、加固、控成本 |
| D. Research-leaning Engineer | 想读论文并落地 | 能把论文机制映射到工程模块 |

所有路径都遵守同一主链：

`LLM → Tool Calling → Agent Loop → Planning → Memory/Context/RAG → Agent Types → Eval → Production`

---

## Path A — Agent Zero to One

**目标**：4–6 周内打通最小可运行 Agent。

| 阶段 | 模块 | 产出 |
|------|------|------|
| A1 | [LLM](../foundations/llm/llm.md) | 说清 token、上下文、生成与工具相关能力边界 |
| A2 | [Agent](../foundations/agent/agent.md) | 区分 Chatbot / Workflow / Agent |
| A3 | [Prompt Engineering](../core/prompt-engineering/prompt-engineering.md) + [Tool Use Canonical](../core/tool-use/tool-use.md) | 写出可调用工具的 prompt/schema |
| A4 | [Agent Loop Canonical](../core/agent-loop/agent-loop.md) | 实现 observe → think → act → observe |
| A5 | [Reasoning & Planning Canonical](../core/reasoning-planning/reasoning-planning.md)（入门：何时 plan/reflect/verify） | 给 Loop 加上受预算约束的策略层 |
| A6 | [practice/agent-projects](../practice/agent-projects/README.md)（P0 项目） | 完成最小 Tool-using Agent |

**暂缓**：Multi-Agent、RL、Inference 深水、大规模 Infra。

---

## Path B — Agent Systems Engineer

**前置**：完成 Path A 或等价经验。

| 阶段 | 模块 | 产出 |
|------|------|------|
| B1 | [Agent Architecture Canonical](../core/agent-architecture/agent-architecture.md) | 画出组件图与控制流 / Control-Data Plane |
| B2 | [Memory](../core/memory/memory.md) + [Context Engineering](../core/context-engineering/context-engineering.md) | 设计写闸/召回与窗口注入策略 |
| B3 | [RAG](../core/rag/rag.md) | 在需要外部知识时正确接入检索（分层评测） |
| B4 | [MCP](../protocols/mcp/mcp.md) | 理解可插拔工具面 |
| B5 | 选一：[coding-agent](../agent-types/coding-agent/coding-agent.md) 或 [browser-computer-use](../agent-types/browser-computer-use/browser-computer-use.md) | 完成一个垂直 Agent 深潜 |
| B6 | [Evaluation](../production/evaluation/evaluation.md) | 建立任务集与失败分类 |
| B7 | [practice/system-design](../practice/system-design/system-design.md)（入门题） | 书面设计一个 Agent 系统 |

---

## Path C — Production Agent Engineer

**前置**：Path B。

| 阶段 | 模块 | 产出 |
|------|------|------|
| C1 | [Observability](../production/observability/observability.md) | Trace / span / 工具调用审计 |
| C2 | [Sandbox](../runtime/sandbox/sandbox.md) + [Security](../production/security/security.md) | 威胁模型 + 隔离策略 |
| C3 | [llm-inference](../runtime/llm-inference/llm-inference.md) + [model-optimization](../runtime/model-optimization/model-optimization.md) | 延迟/成本/吞吐权衡 |
| C4 | [infrastructure](../production/infrastructure/infrastructure.md) | 部署拓扑、队列、限流、密钥 |
| C5 | [design-patterns](../production/design-patterns/design-patterns.md) | 沉淀可复用模式 |
| C6 | [multi-agent](../agent-types/multi-agent/multi-agent.md)（按需） | 仅在单 Agent 不够时引入 |
| C7 | [fine-tuning](../runtime/fine-tuning/fine-tuning.md) / [reinforcement-learning](../runtime/reinforcement-learning/reinforcement-learning.md)（按需） | 用数据闭环改进策略 |

---

## Path D — Research → Engineering

| 阶段 | 模块 | 产出 |
|------|------|------|
| D1 | 选定 Core 机制（Loop / Memory / Tool / Multi-Agent） | 明确研究问题 |
| D2 | [research/papers](../research/papers/README.md) + [论文规范](../standards/paper-analysis.md) | 机制笔记，不是摘要堆砌 |
| D3 | [research/open-source-analysis](../research/open-source-analysis/README.md) | 找到至少 1 个实现对照 |
| D4 | 映射回 [core/](../core/README.md) 或 [agent-types/](../agent-types/README.md) | 工程可复用结论 |
| D5 | [practice/agent-projects](../practice/agent-projects/README.md) | 最小复现或对比实验 |

---

## 每周节奏建议

1. **读**：对应模块 README → Canonical 主文
2. **画**：依赖图 + 控制流（Loop / 架构）
3. **做**：一个最小实验或项目切片
4. **评**：用 Eval 思路记录失败模式
5. **链**：写回 references，避免断链知识
