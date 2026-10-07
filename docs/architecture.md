# 仓库架构说明

> 主题：Repository Architecture — 分层理由、边界规则、演进策略。

## 为什么分层，而不是按关键词建文件夹

Agent Engineering 的知识是 **有向依赖** 的：不会 Tool Calling 就谈不上 Agent Loop；没有 Loop 就谈不上可靠 Memory / Multi-Agent；没有 Evaluation / Observability 就谈不上 Production。

因此本仓库用 **层（Layer）** 表达依赖，用 **模块（Module）** 表达职责边界。

## 九层结构

| Layer | Path | 一句话职责 | 不放什么 |
|-------|------|------------|----------|
| Meta | `docs/` `standards/` | 导航、路线、规范 | 技术正文 |
| Foundations | `foundations/` | LLM / Agent 最小前置 | 生产架构、框架教程 |
| Core | `core/` | Agent 通用机制 | 厂商产品软文 |
| Protocols | `protocols/` | 互操作协议 | Tool 选型百科 |
| Agent Types | `agent-types/` | 场景化 Agent 形态 | 把所有 Agent 都塞进来 |
| Runtime | `runtime/` | 执行与模型侧能力 | 业务 Agent 逻辑 |
| Production | `production/` | 可上线工程能力 | 纯研究综述 |
| Practice | `practice/` | 练习、面试、设计题 | 理论替代品 |
| Research | `research/` | 论文 / OSS / 案例 / 厂商实践 | 与核心机制重复的二手转述 |
| References | `references/` | 统一引用索引 | 笔记正文 |

## 关键边界决策

### 1. Prompt Engineering vs Context Engineering

| | Prompt Engineering | Context Engineering |
|--|--------------------|---------------------|
| 主问题 | 如何写好指令与交互结构 | 如何选择、组织、压缩进入上下文的信息 |
| 位置 | `core/prompt-engineering/` | `core/context-engineering/` |
| 关系 | Prompt 是 Context 的一部分策略；Context 覆盖检索、记忆、工具结果、历史裁剪 |

### 2. RAG vs Memory vs Context

- **RAG**：外部知识检索与注入子系统 → `core/rag/`
- **Memory**：跨回合状态与长期记忆机制 → `core/memory/`
- **Context Engineering**：把 RAG / Memory / Tools / History 编排进窗口 → `core/context-engineering/`

三者相邻，但主责不同；禁止在三个目录写同一篇「如何做检索」正文。

### 3. Tool Use vs MCP

- **Tool Use**：模型如何调用工具、工具设计、结果回灌、失败重试 → `core/tool-use/`
- **MCP**：工具/资源/提示的互操作协议与生态 → `protocols/mcp/`

MCP 是协议层，不是「又一种 Tool Calling 教程」。

### 4. Sandbox vs Security

- **Sandbox**：执行隔离、文件系统/网络策略、容器与权限边界 → `runtime/sandbox/`
- **Security**：威胁模型、注入、权限、数据泄漏、供应链 → `production/security/`

Security 会引用 Sandbox，但不把沙箱实现细节写进 Security 主文。

### 5. Evaluation vs Reinforcement Learning

- **Evaluation**：如何衡量 Agent 好坏（任务、轨迹、成本、安全）→ `production/evaluation/`
- **RL**：用反馈优化模型/策略（含 RLHF / RLAIF / Agent RL）→ `runtime/reinforcement-learning/`

评测是生产闭环；RL 是模型/策略改进路径。

### 6. Industry Practices vs Case Studies vs OSS Analysis

| 类型 | 目录 | 焦点 |
|------|------|------|
| 厂商工程文章与实践提炼 | `research/industry-practices/` | Anthropic / OpenAI / Cursor 等方法论 |
| 端到端产品/系统案例 | `research/case-studies/` | 问题、架构、权衡、结果 |
| 开源项目拆解 | `research/open-source-analysis/` | 代码结构、Agent Loop、可复用设计 |

### 7. Design Patterns 放在 Production，不放 Core

模式是「在真实约束下复用的结构」，依赖 Evaluation / Observability / Infra 语境。Core 讲机制，Production 讲可复用模式。

## 一主题一位置规则

1. 每个概念有且仅有一个 **Canonical Module**。
2. 其他模块只能链接，不能复制大段正文。
3. 若发现主题可放两处：优先放在依赖链更靠前、更「机制」的一侧；场景化内容放 `agent-types/` 或 `research/`。

## 模块内文件约定

每个内容模块至少有：

```text
<module>/
└── README.md    # 职责、受众、前置依赖、主题清单、非目标
```

主题正文**优先一篇 Canonical 主文**，而不是拆成十篇浅文；需要深潜时再拆子文并回链主文。

