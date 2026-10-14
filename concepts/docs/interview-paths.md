# 面试路线

> 主题：Interview Paths — 按岗位映射考点到模块，形成可训练路径。

## 岗位映射

| 岗位 | 高频模块 | 次高频 |
|------|----------|--------|
| Agent Engineer | Loop、Tool Use、Architecture、Eval、Obs | Memory、Context、MCP、Security |
| AI Engineer | LLM 基础、RAG、Prompt/Context、Infra | Fine-tuning、Inference |
| LLM Engineer | Inference、Optimization、FT/RL、Eval | Agent Loop（加分） |
| Full-stack + Agent | Coding Agent、MCP、Sandbox、System Design | Observability |

详细题库目录：[`practice/interview-questions/`](../practice/interview-questions/README.md)

---

## Track I1 — Agent Engineer（推荐主轨）

### Stage 1 — 概念辨析（必须无漏洞）

- Chatbot vs Agent vs Workflow
- Tool Calling 失败模式
- Agent Loop 每一步的输入输出
- Planning 何时值得引入

对应模块：[foundations/agent](../foundations/agent/agent.md)、[core/tool-use](../core/tool-use/tool-use.md)、[core/agent-loop](../core/agent-loop/agent-loop.md)、[core/reasoning-planning](../core/reasoning-planning/reasoning-planning.md)

### Stage 2 — 系统设计题

常见题型：

1. 设计一个 Coding Agent
2. 设计带企业知识库的 Support Agent
3. 设计可观测、可回放的 Agent 平台
4. 设计 Computer Use Agent 的安全边界

训练方式：走 [practice/system-design/](../practice/system-design/system-design.md)，答案结构固定为：

目标 → 约束 → 架构 → Loop → Tools/Memory/Context → Eval → Obs → Security → 权衡

### Stage 3 — 深水追问

- Context 爆了怎么办
- 工具结果投毒怎么办
- 多 Agent 何时不该用
- 如何做 offline / online eval
- 成本与延迟怎么拆

对应：[Context Engineering](../core/context-engineering/context-engineering.md)、[security](../production/security/security.md)、[multi-agent](../agent-types/multi-agent/multi-agent.md)、[evaluation](../production/evaluation/evaluation.md)、[infrastructure](../production/infrastructure/infrastructure.md)

### Stage 4 — 项目叙事

用 [practice/agent-projects/](../practice/agent-projects/README.md) 的项目按 STAR + 技术深度讲述：

- 你的控制回路是什么
- 最大失败模式是什么
- 你如何证明变好了（Eval）

---

## Track I2 — AI / LLM Engineer（Agent 加分版）

1. [foundations/llm](../foundations/llm/llm.md) → [llm-inference](../runtime/llm-inference/llm-inference.md) → [model-optimization](../runtime/model-optimization/model-optimization.md)
2. [RAG](../core/rag/rag.md) + [Context Engineering](../core/context-engineering/context-engineering.md)
3. [evaluation](../production/evaluation/evaluation.md)（模型评测 vs Agent 评测差异）
4. 选修：[fine-tuning](../runtime/fine-tuning/fine-tuning.md) / [reinforcement-learning](../runtime/reinforcement-learning/reinforcement-learning.md)
5. 加分：能把模型侧优化连回 Agent Loop 行为变化

---

## Track I3 — 两周冲刺计划

| 天 | 焦点 |
|----|------|
| D1–D2 | Agent / Tool / Loop 口述 + 手绘 |
| D3–D4 | Memory / Context / RAG 边界题 |
| D5–D6 | Coding Agent 或 Support Agent 系统设计 |
| D7 | Eval + Observability |
| D8 | Security + Sandbox |
| D9 | 1 个开源项目拆解口述 |
| D10–D12 | 面试题库刷题（按模块标签） |
| D13–D14 | Mock：系统设计 + 深挖失败模式 |

---

## 面试作答质量标准（本仓库口径）

好的回答必须包含：

1. **机制**：发生了什么
2. **结构**：组件与数据流
3. **失败模式**：会怎么坏
4. **度量**：如何知道好坏
5. **权衡**：成本、延迟、复杂度、安全

只有名词堆砌 = 不合格。
