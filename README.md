# Agent Engineering Handbook

> 从 LLM 基础到 Production Agent 的系统化工程知识库。

本仓库不是文章收藏夹，而是面向长期维护的 **Agent Engineering Knowledge Base**：把概念、架构、实现、评估、部署、优化串成一条可学习、可实践、可面试的工程路径。

---

## 这个仓库解决什么问题

最终读者应该能通过本仓库完成：

| 目标 | 对应区域 |
|------|----------|
| 从 0 学习 Agent | [学习路线](docs/learning-paths.md) |
| 理解底层原理 | `foundations/` → `core/` |
| 理解 Agent Architecture | `core/agent-architecture/` |
| 阅读 Agent 论文 | `research/papers/` |
| 分析开源 Agent 项目 | `research/open-source-analysis/` |
| 自己实现 Agent | `practice/agent-projects/` |
| 设计 Production Agent | `production/` + `practice/system-design/` |
| 调试 / 评估 / 部署 / 优化 | `production/` + `runtime/` |
| 准备 Agent / AI / LLM Engineer 面试 | `practice/interview-questions/` |

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

更完整的依赖图见：[知识依赖关系](docs/knowledge-graph.md)

---

## 仓库分层总览

| 层级 | 目录 | 职责 |
|------|------|------|
| 0 Meta | `docs/` `standards/` | 导航、路线、写作与分析规范 |
| 1 Foundations | `foundations/` | LLM 与 Agent 最小必要基础 |
| 2 Core Systems | `core/` | Agent 的核心机制与子系统 |
| 3 Protocols | `protocols/` | 工具与外部系统互操作协议（MCP 等） |
| 4 Agent Types | `agent-types/` | 面向场景的 Agent 形态 |
| 5 Runtime | `runtime/` | 执行隔离、推理、模型优化与学习 |
| 6 Production | `production/` | 评估、可观测、安全、基础设施、模式 |
| 7 Practice | `practice/` | 项目、面试、系统设计 |
| 8 Research | `research/` | 论文、开源分析、案例、厂商实践 |
| 9 References | `references/` | 统一引用索引 |

```text
agent-engineering-handbook/
├── README.md
├── CONTRIBUTING.md
├── docs/                      # 路线图、知识图、质量标准
├── standards/                 # 论文/开源/笔记/引用规范
├── foundations/               # LLM + Agent 基础
├── core/                      # Architecture / Loop / Tools / Memory / RAG / Context / Prompt
├── protocols/                 # MCP 等协议层
├── agent-types/               # Coding / Browser / Multi-Agent
├── runtime/                   # Sandbox / Inference / Optimization / FT / RL
├── production/                # Eval / Obs / Security / Infra / Patterns
├── practice/                  # Projects / Interview / System Design
├── research/                  # Papers / OSS / Cases / Industry
└── references/                # 统一 References
```

---

## 快速导航

### 按目标进入

- **想系统学习** → [学习路线](docs/learning-paths.md)
- **想高效阅读** → [阅读路线](docs/reading-paths.md)
- **想准备面试** → [面试路线](docs/interview-paths.md)
- **想动手做项目** → [项目实践路线](docs/practice-paths.md)
- **想理解知识关系** → [知识依赖关系](docs/knowledge-graph.md)
- **想贡献内容** → [CONTRIBUTING.md](CONTRIBUTING.md) + [文档质量标准](docs/quality-standards.md)

### 按模块进入

<details>
<summary><strong>1. Foundations</strong></summary>

- [LLM 基础](foundations/llm/llm.md)
- [Agent 基础](foundations/agent/agent.md)

</details>

<details>
<summary><strong>2. Core Systems</strong></summary>

- [Agent Architecture](core/agent-architecture/agent-architecture.md)
- [Agent Loop](core/agent-loop/agent-loop.md)
- [Reasoning & Planning](core/reasoning-planning/reasoning-planning.md)
- [Tool Use](core/tool-use/tool-use.md)
- [Memory](core/memory/memory.md)
- [Context Engineering](core/context-engineering/context-engineering.md)
- [Prompt Engineering](core/prompt-engineering/prompt-engineering.md)
- [RAG](core/rag/rag.md)