主题文件命名：`kebab-case.md`，例如 `tool-use.md`、`agent-loop.md`。

## Canonical 进度

| 模块 | 主文 | 状态 |
|------|------|------|
| `foundations/llm` | [llm.md](../foundations/llm/llm.md) | Canonical（2026-10） |
| `foundations/agent` | [agent.md](../foundations/agent/agent.md) | Canonical（2026-10） |
| `core/tool-use` | [tool-use.md](../core/tool-use/tool-use.md) | Canonical（2026-10） |
| `core/agent-loop` | [agent-loop.md](../core/agent-loop/agent-loop.md) | Canonical（2026-10） |
| `core/reasoning-planning` | [reasoning-planning.md](../core/reasoning-planning/reasoning-planning.md) | Canonical（2026-10） |
| `core/agent-architecture` | [agent-architecture.md](../core/agent-architecture/agent-architecture.md) | Canonical（2026-10） |
| `core/memory` | [memory.md](../core/memory/memory.md) | Canonical（2026-10） |
| `core/rag` | [rag.md](../core/rag/rag.md) | Canonical（2026-10） |
| `core/context-engineering` | [context-engineering.md](../core/context-engineering/context-engineering.md) | Canonical（2026-10） |
| `core/prompt-engineering` | [prompt-engineering.md](../core/prompt-engineering/prompt-engineering.md) | Canonical（2026-10） |
| `protocols/mcp` | [mcp.md](../protocols/mcp/mcp.md) | Canonical（2026-10） |
| `agent-types/coding-agent` | [coding-agent.md](../agent-types/coding-agent/coding-agent.md) | Canonical（2026-10） |
| `agent-types/browser-computer-use` | [browser-computer-use.md](../agent-types/browser-computer-use/browser-computer-use.md) | Canonical（2026-10） |
| `agent-types/multi-agent` | [multi-agent.md](../agent-types/multi-agent/multi-agent.md) | Canonical（2026-10） |
| `runtime/sandbox` | [sandbox.md](../runtime/sandbox/sandbox.md) | Canonical（2026-10） |
| `runtime/llm-inference` | [llm-inference.md](../runtime/llm-inference/llm-inference.md) | Canonical（2026-10） |
| `runtime/model-optimization` | [model-optimization.md](../runtime/model-optimization/model-optimization.md) | Canonical（2026-10） |
| `runtime/fine-tuning` | [fine-tuning.md](../runtime/fine-tuning/fine-tuning.md) | Canonical（2026-10） |
| `runtime/reinforcement-learning` | [reinforcement-learning.md](../runtime/reinforcement-learning/reinforcement-learning.md) | Canonical（2026-10） |
| `production/evaluation` | [evaluation.md](../production/evaluation/evaluation.md) | Canonical（2026-10） |
| `production/observability` | [observability.md](../production/observability/observability.md) | Canonical（2026-10） |
| `production/security` | [security.md](../production/security/security.md) | Canonical（2026-10） |
| `production/infrastructure` | [infrastructure.md](../production/infrastructure/infrastructure.md) | Canonical（2026-10） |
| `production/design-patterns` | [design-patterns.md](../production/design-patterns/design-patterns.md) | Canonical（2026-10） |

Practice：`practice/agent-projects` P0–P8 · [interview-questions](../practice/interview-questions/interview-questions.md) · [system-design](../practice/system-design/system-design.md)  
Research：论文全量笔记 + Industry/OSS/Case（仍禁止第二套机制正文与虚构数据）。

## 下一阶段写作优先级

1. ~~全部工程模块 Canonical（1A）~~ ✅
2. ~~Practice 可执行里程碑 + 面试/系统设计索引~~ ✅
3. ~~Research：已登记论文全量笔记 + Industry/OSS/Case~~ ✅（2026-10）
4. Evergreen Review：~~工具文档~~ ✅ · ~~Structured Outputs / caching / seed~~ ✅ · ~~caching TTL / 最低 token · Gemini seed~~ ✅（[caching-seed](./evergreen-2026-10-caching-seed.md)，2026-10）· ~~streaming/strict · refusal · HITL · compaction 机制~~ ✅（[agents-stream](./evergreen-2026-10-agents-stream.md)，2026-10）· **价目倍率 / 模型矩阵 / 产品默认 UX / 位置偏差评测** 继续
5. ~~NICE_TO_HAVE：进度文案 / 路径链接 / Talks 回链 / OSS Metadata / MIT 声明~~ ✅（2026-10）
6. 可选：更多 Case Studies、新 references 增量笔记（有 URL 才加；`TODO: Verify` 列表保留）

审计：[phase-2-canonical-audit.md](./phase-2-canonical-audit.md)
