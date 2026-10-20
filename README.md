# Agent Engineering Handbook

> 从 LLM 基础到 Production Agent 的系统化工程知识库。

本仓库不是文章收藏夹，而是面向长期维护的 **Agent Engineering Knowledge Base**：把概念、架构、实现、评估、部署、优化串成一条可学习、可实践、可面试的工程路径。

---

## 顶层双支柱

| 支柱 | 目录 | 写什么 | 不写什么 |
|------|------|--------|----------|
| **概念知识** | [`concepts/`](concepts/) | 定义、机制、边界、评测框架、研究索引 | 团队私有 playbook、门禁数字、踩坑流水账 |
| **工程经验** | [`engineering-experience/`](engineering-experience/) | 可复现的坑、门槛数字、落地 playbook、真实约束下的取舍 | Canonical 机制正文（回链 concepts） |

**禁止混写**：同一主题的机制与经验分目录维护；经验文只链概念，不复制大段机制。

**课程材料独立**：如 `CS329A自我改进智能体/` 等课程目录留在仓库根（或你后续单独规划的课程文件夹），**不**并入上述双支柱。

---

## 这个仓库解决什么问题

最终读者应该能通过本仓库完成：

| 目标 | 对应区域 |
|------|----------|
| 从 0 学习 Agent | [学习路线](concepts/docs/learning-paths.md) |
| 理解底层原理 | `concepts/foundations/` → `concepts/core/` |
| 理解 Agent Architecture | `concepts/core/agent-architecture/` |
| 阅读 Agent 论文 | `concepts/research/papers/` |
| 分析开源 Agent 项目 | `concepts/research/open-source-analysis/` |
| 自己实现 Agent | `concepts/practice/agent-projects/` |
| 设计 Production Agent | `concepts/production/` + `concepts/practice/system-design/` |
| 调试 / 评估 / 部署 / 优化 | `concepts/production/` + `concepts/runtime/` |
| 准备 Agent / AI / LLM Engineer 面试 | `concepts/practice/interview-questions/` |
| 查阅 / 补充落地经验 | [`engineering-experience/`](engineering-experience/)（骨架，待补充） |

---

## 知识主链（先看这个）

本仓库按 **依赖关系** 组织，而不是按关键词平铺：

```text
LLM Fundamentals
        ↓
   Tool Calling
        ↓
    Agent Loop
        ↓
 Reasoning / Planning
        ↓
  Memory + Context + RAG
        ↓
   Specialized Agents
  (Coding / Browser / Multi-Agent)
        ↓
   Evaluation + Observability
        ↓
 Production Infrastructure
 (+ Security / Sandbox / Inference)
```

更完整的依赖图见：[知识依赖关系](concepts/docs/knowledge-graph.md)

---

## 仓库结构总览

| 区域 | 目录 | 职责 |
|------|------|------|
| 站点 / 元信息 | 根 `README` · `CONTRIBUTING` · Docsify 入口 | 导航与贡献约定 |
| 概念知识 | `concepts/` | 见下表分层 |
| 工程经验 | `engineering-experience/` | 主题占位，正文待补充 |
| 课程（独立） | 如 `CS329A自我改进智能体/` | 课程笔记，不进双支柱 |

### concepts/ 内分层

| 层级 | 目录 | 职责 |
|------|------|------|
| 0 Meta | `concepts/docs/` | 导航、路线、质量标准 |
| 1 Foundations | `concepts/foundations/` | LLM 与 Agent 最小必要基础 |
| 2 Core Systems | `concepts/core/` | Agent 的核心机制与子系统 |
| 3 Protocols | `concepts/protocols/` | 工具与外部系统互操作协议（MCP 等） |
| 4 Agent Types | `concepts/agent-types/` | 面向场景的 Agent 形态 |
| 5 Runtime | `concepts/runtime/` | 执行隔离、推理、模型优化与学习 |
| 6 Production | `concepts/production/` | 评估、可观测、安全、基础设施、模式 |
| 7 Practice | `concepts/practice/` | 项目、面试、系统设计 |
| 8 Research | `concepts/research/` | 论文、开源分析、案例、厂商实践 |
| 9 References | `concepts/references/` | 统一引用索引 |

```text
agent-engineering-handbook/
├── README.md
├── CONTRIBUTING.md
├── index.html / _sidebar.md / _navbar.md
├── concepts/
│   ├── docs/
│   ├── foundations/ … production/
│   ├── practice/ research/ references/
│   └── …
├── engineering-experience/    # 骨架，待补充
└── CS329A自我改进智能体/       # 课程，独立于双支柱
```

---

## 快速导航

### 按目标进入

- **想系统学习** → [学习路线](concepts/docs/learning-paths.md)
- **想高效阅读** → [阅读路线](concepts/docs/reading-paths.md)
- **想准备面试** → [面试路线](concepts/docs/interview-paths.md)
- **想动手做项目** → [项目实践路线](concepts/docs/practice-paths.md)
- **想理解知识关系** → [知识依赖关系](concepts/docs/knowledge-graph.md)
- **想贡献内容** → [CONTRIBUTING.md](CONTRIBUTING.md) + [文档质量标准](concepts/docs/quality-standards.md)
- **想写 / 查工程经验** → [engineering-experience](engineering-experience/README.md)

