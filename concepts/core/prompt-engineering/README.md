# Prompt Engineering

> 职责：指令、角色、示例与输出协议设计 — 在 Agent 系统中服务工具调用与可靠行为。

## Canonical 文档

**主文（Source of Truth）**：[prompt-engineering.md](./prompt-engineering.md)

Phase 1 计划拆成的多篇主题已合并进上述 Canonical 文档，避免浅层重复。后续仅在需要独立深潜时再拆子文，并回链主文。

核心立场：

- **重要，但不是** Agent Engineering 的中心  
- 中心是 **Loop + Tools + Context**  
- **Prompt = Instruction**；窗口策展见 [Context Engineering](../context-engineering/context-engineering.md)

## 受众

编写/维护 Agent 系统提示、工具描述措辞与输出契约的工程师。

## 前置依赖

`foundations/llm`（建议同步了解 `foundations/agent`）

## 主题覆盖（已并入 Canonical）

| 原计划文件 | 主题 | Canonical 落点 |
|------------|------|----------------|
| `prompts-for-agents.md` | Agent 场景下的系统提示结构 | §3 |
| `tool-aware-prompting.md` | 引导工具使用与避免滥用（ACI 措辞） | §4 |
| `structured-output.md` | JSON/Schema/约束解码的工程用法 | §5 |
| `few-shot-and-recipes.md` | 示例选择与配方化 | §6 |
| `prompt-vs-context.md` | 与 Context Engineering 的分工（必读） | §2 |
| `evaluation-of-prompts.md` | Prompt 变更如何回归测试 | §8 |

## 非目标

- 窗口级信息编排、预算、压缩、Observation Explosion → [context-engineering.md](../context-engineering/context-engineering.md)
- 工具执行、鉴权、幂等机制正文 → [tool-use.md](../tool-use/tool-use.md)
- 模型训练式对齐 → [`runtime/fine-tuning`](../../runtime/fine-tuning/README.md) / [`runtime/reinforcement-learning`](../../runtime/reinforcement-learning/README.md)

## 边界

- **不是** Context Engineering 的改名：Prompt 管指令合同，Context 管当次策展  
- **不是** Tool Use：ACI 措辞在此；执行链路在 Tool Use  
- 与易混模块：改「说不清」→ Prompt；改「塞什么/塞太多」→ Context；改「怎么调/谁执行」→ Tools

## 上游 / 下游

- 上游：`foundations/llm`、`foundations/agent`
- 下游：[Context Engineering](../context-engineering/context-engineering.md)、多数 `agent-types/`、[`production/evaluation`](../../production/evaluation/README.md)
