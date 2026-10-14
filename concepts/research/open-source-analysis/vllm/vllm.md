# OSS: vLLM

## Metadata

- Repo / License / Language: [repo-vllm](../../../references/repos.md#repo-vllm) · License 见上游仓库 · Python
- Last reviewed: 2026-10-07
- Canonical modules mapped: llm-inference, model-optimization, observability, infrastructure, sandbox

## Scope / Non-goals

**范围**：从 Agent 工程视角拆解 vLLM 作为 **推理执行平面** 的可迁移设计——连续批处理、PagedAttention 式 KV 管理、高吞吐 serving 直觉。

**非目标**：

- 复述完整安装教程或替代官方文档
- 把 vLLM 写成唯一正确选型
- 重写 [llm-inference](../../../runtime/llm-inference/llm-inference.md) Canonical

**核实基线**：公开仓库 [repo-vllm](../../../references/repos.md#repo-vllm)；机制细节随版本演进，部署前对照当前文档 / 论文 [paper-pagedattention-2023](../../../references/papers.md#paper-pagedattention-2023)。

## Agent Loop 映射

| Agent 概念 | vLLM 角色 |
|------------|-----------|
| Model call | OpenAI-compatible / 原生 API 上的补全或 chat 生成 |
| Tool calling | **不在** vLLM 内执行工具；仅加速 token 生成；工具环仍在 Agent harness |
| 终止 | `max_tokens` / stop；Agent 的 max_steps 仍在控制面 |
| 观测 | 用量、延迟、队列；需与 Agent trace 关联 `request_id` |

```text
Agent Control Plane（Loop / Tools / Policy）
        ↓ model request
vLLM Data Plane（调度 · KV · 批处理）
        ↓ tokens / tool_call JSON（若模型产出）
回到 Control Plane 执行工具
```

## Tool surface

vLLM 本身不是 Tool Registry。对 Coding/Data Agent，常见模式是：

- 推理：vLLM
- 执行：独立 Sandbox / 业务 API

禁止把「模型吐出的 shell」直接在推理节点执行。

## State / Memory / Context

- **KV cache** = 单次/前缀生成的注意力状态，≠ Agent Memory。
- 前缀缓存 / 复用可降 TTFT 与成本，但要求 **稳定 system 前缀**（Context/Prompt 设计约束）。
- 多租户：隔离请求队列与缓存键，防止前缀串扰（安全与计费）。

## Eval / Obs hooks

建议挂钩：

- 每 Agent step 记录：`model`、`input/output tokens`、`queue_wait`、`ttft`、`e2e`
- 与 [observability](../../../production/observability/observability.md) 的 `agent.model_call` span 对齐
- 容量评测与任务评测分离（Inference SLO ≠ 任务成功率）

## Reusable designs

1. **控制面 / 数据面分离**：Harness 管 Loop；引擎管吞吐与显存。
2. **KV 分页思想**：显存碎片是长上下文 Agent 的隐成本中心。
3. **连续批处理**：多 Agent 并发时的调度直觉——公平性 vs 延迟尾巴。
4. **前缀稳定**：为缓存命中而设计 Prompt 分区（见 Context/Prompt Canonical）。

## Anti-patterns

- 在 GPU 节点上跑不可信工具代码
- 用「换更快引擎」掩盖 Context 膨胀与工具空转
- 把 PagedAttention 论文数字当你业务的绝对延迟承诺
- 无 `run_id` 关联的引擎日志（无法复盘 Agent 失败）

## Mapped handbook modules

- [llm-inference](../../../runtime/llm-inference/llm-inference.md)（主）
- [model-optimization](../../../runtime/model-optimization/model-optimization.md)
- [observability](../../../production/observability/observability.md)
- [infrastructure](../../../production/infrastructure/infrastructure.md)
- [sandbox](../../../runtime/sandbox/sandbox.md)（执行面对照）

## References

- [repo-vllm](../../../references/repos.md#repo-vllm)
- [paper-pagedattention-2023](../../../references/papers.md#paper-pagedattention-2023)
- [paper-orca-2022](../../../references/papers.md#paper-orca-2022)（连续批处理思想谱系）