### 按模块进入

<details>
<summary><strong>1. Foundations</strong></summary>

- [LLM 基础](concepts/foundations/llm/llm.md)
- [Agent 基础](concepts/foundations/agent/agent.md)

</details>

<details>
<summary><strong>2. Core Systems</strong></summary>

- [Agent Architecture](concepts/core/agent-architecture/agent-architecture.md)
- [Agent Loop](concepts/core/agent-loop/agent-loop.md)
- [Reasoning & Planning](concepts/core/reasoning-planning/reasoning-planning.md)
- [Tool Use](concepts/core/tool-use/tool-use.md)
- [Memory](concepts/core/memory/memory.md)
- [Context Engineering](concepts/core/context-engineering/context-engineering.md)
- [Prompt Engineering](concepts/core/prompt-engineering/prompt-engineering.md)
- [RAG](concepts/core/rag/rag.md)

</details>

<details>
<summary><strong>3. Protocols</strong></summary>

- [MCP](concepts/protocols/mcp/mcp.md)

</details>

<details>
<summary><strong>4. Agent Types</strong></summary>

- [Coding Agent](concepts/agent-types/coding-agent/coding-agent.md)
- [Browser / Computer Use](concepts/agent-types/browser-computer-use/browser-computer-use.md)
- [Multi-Agent](concepts/agent-types/multi-agent/multi-agent.md)

</details>

<details>
<summary><strong>5. Runtime</strong></summary>

- [Sandbox](concepts/runtime/sandbox/sandbox.md)
- [LLM Inference](concepts/runtime/llm-inference/llm-inference.md)
- [Model Optimization](concepts/runtime/model-optimization/model-optimization.md)
- [Fine-tuning](concepts/runtime/fine-tuning/fine-tuning.md)
- [Reinforcement Learning](concepts/runtime/reinforcement-learning/reinforcement-learning.md)

</details>

<details>
<summary><strong>6. Production</strong></summary>

- [Agent Evaluation](concepts/production/evaluation/evaluation.md)
- [Observability](concepts/production/observability/observability.md)
- [Security](concepts/production/security/security.md)
- [Infrastructure](concepts/production/infrastructure/infrastructure.md)
- [Agent Design Patterns](concepts/production/design-patterns/design-patterns.md)

</details>

<details>
<summary><strong>7. Practice</strong></summary>

- [Agent Projects](concepts/practice/agent-projects/README.md)
- [Interview Questions](concepts/practice/interview-questions/interview-questions.md)
- [System Design](concepts/practice/system-design/system-design.md)

</details>

<details>
<summary><strong>8. Research</strong></summary>

- [Papers](concepts/research/papers/README.md)
- [Open Source Project Analysis](concepts/research/open-source-analysis/README.md)
- [Case Studies](concepts/research/case-studies/README.md)
- [Industry Practices](concepts/research/industry-practices/README.md)（Anthropic / OpenAI / Cursor 等）

</details>

<details>
<summary><strong>9. Meta</strong></summary>

- [仓库架构说明](concepts/docs/architecture.md)
- [文档质量标准](concepts/docs/quality-standards.md)
- [References 索引](concepts/references/README.md)

</details>

---

## 设计原则（简版）

1. **依赖优先**：先有 LLM / Tool Calling，再有 Agent Loop，再有 Memory / Multi-Agent / Production。
2. **一主题一位置**：同一概念只放在一个主目录；交叉引用用链接，不复制正文。
3. **概念与经验分离**：机制进 `concepts/`，落地经验进 `engineering-experience/`。
4. **工程导向**：每个概念模块最终要回答「怎么建、怎么测、怎么挂、怎么稳」。
5. **规范先于数量**：先有质量标准与落点约定，再填充内容，避免知识库腐烂。

完整架构决策见：[concepts/docs/architecture.md](concepts/docs/architecture.md)

---

## 当前阶段

| 阶段 | 状态 |
|------|------|
| Phase 1 — Repository Architecture | ✅ 目录、路线、规范 |
| Phase 2 — Canonical Knowledge（1A+2A） | ✅ 工程 Canonical + Practice + Research 笔记 |
| Research 深潜（有来源） | ✅ 论文 27 篇笔记 · Industry×7 · OSS×3 · Case×3 · Talk×2 |
| Evergreen Review | ✅ 机制级复核已落地；价目/模型矩阵等周期 Verify |
| 双支柱拆分 | ✅ `concepts/` + `engineering-experience/` 骨架 |

入口见上方「按模块进入」。进度与审计：[architecture](concepts/docs/architecture.md) · [Phase 2 Audit](concepts/docs/phase-2-canonical-audit.md)

---

## 在线阅读

站点（GitHub Pages / Docsify）：

**https://vangong1999.github.io/llm-agents-interview/**

源码仓库：https://github.com/VanGong1999/llm-agents-interview

---

## License

[MIT](LICENSE)
