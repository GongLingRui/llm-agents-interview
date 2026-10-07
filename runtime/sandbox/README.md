# Sandbox

> 职责：工具与代码执行的隔离机制 — 文件系统、网络、进程、权限与生命周期。

## Canonical 文档

**主文（Source of Truth）**：[sandbox.md](./sandbox.md)

Phase 1 计划拆成的多篇主题已合并进上述 Canonical 文档，避免浅层重复。后续仅在需要独立深潜（例如某一运行时逃逸案例集）时再拆子文，并回链主文。

核心立场：

- Sandbox = **副作用隔离执行平面**（机制），不是完整威胁模型  
- **AuthZ 在沙箱外**；沙箱回答「允许之后副作用落在哪里」  
- 默认：**断网 · 最小挂载 · 非特权 · 硬超时 · 产物显式导出**  
- 威胁模型 / 注入防护深文 → [Security](../../production/security/security.md)

## 受众

实现 Tool Executor、Coding Agent 运行时或平台隔离层的工程师。

## 前置依赖

硬前置：[Tool Use](../../core/tool-use/tool-use.md)  
建议：[Agent Loop](../../core/agent-loop/agent-loop.md)

## 非目标

- 完整威胁模型与注入防护深文 → [Security](../../production/security/security.md)
- 工具 schema / 选择 / 回灌机制 → [Tool Use](../../core/tool-use/tool-use.md)
- Coding 业务闭环 → [Coding Agent](../../agent-types/coding-agent/coding-agent.md)
- MCP 协议细节 → [`protocols/mcp`](../../protocols/mcp/README.md)

## 边界

- **不是** Security：本文只写隔离机制与挂钩  
- **不是** Tool Use：执行契约在 Tool Use，隔离宿主在本文  
- **容器 / 微 VM** 是实现手段，不是 Sandbox 定义本身

## 上游 / 下游

- 上游：`core/tool-use`、`core/agent-loop`
- 下游：`agent-types/coding-agent`、`agent-types/browser-computer-use`、`production/security`、`production/observability`、`production/infrastructure`