</details>

<details>
<summary><strong>3. Protocols</strong></summary>

- [MCP](protocols/mcp/mcp.md)

</details>

<details>
<summary><strong>4. Agent Types</strong></summary>

- [Coding Agent](agent-types/coding-agent/coding-agent.md)
- [Browser / Computer Use](agent-types/browser-computer-use/browser-computer-use.md)
- [Multi-Agent](agent-types/multi-agent/multi-agent.md)

</details>

<details>
<summary><strong>5. Runtime</strong></summary>

- [Sandbox](runtime/sandbox/sandbox.md)
- [LLM Inference](runtime/llm-inference/llm-inference.md)
- [Model Optimization](runtime/model-optimization/model-optimization.md)
- [Fine-tuning](runtime/fine-tuning/fine-tuning.md)
- [Reinforcement Learning](runtime/reinforcement-learning/reinforcement-learning.md)

</details>

<details>
<summary><strong>6. Production</strong></summary>

- [Agent Evaluation](production/evaluation/evaluation.md)
- [Observability](production/observability/observability.md)
- [Security](production/security/security.md)
- [Infrastructure](production/infrastructure/infrastructure.md)
- [Agent Design Patterns](production/design-patterns/design-patterns.md)

</details>

<details>
<summary><strong>7. Practice</strong></summary>

- [Agent Projects](practice/agent-projects/README.md)
- [Interview Questions](practice/interview-questions/interview-questions.md)
- [System Design](practice/system-design/system-design.md)

</details>

<details>
<summary><strong>8. Research</strong></summary>

- [Papers](research/papers/README.md)
- [Open Source Project Analysis](research/open-source-analysis/README.md)
- [Case Studies](research/case-studies/README.md)
- [Industry Practices](research/industry-practices/README.md)（Anthropic / OpenAI / Cursor 等）

</details>

<details>
<summary><strong>9. Meta & Standards</strong></summary>

- [仓库架构说明](docs/architecture.md)
- [开源项目分析规范](standards/open-source-analysis.md)
- [论文分析规范](standards/paper-analysis.md)
- [文章笔记规范](standards/article-notes.md)
- [Reference 管理规范](standards/reference-management.md)
- [文档质量标准](docs/quality-standards.md)
- [References 索引](references/README.md)

</details>

---

## 设计原则（简版）

1. **依赖优先**：先有 LLM / Tool Calling，再有 Agent Loop，再有 Memory / Multi-Agent / Production。
2. **一主题一位置**：同一概念只放在一个主目录；交叉引用用链接，不复制正文。
3. **工程导向**：每个模块最终要回答「怎么建、怎么测、怎么挂、怎么稳」。
4. **规范先于数量**：先有分析/笔记/引用规范，再填充内容，避免知识库腐烂。
5. **骨架可演进**：第一阶段只建架构与主题定义，不堆无意义空文件。

完整架构决策见：[docs/architecture.md](docs/architecture.md)

---

## 当前阶段

| 阶段 | 状态 |
|------|------|
| Phase 1 — Repository Architecture | ✅ 目录、路线、规范 |
| Phase 2 — Canonical Knowledge（1A+2A） | ✅ 工程 Canonical + Practice + Research 笔记 |
| Research 深潜（有来源） | ✅ 论文 27 篇笔记 · Industry×6 · OSS×3 · Case×3 · Talk×2 |
| Evergreen Review | ✅ 机制级复核已落地；价目/模型矩阵等周期 Verify |

入口见上方「按模块进入」。进度与审计：[docs/architecture.md](docs/architecture.md) · [Phase 2 Audit](docs/phase-2-canonical-audit.md)

---

## 在线阅读

站点（GitHub Pages / Docsify）：

**https://vangong1999.github.io/agent-engineering-handbook/**

源码仓库：https://github.com/VanGong1999/agent-engineering-handbook

---

## License

[MIT](LICENSE)
