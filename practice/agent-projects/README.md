# Agent Projects

> 职责：定义可执行的项目里程碑，对齐知识主链。

详细路线：[docs/practice-paths.md](../../docs/practice-paths.md)

## 项目目录（可执行里程碑）

| ID | 目录 | 锁定机制 |
|----|------|----------|
| P0 | [P0-minimal-tool-agent](./P0-minimal-tool-agent/README.md) | Tool Use + Loop |
| P1 | [P1-robust-loop-agent](./P1-robust-loop-agent/README.md) | 重试/终止/错误 |
| P2 | [P2-memory-context-agent](./P2-memory-context-agent/README.md) | Memory + Context budget |
| P3 | [P3-rag-agent](./P3-rag-agent/README.md) | RAG + grounding |
| P4 | [P4-mcp-agent](./P4-mcp-agent/README.md) | MCP 接入 |
| P5 | [P5-coding-agent-sandbox](./P5-coding-agent-sandbox/README.md) | Coding + Sandbox |
| P6 | [P6-eval-obs-harness](./P6-eval-obs-harness/README.md) | Eval + Observability |
| P7 | [P7-production-slice](./P7-production-slice/README.md) | Infra 最小切片 |
| P8a | [P8a-optional-multi-agent](./P8a-optional-multi-agent/README.md) | Multi-Agent（可选） |
| P8b | [P8b-optional-browser-agent](./P8b-optional-browser-agent/README.md) | Browser Agent（可选） |

## 规则

1. 每个项目只锁定 1–2 个新机制。
2. 必须有 Goal / Non-goals / Depends / Acceptance / Failures。
3. **不**在此目录放完整可运行代码仓库（实现放你自己的练习仓）。
4. 先单 Agent，再考虑 P8。
