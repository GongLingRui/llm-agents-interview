# 项目实践路线

> 主题：Practice Paths — 用递增项目把知识主链「做出来」。

项目清单详情见：[`practice/agent-projects/README.md`](../practice/agent-projects/README.md)

## 原则

1. 每个项目只锁定 **1–2 个新机制**，避免 Demo 过大。
2. 每个项目必须有：**目标、非目标、验收标准、失败模式记录**。
3. 项目难度跟随知识依赖，不跳级上 Multi-Agent / RL。
4. 优先可本地运行；云服务作为可选项。

---

## Milestone 地图

```text
P0  Minimal Tool Agent
        ↓
P1  Robust Loop (retries, timeouts, tool errors)
        ↓
P2  Memory + Context budget
        ↓
P3  RAG-backed Agent
        ↓
P4  MCP-connected Agent
        ↓
P5  Coding Agent (sandbox)
        ↓
P6  Eval harness + Observability
        ↓
P7  Production slice (auth, rate limit, tracing)
        ↓
P8  (Optional) Multi-Agent or Browser Agent
```

---

## 分阶段说明

### P0 — Minimal Tool Agent

- **依赖**：LLM、Agent 基础、Tool Use、Agent Loop
- **做**：1 个 LLM + 2–3 个工具 + while loop
- **验收**：能完成需要工具的任务；能解释每轮状态

### P1 — Robust Loop

- **新增**：错误处理、重试、最大步数、人类中断点
- **验收**：工具失败不会死循环；有清晰终止条件

### P2 — Memory + Context Budget

- **新增**：短记忆、摘要、窗口预算
- **验收**：长对话不无脑塞满上下文；关键状态可恢复

### P3 — RAG-backed Agent

- **新增**：检索 → 引用 → 回答/行动
- **验收**：能区分「检索失败」与「推理失败」

### P4 — MCP-connected Agent

- **新增**：通过 MCP 暴露/消费工具
- **验收**：同一 Agent Loop 可替换工具后端

### P5 — Coding Agent（沙箱）

- **新增**：文件/命令工具 + Sandbox
- **验收**：能在隔离环境改代码跑测试；有权限边界

### P6 — Eval + Observability

- **新增**：任务集、轨迹记录、基础指标
- **验收**：改 prompt/工具后能量化对比

### P7 — Production Slice

- **新增**：鉴权、限流、密钥管理、追踪导出
- **验收**：具备最小上线清单（不是完整 SaaS）

### P8 — Optional 分支

- **Multi-Agent**：仅当单 Agent 职责清晰仍不够
- **Browser Agent**：仅在完成 Tool/Obs/Security 后

---

## 与学习 / 面试路线的对齐

| 实践里程碑 | 学习路径阶段 | 面试用途 |
|------------|--------------|----------|
| P0–P1 | Path A | 讲清 Loop |
| P2–P4 | Path B | 系统设计素材 |
| P5–P7 | Path C | 生产深挖 |
| P8 | 按需 | 差异化亮点 |

## 记录模板（每个项目 README 应有）

```markdown
# Project: <name>
## Goal
## Non-goals
## Depends on modules
## Architecture sketch
## Acceptance criteria
## Failure modes observed
## What I'd improve next
## References
```
