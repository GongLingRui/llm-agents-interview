# Case Studies

> 职责：端到端 Agent 系统案例 — 问题、架构、权衡、结果与复盘。  
> **策略**：按核实公开材料增量添加，禁止虚构指标；指标必须注明来源。

## 与 Industry Practices 的区别

- **Industry Practices**：厂商工程文章提炼与模块映射  
- **Case Studies**：单一产品/系统的问题→方案→结果叙事

## 当前案例

| 案例 | 说明 |
|------|------|
| [anthropic-multi-agent-research.md](./anthropic-multi-agent-research.md) | Research 功能：orchestrator–worker；内部指标已标注来源 |
| [anthropic-coding-agent-swe-bench.md](./anthropic-coding-agent-swe-bench.md) | Coding Agent + 可验证测试环 / ACI |
| [cursor-dynamic-context-harness.md](./cursor-dynamic-context-harness.md) | Coding harness：动态上下文发现与 MCP 工具面 |

## 案例模板（新增文件时使用）

```markdown
# Case: <name>
## Problem
## Constraints
## Architecture（链 Canonical，不重写机制）
## What worked
## What failed
## Metrics（注明来源；不可核实则 TODO: Verify）
## Mapped modules
## References（references/* ID）
```

## 建议映射模块

Architecture · Loop · Tool Use · Context · Eval · Obs · Security · Infra · Multi-Agent · Coding Agent
