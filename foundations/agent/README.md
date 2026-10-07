# Agent 基础

> 职责：定义 Agent，划清 Chatbot / Workflow / Agent 边界，建立能力模型与控制哲学（自主度、HITL、终止权）。

## Canonical 文档

**主文（Source of Truth）**：[agent.md](./agent.md)

Phase 1 计划拆成的多篇主题（定义、选型、能力模型、自主度与控制、早期失败）已合并进上述 Canonical 文档。Tool Use / Agent Loop / Memory / RAG / Context / Multi-Agent / Production 的深文仍在各自模块；本主文只做定义与选型层。

## 受众

所有读者的第二站（读完 LLM 基础后）。

## 前置依赖

硬前置：[LLM 基础](../llm/llm.md)（接口契约与采样/结构化输出路径）

## 非目标

- 具体框架 API 教程
- Multi-Agent 拓扑 → [`agent-types/multi-agent`](../../agent-types/multi-agent/README.md)
- 生产加固 → [`production/`](../../production/README.md)
- Tool / Loop / Memory / RAG / Context 机制正文 → 各自 Core Canonical

## 边界

- **本文** = 定义、选型、能力光谱、控制哲学、早期失败识别
- **Tool Use** = 行动契约；**Agent Loop** = 控制回路（本文只链接）
- **Agent Architecture** = 组件组装；不在 foundations 展开

## 上游 / 下游

- 上游：`foundations/llm`
- 下游：[`core/tool-use`](../../core/tool-use/tool-use.md)、[`core/agent-loop`](../../core/agent-loop/agent-loop.md)、`core/prompt-engineering`
