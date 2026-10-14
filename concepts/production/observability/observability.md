# Agent Observability（Canonical）

> **一句话主题**：Agent Observability 让每次 run 的 **决策、工具副作用、上下文消耗与终止原因** 可追踪、可回放、可诊断——回答「发生了什么、卡在哪、钱花在哪、能不能复现」，而不是「好不好」。
>
> **状态**：Canonical · **Last reviewed**: 2026-10  
> **受众**：实现 Agent Runtime / 平台可观测、负责线上排障与合规审计的工程师  
> **硬前置**：[Agent Loop](../../core/agent-loop/agent-loop.md)、[Tool Use](../../core/tool-use/tool-use.md)  
> **软前置**：[Context Engineering](../../core/context-engineering/context-engineering.md)、[Agent Architecture](../../core/agent-architecture/agent-architecture.md)；边界对照 [`evaluation`](../evaluation/README.md)

---

## 范围 / 非目标

**范围**：Agent 遥测语义（run / step / model / tool）；traces 与 spans；工具审计；上下文 token 图；轨迹回放；用户反馈；与 Loop 挂钩；隐私红线；失败诊断；与 Evaluation 边界；面试。

**非目标**：

| 主题 | 去哪里 |
|------|--------|
| 任务成功/质量/回归门禁的指标体系与数据集方法论 | [`production/evaluation`](../evaluation/README.md) |
| 威胁模型、注入防护、沙箱硬化正文 | [`production/security`](../security/README.md) |
| 部署拓扑、队列、托管平台清单 | [`production/infrastructure`](../infrastructure/README.md) |
| Context 装配策略深文 | [Context Engineering](../../core/context-engineering/context-engineering.md) |
| 具体 APM/日志厂商安利 | 不写 |

```text
Evaluation  = 判好坏（rubric / 任务集 / 回归门禁）
Observability = 可观测（trace / 审计 / 回放 / 诊断）
```

Obs 产出轨迹与信号，可被 Eval **消费**；Obs **不定义**「什么叫成功」。

---

## 1. 知识模型

### 1.1 定义

**Agent Observability** 是面向 Agent Runtime 的遥测与诊断能力：在预算与隐私约束下，把一次任务从目标到终止的 **控制流、数据流、副作用与资源消耗** 变成可查询证据。

至少回答：①走了哪些步；②每步模型看见/决定了什么；③工具做了什么、是否授权；④token/延迟/成本如何增长；⑤失败能否复现与归因。

### 1.2 为何传统 APM 不够

| 传统请求追踪 | Agent run |
|--------------|-----------|
| 固定调用图 | 动态步数、模型驱动分支 |
| 结构化 API I/O | 文本/多模态上下文与工具观察 |
| 失败≈5xx/超时 | 空转、错工具、预算耗尽、目标漂移… |
| 重放≈重放请求 | 需采样、沙箱、checkpoint |

须在 OpenTelemetry 式 **trace/span** 上叠加 **Agent 语义层**（run、step、tool_call、context_budget、termination_reason）。

### 1.3 Mental Model

```text
Trace (run)
 ├─ Span: build_context (step i)
 ├─ Span: model_call (step i)
 ├─ Span: tool_batch → tool_call A/B…
 ├─ Span: checkpoint
 └─ Span/Event: terminate(reason)
```

**State** = 可恢复真相；**Trace** = 旁路证据；**Eval 标签** = 事后裁决——三者勿混成一张表。

---

## 2. 与 Evaluation 的边界（强制）

| 维度 | Observability | Evaluation |
|------|---------------|------------|
| 核心问题 | 发生了什么？能否复现？ | 好不好？是否回归？ |
| 主产物 | traces、events、审计、token 图 | 分数、通过率、切片、门禁 |
| 时间尺度 | 实时/近实时排障 | 离线集、影子、在线实验 |
| 单次 run | 完整轨迹与资源账本 | 可选打分（常抽样） |
| 成功定义 | **不定义** | **定义并测量** |
| 失败处理 | 诊断、告警、归因 | 进数据集与回归 |

**允许交叉**：脱敏轨迹导出给 Eval；Eval 失败样例回灌 Obs 告警阈值；👍/👎 同时进 Obs 事件与 Eval 标签管道（**存储 schema 分流**）。

**本文禁止展开**：rubric、LLM-as-judge、任务集构建长文；也禁止把「成功率」当唯一 Obs 指标而忽略轨迹完整性。

