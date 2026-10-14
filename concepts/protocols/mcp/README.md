# MCP（Model Context Protocol）

> 职责：理解 MCP 作为工具/资源/提示的**互操作协议层**，以及它如何接入 Agent Loop——不是又一种 Tool Calling 教程。

## Canonical 文档

**主文（Source of Truth）**：[mcp.md](./mcp.md)

核心立场：

- **MCP = 协议与互操作**；**Tool Use = Agent 内机制**（见 [`core/tool-use`](../../core/tool-use/tool-use.md)）
- Host / Client / Server；完整对话留在 Host；Server 相互隔离
- Tools / Resources / Prompts 控制权不同，不可混用
- 引入 MCP 要付协议税；无多宿主互操作需求时本地工具往往足够
- 鉴权与信任边界在 Host；完整威胁模型链 [`production/security`](../../production/security/README.md)

Phase 1 计划拆成的多篇主题（mental model、primitives、wiring、auth、ecosystem、failure modes）已合并进上述 Canonical，避免浅层重复。

## 受众

要把可插拔工具/资源面接到 IDE、桌面或企业 Agent 的工程师。

## 前置依赖

硬前置：[Tool Use](../../core/tool-use/tool-use.md)  
建议：[Agent Loop](../../core/agent-loop/agent-loop.md)

## 非目标

- Tool Calling 协议细节正文 → [tool-use.md](../../core/tool-use/tool-use.md)
- 把 MCP 写成唯一 Agent 框架入门 → 先读 Tool Use → Agent Loop
- 无限罗列第三方 server 目录 → 不写
- 完整威胁模型 → [`production/security`](../../production/security/README.md)

## 边界

| | Tool Use | MCP |
|--|----------|-----|
| 层级 | 机制：声明/调用/执行/回灌 | 协议：跨进程/厂商标准化暴露 |
| 本仓库 | `core/tool-use` | `protocols/mcp` |

## 上游 / 下游

- 上游：[`core/tool-use`](../../core/tool-use/tool-use.md)、建议 [`core/agent-loop`](../../core/agent-loop/agent-loop.md)
- 下游：IDE / 桌面 Agent、可插拔企业工具面；[`production/security`](../../production/security/README.md)
