# Coding Agent

> 职责：软件工程场景下的 Agent — 仓库感知、代码读写/编辑、命令执行、测试—修复闭环。

## Canonical 文档

**主文（Source of Truth）**：[coding-agent.md](./coding-agent.md)

Phase 1 计划拆成的多篇主题（loop、repo context、edit/diff、test-driven、sandboxing、evaluation）已合并进上述 Canonical 文档。Sandbox / Security / Tool Use / Loop / Context 的机制正文仍在各自模块；本主文只给出编码垂直特化与接口约定。

## 受众

实现或评估软件工程 Agent / Coding Runtime 的工程师。

## 前置依赖

硬前置：[Agent Loop](../../core/agent-loop/agent-loop.md)、[Tool Use](../../core/tool-use/tool-use.md)  
强烈建议：[Context Engineering](../../core/context-engineering/context-engineering.md)、[`runtime/sandbox`](../../runtime/sandbox/README.md)

## 非目标

- Tool Use / Loop / Sandbox / Security 机制正文 → 各模块 Canonical 或 README
- 某 IDE 产品手册、功能对照软文
- 语言语法教程

## 边界

| 主题 | 主责 |
|------|------|
| 工具契约与执行管线 | `core/tool-use` |
| 控制回路 / 终止 / checkpoint | `core/agent-loop` |
| 窗口装配与预算机制 | `core/context-engineering` |
| 隔离实现 | `runtime/sandbox` |
| 威胁模型 | `production/security` |
| 编码闭环与仓库策略 | **本文** |

## 上游 / 下游

- 上游：`core/agent-loop`、`core/tool-use`（建议 `core/context-engineering`）
- 下游：`practice/agent-projects`、`research/industry-practices`、`research/open-source-analysis`；可与 `multi-agent` 组合