Anthropic 强调可观测与按需增加复杂度（[article-anthropic-building-effective-agents-2024](../../references/articles.md#article-anthropic-building-effective-agents-2024)）；本手册落成 **Runtime 挂钩 + 语义 span**，评测体系归 Evaluation。

---

## 3. 遥测语义模型

### 3.1 实体层级

| 实体 | 含义 | ID |
|------|------|-----|
| Tenant/User | 隔离与配额 | `tenant_id`, `user_id` |
| Session | 多 run 容器（可选） | `session_id` |
| **Run** | 一次目标驱动 Loop | `run_id` ≈ trace id |
| **Step** | 一轮 context→model→tools? | `step_id` / `step_index` |
| Model Call | 一次 LLM 调用 | `model_span_id` |
| Tool Call | 一次工具执行 | `tool_call_id`（对齐模型返回） |
| Checkpoint | 可恢复快照引用 | `checkpoint_id` |
| Feedback | 反馈事件 | `feedback_id` |

### 3.2 必备属性

| 层级 | 关键字段 |
|------|----------|
| **Run** | `goal_hash`；agent/prompt/tool_schema/policy **version**；`model_route`；`status`；`termination_reason`；起止时间；`totals`（steps/model/tool/tokens/cost） |
| **Step** | `step_index`；`phase`；关联 model/tool spans；`budget_snapshot` |
| **Model** | provider/model_id；采样摘要；usage（含 cached）；`finish_reason`；latency；`error_class`；I/O **对象引用**（禁高基数全文标签） |
| **Tool** | name；args/result digest；error_code；`authz_decision`；latency/retries；`side_effect_class`；依赖（HTTP/sandbox） |

`status` 建议：`running | succeeded | failed | cancelled | waiting_human | budget_exhausted`。

### 3.3 Span / Event / Log / Metric

| 类型 | 用途 | 例子 |
|------|------|------|
| Span | 有起止工作单元 | model_call、tool_call |
| Event | 点状事实 | `budget_warning`、`hitl_requested` |
| Log | 人类可读补充（仍脱敏） | 堆栈 |
| Metric | 聚合 | `agent.run.steps`、`agent.tool.error_rate` |

**结构化优先于自由文本**；自由文本是附件，不是主键。

---

## 4. Traces 与 Spans

### 4.1 设计原则

1. **一 run 一条根 trace**（或显式 parent run 链接）  
2. **Span name 稳定、低基数**（`agent.model_call`，勿塞用户问题）  
3. 高基数进 attributes / 旁路存储，**不进 metric label**  
4. 父子反映控制流：`step → model → tools*`（并行工具为兄弟）  
5. 错误分类：`timeout | schema | authz | upstream_5xx | model_refuse | cancelled`

### 4.2 OpenTelemetry 映射（建议）

不必绑厂商。概念映射：run ≈ 根 Trace / `invoke_agent`；内部 step 仍可用稳定名（如 `agent.step`）；model ≈ inference span（`gen_ai.operation.name`=`chat` / `generate_content` 等）；tool ≈ `execute_tool`；termination = Event + run status。

**Evergreen 2026-10-07（机制/字段名，非价目）**：官方 GenAI 语义约定已迁至独立仓库 [open-telemetry/semantic-conventions-genai](https://github.com/open-telemetry/semantic-conventions-genai)；原 [opentelemetry.io gen-ai 页](https://opentelemetry.io/docs/specs/semconv/gen-ai/) 仅作迁移说明。文档与属性登记表状态均为 **Development**（尚无 Stable `gen_ai.*`）。业务侧仍以**内部稳定 schema** 为真相，导出适配器再映射到 `gen_ai.*`。

Agent 轨迹常用属性（均 Development；名称以官方 registry 为准，勿自造）：

| 用途 | 属性键 | 备注 |
|------|--------|------|
| 操作类型 | `gen_ai.operation.name` | 预定义值含 `chat`、`invoke_agent`、`execute_tool`、`embeddings`、`retrieval` 等 |
| Provider | `gen_ai.provider.name` | 如 `openai`、`anthropic`；取代已弃用的 `gen_ai.system` |
| 请求/响应模型 | `gen_ai.request.model`、`gen_ai.response.model` | 请求模型在可用时 Conditionally Required |
| Token | `gen_ai.usage.input_tokens`、`gen_ai.usage.output_tokens` | 取代 `prompt_tokens` / `completion_tokens`；input **含** cached |
| 缓存细分 | `gen_ai.usage.cache_read.input_tokens`、`gen_ai.usage.cache_write.input_tokens` | Recommended when applicable；计入 input 总量 |
| 推理 token | `gen_ai.usage.reasoning.output_tokens` | 计入 output 总量 |
| 结束原因 | `gen_ai.response.finish_reasons` | string[] |
| 工具 | `gen_ai.tool.name`（Required on execute_tool）、`gen_ai.tool.call.id`、`gen_ai.tool.type` | args/result 为 **Opt-In**（敏感） |
| 正文/消息 | `gen_ai.input.messages`、`gen_ai.output.messages`、`gen_ai.system_instructions`、`gen_ai.tool.definitions` | 均为 **Opt-In**；默认勿上生产全文 |

错误类可叠用核心约定中已 **Stable** 的 `error.type`（及可选 `server.address` / `server.port`）。完整表与 span 形状见 [std-otel-genai-semantic-conventions](../../references/docs.md#std-otel-genai-semantic-conventions)；复核笔记见 [evergreen-2026-10-otel.md](../../docs/evergreen-2026-10-otel.md)。

### 4.3 采样

| 场景 | 建议 |
|------|------|
| 错误 / 预算耗尽 / HITL | **100%** 保留轨迹引用 |
| 金丝雀 / 高价值租户 | 高采样或全量 |
| 成功短 run | 头部采样 + **全量 metric/审计计数** |
| 敏感工具 | 采样与 **脱敏管道** 绑定 |

采样丢的是原文附件，不是「是否发生过」。

---

## 5. 工具审计（Tool Audit）

工具是副作用边界。审计证明：**谁**在**何种授权**下调用了**哪个工具**；参数/结果是否合规；失败/重试/补偿可追溯；事后能回答「有没有对客户 X 执行退款？」。

与 [Tool Use](../../core/tool-use/tool-use.md) §8.1 对齐的最小字段：`run/step/tool_call_id`、`tool_name`、`actor`、`authz{decision,rule_id,scopes}`、args/result（`redacted | object_ref` + digest）、timing、`side_effect_class`、`resource_ids`、`idempotency_key`。

| | 只读 | 写/外部副作用 |
|--|------|----------------|
| 保留 | 摘要 + digest | **完整审计**（仍脱敏） |
| 回放 | 可打桩/重查 | **默认不自动重放** |
| 告警 | 延迟/错误率 | 未授权、幂等冲突、补偿失败 |

完整性：鉴权后、执行前写 `tool.intent`，完成后写 `tool.result`；**deny 也记**；异步用同一 `tool_call_id` 串 `pending→completed`；审计 **append-only + 权限分离**。

---

## 6. 上下文 Token 图

Context Engineering 负责装配；Observability 负责 **度量**每次 model call 的窗口账本。

建议分区：`system`、`tools_schema`、`history`、`memory`、`rag`、`observations`、`scratch/plan`、`other`。另记：`window_limit`、`used/evicted_tokens`、`compaction_count`、`truncated_events`。

形态：按 step 的分区堆叠条 + 累计 cost/tokens 折线，标注 compaction / overflow / budget_warn。

用途：解释成本台阶；发现 Observation Explosion；核对 Context 政策是否真降窗。  
「压缩是否损伤任务成功」→ Evaluation；此处只保证 **账本可核对**。

计数：以 **provider usage** 为计费真相；本地估算器做装配前校验；cached tokens 单列；多模态折算 **TODO: Verify**。

---

## 7. 回放（Replay）

| 层级 | 行为 | 用途 |
|------|------|------|
| **L0** | 只读展示已存 model/tool I/O | 人审、纠纷（默认必做） |
| **L1** | 冻结上下文再调模型（非确定性） | 采样方差（须标注） |
| **L2** | 沙箱重跑只读/模拟写 | 调试工具 |
| **L3** | checkpoint + 环境快照 | 罕见、贵 |

默认产品：**L0 + 选择性 L2**。

最小 **Replay Bundle**：版本钉扎（agent/prompt/tool/policy/model/**脱敏策略**）；逐步 context/model/tool 引用；checkpoint 序列与 `termination_reason`；采样参数/`seed`。

非确定性：温度>0 则 L1 不可比特级复现——比结构（工具选择/关键槽位）；检索用当时夹具或标 `live_refresh`；写操作默认 **dry-run**，真实重放需人工闸门 + 幂等键。

Checkpoint（Loop）= **可写恢复**；Replay（Obs）= **不可变证据**。共享 ID，存储权限分离。

---

## 8. 用户反馈信号

| 类型 | 例子 | Obs 用法 |
|------|------|----------|
| 显式 | 👍👎、评分、举报、纠正 | 事件挂 `run_id`/`span_id` |
| 隐式 | 重试、放弃、人工接管 | 弱标签，仅线索/抽样 |
| 运营 | 工单升级、退款争议 | 高优先关联审计 |

约束：反馈是 **事件**，不改写历史轨迹；显式标签可复制进 Evaluation；隐式噪声大，**不单独当成功指标**；纠正文本按 transcript 脱敏。

闭环：`👎 → 诊断清单 →（抽样）Eval 失败集 → 同 termination_reason 簇告警`。  
**有反馈系统 ≠ 有评测体系。**

---

## 9. 与 Agent Loop 挂钩

Observability 不是外挂 log；必须在 Loop 控制点 **强制发射**。

| Loop 拍节 | 发射 | 关键字段 |
|-----------|------|----------|
| run 创建 | `run.start` | versions、预算、tenant |
| `build_context` 后 | `context.built` | token 分区、驱逐摘要 |
| model 前后 | model span | usage、finish_reason |
| 解析 tool_calls | `decision.tools` | names、并行度 |
| 鉴权 | `tool.authz` | allow/deny/HITL |
| 工具执行 | tool spans | §5 审计 |
| checkpoint | `state.checkpoint` | checkpoint_id |
| 预算 | `budget.tick` / warn | 剩余与阈值 |
| 终止 | `run.end` | reason、totals |
| HITL | `hitl.*` | 等待原因、决议 |

挂钩示意：

```text
run.start → loop:
  context.built(token 分区) → model_call span →
  (final? run.end) :
  for tool: authz → (deny | hitl | tool span+审计) →
  checkpoint → budget.tick → (stop? run.end(reason))
```

失败仍须可观测：空响应/JSON 失败/工具超时要有 **error_class**；`cancellation` 与 `budget_exhausted` 为一等原因码；子 Agent 用 child trace，根 run 留边摘要与预算贡献。

遥测失败 **不得拖死 Loop**（异步有界队列），但必须暴露丢失 metric；写工具关键审计可在可接受延迟内同步落盘。

---

## 10. 隐私红线

**默认可观测 ≠ 默认可明文存储。** 冲突时：先满足红线，再降级证据粒度。

| 类别 | 处理 |
|------|------|
| 密钥/cookie/私钥 | **禁止**进日志/span；检测器扫描 |
| 支付卡/证件/生物特征 | 禁止或强加密 + 极短 TTL + 严 ACL |
| 用户 PII | 脱敏/令牌化；角色可见 |
| 工具返回密钥材料 | 执行器剥离后再引用 |
| 跨租户 | 查询强制 tenant 谓词；导出审批 |
| 机密系统指令 | 对普通支持角色遮罩 |

分级存储：

```text
L_meta   : IDs、版本、计数、原因码、延迟     → 宽 ACL、长保留
L_digest : 参数/结果哈希、token 分区         → 中 ACL
L_body   : 脱敏后的输入输出正文             → 窄 ACL、较短保留
L_raw    : 原始敏感正文（若必须）           → 保险库、极短 TTL、双人控制
```

回放默认读 `L_body`；读 `L_raw` 须审计「谁看了」。保留按合同/法规；右被遗忘时删正文引用，动作元数据可合法分离保留；Eval/训练导出另走同意管道。

违规典型：prompt 全文进 span 属性；错误回显 secrets；demo 灌生产轨迹；全员 raw 只读。

---

## 11. 失败诊断手册

**Obs 视角**：先证据后假设。「好不好」仍归 Eval。

流程：定位 `run_id` → 读 `termination_reason/status` → step 时间线 → 该步 token 图 + `finish_reason` → tool authz/errors → 必要时 L0（写不自动重放）→ 归类（控制流/工具/上下文/模型/策略/基建）→ 修复或抽样进 Eval。

| 模式 | 信号 | 先看 | 线索归属 |
|------|------|------|----------|
| 空转 | 步数高、args digest 重复、无进展 | decision 摘要/计划 | Loop 预算/熔断；Obs 负责看见 |
| 错工具/参数 | schema/4xx、版本不匹配 | tool span + 模型输出引用 | Tool Use ACI/schema |
| 观察膨胀 | observations 暴涨、频繁 compaction | token 图、result bytes Top-N | Context Engineering |
| 授权环 | 反复 deny 仍被点名 | rule_id、deny observation | 清晰回灌/收紧工具集 |
| 预算耗尽 | `budget_*`、成本陡峭 | span 成本贡献、重试风暴 | 配额/重试 |
| HITL 卡住 | `waiting_human` 超时 | 等待原因、通知 SLA | Loop 超时降级 |
| 成功但 👎 | succeeded + 负反馈 | L0 答案与副作用 | Eval 定成功；Obs 交证据 |
| 路由异常 | 同 run model_id 变化+错误尖峰 | model_route、JSON 严格性 | Architecture 适配层 |

**适合 Obs 告警**：工具错误率、鉴权风暴、预算耗尽、HITL 超时、轨迹写入失败、脱敏管道失败。  
**不适合单独当 Obs 告警**：离线集分数波动（Eval 门禁）。

---

## 12. 生产架构要点

```text
Loop Workers → Emitter SDK → Collector
  → Trace Store / Audit Log / Blob(脱敏正文) / Metrics
Replay UI（只读）→ Trace + Blob
```

要求：Emitter 异步有界；关键写审计可同步；Debug UI 只读；导出 Eval 的 pipeline 显式、可关、可审计；内部语义模型可移植，避免 UI 厂商 schema 反推核心。

---

## 13. Observability 自身的失败模式

| 失败 | 症状 | 缓解 |
|------|------|------|
| 观测不全 | 无 tool span | 执行器强制埋点；契约测试 |
| 观测过重 | Loop 延迟升 | 异步、采样、正文外置 |
| 高基数爆炸 | metric 成本炸 | name/label 规范审查 |
| 隐私事故 | PII 进明文 | 扫描器、分级、演练 |
| 假回放 | 环境漂移当可复现 | 版本钉扎、夹具、标明 L1 |
| 双真相 | trace≠checkpoint | 共享 ID；恢复以 State 为准 |
| 厂商锁定 | 只能某 UI | 内部语义 + 可移植导出 |
| 轨迹投毒 | 假 span 掩盖调用 | Emitter 鉴权；应用日志与审计分离 |

---

## 14. 权衡

| 更强观测 | 代价 |
|----------|------|
| 全量正文 | 存储、隐私、法律风险 |
| 精细 token 分区 | 装配路径插桩成本 |
| 高采样 L1 | 模型费用 |
| 实时 UI | 平台复杂度 |
| 严格脱敏 | 排障信息变少 |

**默认建议**：元数据与审计全量；正文按风险分级；错误与敏感动作全量保留引用；成功路径可采样。复杂度扩张须与证据能力匹配——看不见就不该上自动写/多 Agent（呼应 Anthropic「按需增加复杂度」）。

---

## 15. Interview Questions

### Beginner

**Q1. Agent Observability 解决什么问题？**  
- **短答**：让 run 的步骤、工具副作用、资源消耗与终止原因可追踪可诊断。  
- **详解**：补齐传统 APM 对动态多步 Agent 的语义缺口。  
- **追问**：与普通微服务 tracing 最大差别？  
- **常见错误**：只打最终答案日志。  
- **相关**：§1。

**Q2. Evaluation 与 Observability 如何分工？**  
- **短答**：Eval 判好坏；Obs 可观测。  
- **详解**：轨迹可被 Eval 消费，但不在 Obs 定义成功。  
- **追问**：用户 👎 应进哪边？  
- **常见错误**：用成功率替代轨迹完整性。  
- **相关**：§2。

**Q3. Run / Step / Span 分别是什么？**  
- **短答**：Run=一次 Loop；Step=一轮；Span=有起止工作单元。  
- **详解**：model/tool 常为 step 下子 span。  
- **追问**：并行工具如何建模？  
- **常见错误**：一步多模型调用无关联。  
- **相关**：§3–4。

**Q4. 为什么 span name 不能放用户问题？**  
- **短答**：高基数与隐私双杀。  
- **详解**：问题进脱敏正文引用。  
- **追问**：metric label 有何限制？  
- **常见错误**：把 run_id 当 metric label 风暴。  
- **相关**：§4、§10。

**Q5. 工具审计最少记什么？**  
- **短答**：谁、何工具、鉴权、参数/结果摘要或引用、时延、副作用类。  
- **详解**：deny 也要记。  
- **追问**：只读与写工具差异？  
- **常见错误**：只在成功时记日志。  
- **相关**：§5。

**Q6. 什么是上下文 token 图？**  
- **短答**：每次模型调用各分区 token 占用的账本/可视化。  
- **详解**：用于发现观察膨胀与成本台阶。  
- **追问**：计费以谁为准？  
- **常见错误**：只看总 tokens 不看分区。  
- **相关**：§6。

**Q7. L0 回放与 L1 回放差别？**  
- **短答**：L0 只读历史 I/O；L1 再调模型故非确定性。  
- **详解**：写工具默认不自动重放。  
- **追问**：何时需要沙箱 L2？  
- **常见错误**：宣称比特级复现温度采样。  
- **相关**：§7。

**Q8. 显式反馈与隐式反馈？**  
- **短答**：显式=评分/举报；隐式=重试/放弃等行为。  
- **详解**：皆为事件；成功定义仍在 Eval。  
- **追问**：隐式信号噪声如何控制？  
- **常见错误**：把停留时长当质量真理。  
- **相关**：§8。

**Q9. Loop 上最关键的 Obs 挂钩点？**  
- **短答**：context 建成、model、authz、tool、checkpoint、terminate。  
- **详解**：缺一则诊断断链。  
- **追问**：遥测失败能否阻塞 Loop？  
- **常见错误**：只在外层 middleware 打点。  
- **相关**：§9。

**Q10. 举三条隐私红线。**  
- **短答**：密钥禁入日志；PII 脱敏；跨租户查询隔离。  
- **详解**：分级存储 L_meta→L_raw。  
- **追问**：排障与合规冲突怎么办？  
- **常见错误**：生产 raw 对全员开放。  
- **相关**：§10。

### Intermediate

**Q11. 如何设计 termination_reason 枚举？**  
- **短答**：机器可读、互斥优先、可扩展；区分 success/cancel/budget/policy/tool/model。  
- **详解**：供告警与 Eval 切片共用码表。  
- **追问**：多原因如何记录？  
- **常见错误**：自由文本当唯一原因。  
- **相关**：§3、§11。

**Q12. 采样丢弃了什么、不能丢什么？**  
- **短答**：可丢正文附件；不能丢计数与关键审计。  
- **详解**：错误与写操作提高保留。  
- **追问**：金丝雀如何配？  
- **常见错误**：错误路径也被采样掉。  
- **相关**：§4.3。

**Q13. 异步工具如何追踪？**  
- **短答**：同一 tool_call_id 贯穿 pending→completed。  
- **详解**：决策不得假设 pending 已成功。  
- **追问**：UI 如何展示？  
- **常见错误**：完成时新建无关联 id。  
- **相关**：§5、§9。

**Q14. 如何用 token 图定位 Observation Explosion？**  
- **短答**：看 observations 分区占比与 result bytes Top-N 工具。  
- **详解**：对齐 Context 归一化政策版本。  
- **追问**：与「模型变贵」如何区分？  
- **常见错误**：只怪模型单价。  
- **相关**：§6、§11。

**Q15. Replay Bundle 必须钉扎哪些版本？**  
- **短答**：agent/prompt/tool schema/policy/model（及脱敏策略）。  
- **详解**：否则回放不可解释。  
- **追问**：缺 checkpoint 能否 L0？  
- **常见错误**：只存最终 transcript。  
- **相关**：§7。

**Q16. 写操作回放的安全闸门？**  
- **短答**：默认 dry-run；真实重放需人工+幂等键+环境隔离。  
- **详解**：审计「谁触发了重放」。  
- **追问**：补偿事务如何验证？  
- **常见错误**：调试按钮直连生产写。  
- **相关**：§7、§5。

**Q17. 用户 👎 后 Obs 流水线做什么？**  
- **短答**：挂载诊断清单、保留证据、可选抽样进 Eval。  
- **详解**：不改写历史 span。  
- **追问**：如何防刷反馈？  
- **常见错误**：用反馈分替代任务成功定义。  
- **相关**：§8、§2。

**Q18. 遥测丢失如何被发现？**  
- **短答**：emit_failures metric、结束校验应有 span 数、抽样对账。  
- **详解**：关键审计可同步写。  
- **追问**：对账缺 tool span 怎么办？  
- **常见错误**：假定异步队列永成功。  
- **相关**：§9、§12–13。

**Q19. 空转在 Obs 上的检测信号？**  
- **短答**：步数高、args digest 重复、无进展事件。  
- **详解**：告警阈值按任务类型校准。  
- **追问**：合法长探索如何降误报？  
- **常见错误**：一律按步数杀。  
- **相关**：§11。

**Q20. 为什么 State 与 Trace 要分离？**  
- **短答**：State 用于恢复；Trace 用于证据；权限与可变性不同。  
- **详解**：共享 ID，不以 Trace 覆盖 State。  
- **追问**：不一致时听谁的？  
- **常见错误**：只在日志里「重建状态」。  
- **相关**：§1.3、§7。

### Advanced

**Q21. 设计多租户下的轨迹查询隔离。**  
- **短答**：每条记录强制 tenant_id；查询注入谓词；导出审批。  
- **详解**：角色对 L_body/L_raw 分级。  
- **追问**：平台运维破窗流程？  
- **常见错误**：应用层过滤当唯一防线。  
- **相关**：§10、§12。

**Q22. 子 Agent 委派的 trace 关联模型。**  
- **短答**：子 run child trace；父保留边摘要与预算贡献。  
- **详解**：委派环检测事件。  
- **追问**：成本如何归账到根目标？  
- **常见错误**：子轨迹孤立无法聚合。  
- **相关**：§9、Architecture。

**Q23. 如何对 Obs 管道做混沌测试？**  
- **短答**：注入 collector 延迟/丢弃、脱敏失败，断言 Loop 仍安全且有丢失信号。  
- **详解**：写路径审计不得静默丢。  
- **追问**：在生产比例？  
- **常见错误**：只测 UI 展示。  
- **相关**：§13。

**Q24. 脱敏与可调试的张力如何产品化？**  
- **短答**：分级存储+按需提权+字段遮罩+保留期分层。  
- **详解**：默认视图覆盖多数排障。  
- **追问**：提权 SLA？  
- **常见错误**：二选一走极端。  
- **相关**：§10、§14。

**Q25. GenAI span 属性映射到内部模型注意什么？**  
- **短答**：内部稳定 schema 为真相；导出适配器转换到 `gen_ai.*`。  
- **详解**：OTel GenAI 约定整体仍为 Development；弃用键（如 `prompt_tokens`、`gen_ai.system`）勿写入新代码。  
- **追问**：社区约定变更怎么办？适配器版本钉死 + 双写过渡。  
- **常见错误**：业务代码直接打 `gen_ai.*` 或厂商属性键当主键。  
- **相关**：§4.2。

**Q26. 成本爆炸的 Obs 归因框架？**  
- **短答**：steps × tokens/step × price + tool；按分区与工具 Top-N。  
- **详解**：区分重试风暴与窗口膨胀。  
- **追问**：如何自动熔断？  
- **常见错误**：只看月账单。  
- **相关**：§6、§11。

**Q27. 轨迹完整性的 SLO 怎么定？**  
- **短答**：如「错误 run 关键 span 齐全率 ≥ 99.9%」；与延迟 SLO 分离。  
- **详解**：完整性优先于全量正文。  
- **追问**：如何测量齐全率？  
- **常见错误**：用 UI 满意度代替完整性。  
- **相关**：§12、§14。

**Q28. HITL 等待如何观测与告警？**  
- **短答**：hitl.wait/resolve、等待时长直方图、超时原因。  
- **详解**：区分人未响应与通知失败。  
- **追问**：与业务 SLA 如何对齐？  
- **常见错误**：只监控模型延迟。  
- **相关**：§9、§11。

**Q29. 策略版本变更如何观测？**  
- **短答**：run 钉扎 policy_version；对比 deny 率、工具分布、预算耗尽率。  
- **详解**：与 Eval 门禁联动但不替代。  
- **追问**：影子流量如何标？  
- **常见错误**：策略热更无版本字段。  
- **相关**：§3、§2。

**Q30. 如何防止调试 UI 成为攻击面？**  
- **短答**：SSO、细粒度 ACL、只读、审计查看、禁导出 raw。  
- **详解**：预发与生产分离。  
- **追问**：复制 replay bundle 出网？  
- **常见错误**：同一 admin 万能钥匙。  
- **相关**：§10、Security。

### Expert

**Q31. 形式化「可审计性」对写工具的要求。**  
- **短答**：intent/result 双记录、幂等键、资源 ID、防篡改、时钟可信。  
- **详解**：满足事后举证。  
- **追问**：与金融对账对接？  
- **常见错误**：信任模型自述「已退款」。  
- **相关**：§5。

**Q32. 右被遗忘与审计保留的冲突解。**  
- **短答**：内容删除与「动作发生过」的合法保留分离。  
- **详解**：L_body 可删、L_meta 可留（依法务）。  
- **追问**：未决诉讼 hold？  
- **常见错误**：一刀切全删审计。  
- **相关**：§10。

**Q33. 观测数据被投毒（假 span）的威胁。**  
- **短答**：Emitter 鉴权、worker 身份、审计与应用日志分离、完整性校验。  
- **详解**：攻击者可能掩盖工具调用。  
- **追问**：如何检测缺口？  
- **常见错误**：浏览器直写生产 trace。  
- **相关**：§12–13、Security。

**Q34. 大规模下 token 分区统计的性能设计。**  
- **短答**：装配路径增量计数；异步上送；正文外置。  
- **详解**：热路径避免深拷贝全文。  
- **追问**：如何保证分区和为 usage？  
- **常见错误**：每次 stringify 全上下文。  
- **相关**：§6、§12。

**Q35. 跨区域 run 的 trace 关联与驻留。**  
- **短答**：区域本地写、摘要全局索引；正文按法域驻留。  
- **详解**：回放就近读。  
- **追问**：故障转移丢轨迹？  
- **常见错误**：全球复制 raw PII。  
- **相关**：§10、Infra。

**Q36. Obs 自动诊断卡片的规则引擎边界。**  
- **短答**：规则输出假设与证据链接，不自动改生产策略。  
- **详解**：与 Eval 自动打分分离。  
- **追问**：规则如何版本化？  
- **常见错误**：观测规则直接杀流量无审计。  
- **相关**：§11。

**Q37. 多模态 Agent 的可观测增量。**  
- **短答**：模态类型、折算 tokens、附件哈希、截图 PII。  
- **详解**：回放需媒体权限与 TTL。  
- **追问**：视频帧如何抽稀？  
- **常见错误**：截图永久明文存日志。  
- **相关**：§6、§10。

**Q38. 可观测性投资如何约束 Agent 复杂度扩张？**  
- **短答**：看不见就不许加多 Agent/自动写；复杂度与证据能力匹配。  
- **详解**：轨迹完备作发布门槛之一。  
- **追问**：门槛如何量化？  
- **常见错误**：先上 Multi-Agent 再补日志。  
- **相关**：§1、§14。

**Q39. 对抗「轨迹灌水」骗过人工审查。**  
- **短答**：结构化进展事件、副作用强制审计、与外部系统对账。  
- **详解**：文本叙事不可作唯一证据。  
- **追问**：模型伪造 tool_result？  
- **常见错误**：只读聊天记录审案。  
- **相关**：§5、§11。

**Q40. Review 开源框架 Obs 是否生产级。**  
- **短答**：对照挂钩点、审计、脱敏、回放层级、丢失信号、租户隔离打分。  
- **详解**：缺写审计与隐私则不可上生产。  
- **追问**：回调式 vs 中间件式埋点？  
- **常见错误**：有 Dashboard 截图即合格。  
- **相关**：§9、§13。

### System Design

**Q41. 设计生产级 Agent Observability 平台。**  
- **短答**：Emitter、collector、trace/audit/blob/metric、脱敏、Replay UI、导出 Eval。  
- **详解**：画出丢失与破窗路径。  
- **追问**：多区域与保留期？  
- **常见错误**：直接打第三方 SaaS 明文 prompt。  
- **相关**：§12。

**Q42. 设计 Coding Agent 的轨迹与沙箱审计。**  
- **短答**：diff 摘要、测试命令与退出码、sandbox id；禁存 secrets。  
- **详解**：与测试终止联动展示。  
- **追问**：巨 diff 如何存？  
- **常见错误**：整仓打包进日志。  
- **相关**：§5、Coding Agent、Sandbox。

**Q43. 设计客服退款场景的可举证 Obs。**  
- **短答**：查询只读摘要；退款 intent/result/幂等键/HITL 全链路。  
- **详解**：纠纷 L0 + 支付系统对账。  
- **追问**：用户要求删对话？  
- **常见错误**：只靠聊天记录证明退款。  
- **相关**：§5、§7、§10。

**Q44. 从 printf 调试迁到语义 Obs 的里程碑。**  
- **短答**：P0 id → P1 原因码与 usage → P2 脱敏正文引用 → P3 token 图 → P4 回放与告警。  
- **详解**：每阶段有验收。  
- **追问**：上生产不可妥协项？  
- **常见错误**：先做炫酷火焰图。  
- **相关**：§14。

**Q45. 设计 Obs↔Eval 的安全数据接口。**  
- **短答**：只读导出、强制脱敏、标签回写分离；Eval 禁读 L_raw。  
- **详解**：抽样与同意记录。  
- **追问**：在线分数如何避免内容泄漏到宽 ACL？  
- **常见错误**：Eval 集群挂载全量生产日志盘。  
- **相关**：§2、§8、§10。

---

## 16. Cross References

- [Agent Loop](../../core/agent-loop/agent-loop.md) · [Tool Use](../../core/tool-use/tool-use.md)  
- [Context Engineering](../../core/context-engineering/context-engineering.md) · [Agent Architecture](../../core/agent-architecture/agent-architecture.md)  
- [Evaluation](../evaluation/README.md) · [Security](../security/README.md) · [Infrastructure](../infrastructure/README.md)

---

## 17. References

- [article-anthropic-building-effective-agents-2024](../../references/articles.md#article-anthropic-building-effective-agents-2024)  
- [std-otel-genai-semantic-conventions](../../references/docs.md#std-otel-genai-semantic-conventions)  
- Evergreen：[evergreen-2026-10-otel.md](../../docs/evergreen-2026-10-otel.md)  
- 机制依赖：[Agent Loop](../../core/agent-loop/agent-loop.md) · [Tool Use](../../core/tool-use/tool-use.md) · [Context Engineering](../../core/context-engineering/context-engineering.md)  
- 边界：[Evaluation README](../evaluation/README.md)

---

## 18. Self-review / TODOs

- [x] Eval vs Obs 边界明确；未复制 Evaluation 指标方法论长文  
- [x] traces/spans、工具审计、token 图、回放、反馈、Loop 挂钩、隐私、诊断已覆盖  
- [x] 45 面试题  
- [x] Evergreen 2026-10-07 — OpenTelemetry GenAI semantic conventions 字段名/Development 状态（机制，非价目）；见 §4.2  
- [ ] TODO: Verify — 各厂商 usage/缓存 token 与多模态计价细节  
