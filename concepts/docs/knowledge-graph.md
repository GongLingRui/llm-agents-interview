# 知识依赖关系

> 主题：Knowledge Graph — 模块之间的前置依赖与推荐学习边。

## 总图（工程主链）

```text
                    ┌─────────────────────┐
                    │  foundations/llm    │
                    └──────────┬──────────┘
                               │
                    ┌──────────▼──────────┐
                    │ foundations/agent   │
                    └──────────┬──────────┘
                               │
              ┌────────────────┼────────────────┐
              │                │                │
   ┌──────────▼──────┐  ┌──────▼──────┐  ┌──────▼──────────────┐
   │ core/tool-use   │  │ core/prompt │  │ runtime/llm-infer.  │
   └──────────┬──────┘  │ -engineering│  └──────────┬──────────┘
              │         └──────┬──────┘             │
              │                │                    │
   ┌──────────▼────────────────▼──────┐            │
   │         core/agent-loop          │◄───────────┘
   └──────────┬───────────────────────┘
              │
   ┌──────────▼───────────────────────┐
   │   core/reasoning-planning        │
   └──────────┬───────────────────────┘
              │
   ┌──────────▼───────────────────────┐
   │    core/agent-architecture       │
   └──────────┬───────────────────────┘
              │
     ┌────────┼────────┬────────────┐
     │        │        │            │
┌────▼───┐ ┌──▼───┐ ┌──▼────────┐ ┌─▼───────────────┐
│ memory │ │ rag  │ │ context-  │ │ protocols/mcp   │
│        │ │      │ │ engineering│ │ (extends tools) │
└────┬───┘ └──┬───┘ └──────┬────┘ └────────┬────────┘
     │        │            │               │
     └────────┴─────┬──────┴───────────────┘
                    │
         ┌──────────▼──────────┐
         │    agent-types/*    │
         │ coding / browser /  │
         │    multi-agent      │
         └──────────┬──────────┘
                    │
    ┌───────────────┼────────────────┐
    │               │                │
┌───▼────┐   ┌──────▼──────┐  ┌──────▼──────┐
│ eval   │   │ observability│  │  security   │
└───┬────┘   └──────┬──────┘  └──────┬──────┘
    │               │                │
    └───────────────┼────────────────┘
                    │
         ┌──────────▼──────────┐
         │  infrastructure     │
         │  (+ sandbox runtime)│
         └─────────────────────┘
```

## 模块依赖表

| 模块 | 硬前置（必须先懂） | 软前置（强烈建议） | 解锁下游 |
|------|--------------------|--------------------|----------|
| `foundations/llm` | — | — | Agent 基础、Inference、Prompt |
| `foundations/agent` | LLM | — | Tool Use、Agent Loop |
| `core/prompt-engineering` | LLM | Agent 基础 | Context、多数 Agent Types |
| `core/tool-use` | Agent 基础 | Prompt | Agent Loop、MCP、Coding/Browser |
| `core/agent-loop` | Tool Use | Prompt | Reasoning、Architecture、所有 Agent Types |
| `core/reasoning-planning` | Agent Loop | — | Architecture、Multi-Agent、复杂 Coding Agent |
| `core/agent-architecture` | Agent Loop、Reasoning | Memory/Context 概念 | Production Patterns、System Design |
| `core/memory` | Agent Loop | Context | Multi-Agent、长程任务 Agent |
| `core/rag` | LLM | Context、Prompt | Knowledge-heavy Agent、Eval 中的检索指标 |
| `core/context-engineering` | Agent Loop | Memory、RAG、Tool Use | Production Agent、Coding Agent（Canonical：每次调用塞什么） |
| `protocols/mcp` | Tool Use | Agent Loop | 可插拔工具生态、IDE/桌面 Agent |
| `agent-types/coding-agent` | Agent Loop、Tool Use | Context、Sandbox | Projects、Interview、Industry |
| `agent-types/browser-computer-use` | Agent Loop、Tool Use | Observability、Security | Computer-use 案例与评测 |
| `agent-types/multi-agent` | Architecture、Reasoning | Memory、Eval | 协作系统设计 |
| `runtime/sandbox` | Tool Use | Security 威胁意识 | Coding/Browser 安全执行 |
| `runtime/llm-inference` | LLM | — | Infra、Optimization、成本优化 |
| `runtime/model-optimization` | Inference | — | 部署成本与延迟工程 |
| `runtime/fine-tuning` | LLM | Eval | 领域 Agent 定制 |
| `runtime/reinforcement-learning` | LLM、Eval 思想 | Fine-tuning | 策略优化、Agent RL |
| `production/evaluation` | Agent Loop | Architecture | Obs、RL、上线决策 |
| `production/observability` | Agent Loop | Eval | Infra、调试实践 |
| `production/security` | Tool Use、Sandbox | MCP、Browser | Production 上线门槛 |
| `production/infrastructure` | Inference、Eval、Obs | Sandbox、Security | System Design、真实部署 |
| `production/design-patterns` | Architecture | Eval、Obs、Multi-Agent | 面试与系统设计 |
| `practice/*` | 对应被练习模块 | — | 能力内化 |
| `research/*` | 相关 Core 模块 | — | 前沿与证据 |

## 关键依赖边（容易混的）

```text
Tool Use ──extends──► MCP
Memory ──feeds──► Context Engineering ◄──feeds── RAG
Agent Loop ──specializes──► Coding / Browser / Multi-Agent
Evaluation ──signals──► RL / Fine-tuning / Prompt&Context 迭代
Sandbox ──enforces──► Security 策略的一部分
Inference ──constrains──► Infrastructure 成本与延迟
```

## 反模式：错误依赖顺序

| 错误做法 | 问题 |
|----------|------|
| 先学 Multi-Agent 再学 Agent Loop | 会把「多模型聊天」误当成 Multi-Agent |
| 只学 Prompt 不学 Tool Calling | 无法进入真正 Agent |
| 先堆 RAG 技巧再懂 Context | 检索变成目的，而不是服务决策 |
| 先做 Infra 再做 Eval | 不知道测什么，优化会漂 |
| 把 MCP 当 Agent 框架教程入口 | 协议细节淹没控制回路理解 |
