# Infrastructure（Canonical）

> **一句话主题**：Production Infrastructure 是把 Agent Loop **服务化、可排队、可配置、可多环境与多租户运行**的平台层，并把模型调用统一接到 Inference Gateway。
>
> **状态**：Canonical · **Last reviewed**: 2026-10  
> **受众**：设计/落地 Agent 平台拓扑、容量与发布面的工程师  
> **硬前置**：[Agent Loop](../../core/agent-loop/agent-loop.md)、[Agent Architecture](../../core/agent-architecture/agent-architecture.md)  
> **软前置**：[LLM Inference](../../runtime/llm-inference/README.md)、[Observability](../observability/README.md)、[Sandbox](../../runtime/sandbox/README.md)

---

## 范围 / 非目标

**范围**：服务化 Agent 参考拓扑；异步 run 与队列；配置/特性开关与密钥；多环境；限流与配额；多租户隔离与计费钩子；与 Inference Gateway 的衔接契约；失败模式与运维实践；面试题。

**非目标**：

- 通用 Kubernetes / 某云控制台教程
- 评测方法论与指标体系正文 → [`evaluation`](../evaluation/README.md)
- Security 威胁模型与注入防护正文 → [`security`](../security/README.md)（本文只保留平台策略钩子）
- 推理侧批处理、KV cache、路由算法深文 → [`llm-inference`](../../runtime/llm-inference/README.md)
- 沙箱实现手册 → [`sandbox`](../../runtime/sandbox/README.md)
- 架构组件职责定义正文 → [`agent-architecture`](../../core/agent-architecture/agent-architecture.md)

---

## 1. 知识模型：Infrastructure 解决什么

### 1.1 定义

**Agent Infrastructure** 支撑「用户意图 → 可恢复 Run → 多次模型/工具调用 → 可审计结果」的**平台能力**。它不决定如何推理，而决定：

1. 请求如何进入（API / 事件）
2. Run 如何调度、重试、取消
3. Worker 如何水平扩展且无状态
4. 配置与密钥如何按环境/租户注入
5. 容量如何被限流与配额切分
6. 模型流量如何经 Gateway 统一出口

### 1.2 与相邻模块

| 模块 | 回答的问题 | 本文 |
|------|------------|------|
| Agent Architecture | 控制面/数据面、Runtime 切分 | 挂钩，不复制 |
| Agent Loop | 单次 Run 内决策循环 | 挂钩 |
| LLM Inference | TTFT/批处理/前缀缓存 | 只定义 Gateway 契约 |
| Observability | 如何观测 | 埋点钩子，不写遥测模型 |
| Security | 威胁与防护 | 只列平台控制点 |
| Evaluation | 好不好 | 不写 |

### 1.3 Mental Model

```text
Client
  → API Gateway（鉴权 / 入站限流）
  → Run Control Plane（创建/查询/取消）
  → Queue → Agent Workers（无状态 Loop）
       ├→ State Store（checkpoint）
       ├→ Tool Runtime / Sandbox
       └→ Inference Gateway → 模型后端
  → Config / Secrets / Feature Flags
  → Quota / Rate Limit / Tenancy
```

**不变量**：Worker 可随时被杀；真源在 State Store + Queue 语义。SSE/WS fanout 宜独立于执行 Worker。

---

## 2. 服务化 Agent：参考拓扑

### 2.1 Demo → Production

| 阶段 | 形态 | 典型症状 |
|------|------|----------|
| Demo | 单进程同步 `while` | 超时、无法扩展、状态丢 |
| MVP | API + 同步 Worker | 长任务打满连接；难取消 |
| Production | API + Queue + 无状态 Worker + 外置 State | 可恢复、可扩展、可多租户 |

服务化 = 把 **Run 生命周期**从 HTTP 连接寿命解耦，不是「Docker 包一层」。

### 2.2 组件职责

| 组件 | 职责 | 非职责 |
|------|------|--------|
| **API Gateway** | TLS、鉴权、入站限流、路由 | 跑 Loop |
| **Run Controller** | 创建/查询/取消入口 | 直接调模型 |
| **Work Queue** | 缓冲、优先级、延迟、死信 | 业务逻辑 |
| **Agent Worker** | 租约执行 Loop、写 checkpoint | 长期持有租户密钥明文 |
| **State Store** | Run 状态、摘要、预算计数 | 当日志仓库 |
| **Inference Gateway** | 模型出口、路由、降级、计量 | 工具执行 |
| **Config / Secrets** | 版本化配置与密钥注入 | Prompt 语义设计 |

### 2.3 API 契约

**同步**：短路径、秒级。**异步**：多步 Agent、工具等待、HITL、批量。

```text
POST /v1/runs → 202 + run_id
GET  /v1/runs/{id} → status, progress, result_ref
POST /v1/runs/{id}/cancel
GET  /v1/runs/{id}/events (SSE/WS)
```

### 2.4 Worker 无状态化

热缓存允许；重启须能从 `run_id` 恢复：`status/step/budgets/plan_version`、checkpoint 指针、`config_rev/prompt_rev`、工具幂等键。

---

## 3. 队列与异步 Run

### 3.1 为何几乎总需要队列

Agent 延迟重尾（工具超时、重试、HITL、长推理）。队列提供削峰、重试边界、优先级、租户隔离。

### 3.2 状态机（平台视角）

```text
accepted → queued → running → (waiting_tool | waiting_human | waiting_model)
         ↘ cancelled
running / waiting_* → succeeded | failed | timed_out | dead_lettered
```

| 状态 | 平台动作 |
|------|----------|
| `queued` | 计排队延迟 |
| `running` | 租约 + 心跳 |
| `waiting_human` | 不占执行并发；须超时 |
| `waiting_model` | 与推理队列联动 |
| `dead_lettered` | 告警；人工处置 |

### 3.3 租约、心跳与幂等

1. 出队获取 **lease** + `worker_id`
2. heartbeat；过期可接管
3. 副作用工具：`idempotency_key = run_id + tool_call_id`
4. 取消写入 State；Worker 在安全点检查并传播到 Gateway

### 3.4 重试分层

| 层 | 可重试 | 不可盲目重试 |
|----|--------|--------------|
| 队列投递 | Worker 崩溃、瞬时 DB | 已成功副作用 |
| 工具 | 5xx/超时（策略允许） | 4xx、权限拒绝 |
| 模型 | 429/5xx + 退避 | 内容策略拒绝（换策略） |

平台重试针对**基础设施瞬时故障**；业务幂等由 Tool Use/Loop 定义，平台提供预算计数。

### 3.5 多队列与背压

切分：`interactive` / `batch` / `replay`，可按 `tenant_tier` 加权（WFQ）。避免全局 FIFO 饿死交互流量。

```text
Gateway 饱和 → Worker 429/变慢 → waiting_model 堆积
  → Quota 限制入队 → API 429 + Retry-After
```

背压须回到入站；只堵 Worker 内存会 OOM/静默丢。

---

## 4. 配置、特性开关与密钥

### 4.1 分层

| 层 | 例子 | 发布 |
|----|------|------|
| 环境 | 区域、队列名、DSN 指针 | 部署时 |
| 产品 | 默认模型、最大步数、工具集 | 配置服务 + 版本 |
| 租户 | 配额、驻留、品牌片段 | 租户管理面 |
| 开关 | 灰度、降级 | Feature Flag |
| 密钥 | API key、签名密钥 | Secrets Manager |

Prompt/工具 schema/模型路由应**配置化**，勿写死镜像。

### 4.2 Run 绑定版本

启动冻结：`config_rev`、`prompt_rev`、`tool_manifest_rev`、`model_route_rev`、flag 快照哈希。  
热更新**默认不**改在途 Run，否则事故不可复现。

### 4.3 Feature Flags

**适合**：换模型、降步数、切工具、灰度租户。  
**不适合**：当配置库；授权真源仍在 ACL/策略引擎。

### 4.4 密钥（平台要求，非威胁模型）

1. 不入库明文；短时取用  
2. 环境隔离（dev 密钥禁用于 prod 数据面）  
3. BYOK 按 `tenant_id` 检索  
4. 轮换双窗口；轨迹记 `key_id`  
5. Worker 最小注入当前任务密钥集  

脱敏见 Observability；滥用场景见 Security。

### 4.5 发布流水线

```text
提案 → schema/预算校验 → stage 金丝雀 → 观察（Obs 信号）→ 放量 → 一键回滚 config_rev
```

配置回滚应比代码回滚更轻、更常见。

---

## 5. 多环境

| 环境 | 数据 | 模型 | 副作用 |
|------|------|------|--------|
| `dev` | 合成/脱敏 | 小模型/mock | 假工具 |
| `stage` | 生产形状脱敏 | 近 prod 路由 | 受限/影子 |
| `prod` | 真实租户 | 正式容量 | 完整策略 |

**必须隔离**：IdP/密钥 path、存储、队列前缀、模型账号、stage 出站写。  
**晋升**：不可变镜像标签；同 schema 不同值；高危工具显式晋升；禁真实 PII 灌 dev。  
**Shadow/Replay**：无生产副作用；采样标记；评测设计不在本文。

---

## 6. 限流、配额与成本控制

### 6.1 两层限流

| 层 | 保护 | 手段 |
|----|------|------|
| 入站 API | 控制面/队列 | RPS、并发、token bucket |
| 出站 Inference/Tools | 模型池与第三方 | 租户 QPS、token/min、工具/min |

成本单位是 **tokens × steps × 工具调用**，不是 HTTP 请求。

### 6.2 配额维度

每 `tenant_id`（可再拆 project/user）：`max_concurrent_runs`、`max_queued_runs`、`tokens_per_day|minute`、`tool_calls_per_day`、`max_steps|wall_time_per_run`、`max_cost_units_per_day`。  
超额：`reject` / `queue_slow` / `degrade_model` / `truncate_tools`。

### 6.3 公平与三重预算

共享池：硬隔离配额，或 WFQ + 有界借用；重试风暴临时降权。

| 概念 | 归属 | 作用域 |
|------|------|--------|
| Loop Budget | Agent Loop | 单 Run |
| Platform Quota | Infrastructure | 跨 Run 聚合 |
| Gateway Rate Limit | Inference Gateway | 模型后端 |

### 6.4 熔断开关

`disable_new_runs`、模型 `fallback`、高成本工具 `deny`、只读工具集。信号来自 Obs（错误率、队列深度、429）；非评测方法论。

---

## 7. 多租户

### 7.1 隔离级别

| 级别 | 做法 | 适用 |
|------|------|------|
| 逻辑 | 共享集群 + 强制 `tenant_id` | 多数 SaaS |
| 强逻辑 + BYOK | 独立 KMS | 企业 |
| 计算池 | 独立队列/Worker | 嘈杂/合规 |
| 专有 | 独立账号/VPC | 强合规 |

按数据敏感度与爆炸半径升级，勿默认专有集群。

### 7.2 上下文传播

```text
tenant_id, project_id, principal_id, region, data_residency,
quota_class, tool_acl_ref, secret_scope
```

路径：鉴权 → Run → Queue → Worker → Tools/Gateway。**禁止**从自然语言解析租户。

### 7.3 数据面与吵闹邻居

State/对象存储/向量库/日志/缓存均须租户命名空间或强制 filter。  
吵闹邻居：分队列、并发帽、毒丸→DLQ、租户熔断。

### 7.4 计费钩子

Gateway/Tool 出口累计 tokens（含 cached）、工具次数、sandbox_ms、queue/model wait。  
计量走 outbox，失败不得静默丢；价格表不在本文。

---

## 8. 与 Inference Gateway 衔接

> 批处理、前缀缓存、流式中断见 [`runtime/llm-inference`](../../runtime/llm-inference/README.md)。本节只定 **对接契约**。

### 8.1 为何不直连厂商 SDK

密钥散落、无统一超时/重试、无法租户计量、无法一键降级、观测不一致。生产 Worker 的 Model Plane **唯一出口**是 Gateway。

### 8.2 请求/响应契约

Worker → Gateway：`run_id/step_id/span_id`、`tenant_id/project_id`、`model_alias`（非裸模型 ID）、`timeout_ms/priority`、可选 `idempotency_key`/`budget_hint`。  
返回归一化：文本/tool blocks、usage、路由命中、降级标记、可重试错误分类。

### 8.3 流式取消与路由归属

解析到完整 tool call 后可取消剩余生成；取消须在 Gateway **释放容量**。  
Config+Gateway：别名与灰度；Gateway：429/5xx failover；租户/Run：是否允许降级；Tool 策略：降级是否改工具集。轨迹记录 `alias → concrete_model`。

### 8.4 容量两级 + Fake Gateway

`Quota(tenant)` → `Gateway(pool)` → Backend。只做一级会不公或集体失败。  
dev 用 Fake Gateway；契约钉归一化 schema。

---

## 9. 端到端数据流与容量

```text
POST /runs → 鉴权/配额 → accepted → 入队 → 202
Worker 租约出队 → 加载 checkpoint/config_rev
Loop: Context → Gateway → Tools → checkpoint/事件 → 预算/取消/配额
终止 → 结果 → 释租约 → 计量；Client GET/SSE
```

| 扩展轴 | 注意 | 容量单位 |
|--------|------|----------|
| API/Controller | 无粘滞；创建幂等 | 入站 RPS |
| Worker | 受模型/工具槽约束 | concurrent runs、sandbox slots |
| State | 防热点租户 | checkpoint IOPS、单 Run 体积 |
| Gateway | 驻留与多区域 | tokens/min、429 率 |

延迟 ≈ `gateway_in + queue_wait + Σ(model+tool) + fanout`。优先减无效 steps → queue_wait → 模型路由 → 工具并行。

**实践**：先异步再多区域；只认 `model_alias`；三重预算；配置/密钥分家；环境独立队列与数据面；租户上下文全链路；死信一等公民；取消+租约同设计；计量 outbox；Fake Gateway 契约测。Sandbox/Security/Obs 只挂控制点，不写威胁模型或评测方法。

---

## 10. Failure Modes

| 症状 | 原因 | 检测 | 缓解 |
|------|------|------|------|
| 卡在 `queued` | Worker 不足/毒消息 | 队列深度、最老年龄 | 扩容、分队列、DLQ |
| 双执行副作用 | 无租约/幂等 | 重复 idempotency | 租约 + 幂等键 |
| 取消无效 | 未轮询/未传 Gateway | cancel→running 时长 | 短租约；传播取消 |
| 环境串数据 | 配置/密钥串用 | 启动 env 校验 | 硬隔离；fail-fast |
| 单租户拖垮池 | 重试风暴 | 每租户 429 | 熔断；重试预算 |
| 配置不可复现 | 未绑 config_rev | 审计缺失率 | 启动冻结快照 |
| Gateway 绕行 | 直连厂商 | 计量空洞 | 出站只放行 Gateway |
| Checkpoint 膨胀 | 全量 observation | 单 Run 体积 | 外置对象 + 指针 |
| HITL 永等 | 无 waiting 超时 | waiting 年龄 | 超时 + 通知 |

---

## 11. 伪代码：最小控制面

```text
algorithm CreateRun(req, auth):
  tenant ← Authenticate(auth)
  AssertQuota(tenant, cost_estimate(req))
  run ← new Run(id, tenant, accepted,
                 config_rev=CurrentConfig(tenant),
                 budgets=ResolveBudgets(tenant, req))
  Persist(run); Enqueue(run.id, SelectQueue(tenant, req.priority))
  return 202, run.id

algorithm WorkerLoop():
  while true:
    job ← Lease(queue, lease_ttl); run ← Load(job.run_id)
    if run.cancelled: Finalize(cancelled); continue
    while not Terminal(run) and HasLease(job):
      if Cancelled(run) or QuotaExceeded(run.tenant): break
      out ← AgentLoop.Step(run, gateway=IG, tools=TR)
      Checkpoint(run, out); EmitEvents(run, out); RenewLease(job)
      if out.wait_human: Suspend(waiting_human); return
    if not Terminal(run): Requeue(run) else Finalize(run)
```

---

## 12. Trade-offs

| 决策 | 收益 | 代价 |
|------|------|------|
| 全异步 Run | 可扩展、可恢复 | 客户端复杂；最终一致 |
| 强租户计算隔离 | 故障隔离 | 利用率低、运维贵 |
| 配置热更新 | 迭代快 | 须快照才能复现 |
| 统一 Gateway | 治理与计量 | 多一跳；关键依赖 |
| 多队列优先级 | 护交互式 | 调度复杂；或饿死低优先 |

---

## 13. Interview Questions

### Beginner

**Q1. 为何生产 Agent 不能只用同步 HTTP 跑完 Loop？**  
- **短答**：延迟重尾，占满连接且难取消/恢复。  
- **详解**：用 202 + run_id + 队列。  
- **追问**：哪些短任务仍可同步？  
- **常见错误**：加大超时当架构。  
- **相关**：§2–3。

**Q2. Worker「无状态」指什么？**  
- **短答**：进程可杀；真源在 State/队列。  
- **详解**：热缓存须可重建。  
- **追问**：checkpoint 最小字段？  
- **常见错误**：状态只在内存。  
- **相关**：§2.4。

**Q3. Inference Gateway 解决什么？**  
- **短答**：模型统一出口：路由、限流、计量、降级。  
- **详解**：Worker 不直连厂商。  
- **追问**：与 API Gateway 区别？  
- **常见错误**：当成普通反向代理讲完。  
- **相关**：§8，llm-inference。

**Q4. 入站限流与模型限流差别？**  
- **短答**：前者护控制面/队列；后者护模型池与成本。  
- **详解**：还要按 token/step 配额。  
- **追问**：只做 RPS 会怎样？  
- **常见错误**：QPS=容量规划完成。  
- **相关**：§6。

**Q5. 多环境至少隔离什么？**  
- **短答**：密钥、数据、队列、模型账号、出站写。  
- **详解**：见环境矩阵。  
- **追问**：stage 能共用 prod 向量库吗？  
- **常见错误**：同 prefix「省钱」。  
- **相关**：§5。

**Q6. `queued` 与 `running`？**  
- **短答**：排队等待 vs 持有租约执行。  
- **详解**：waiting_* 不占执行并发。  
- **追问**：如何测排队延迟？  
- **常见错误**：混用 status。  
- **相关**：§3.2。

**Q7. 为何要 `config_rev`？**  
- **短答**：绑定 Run 以便复现与回滚。  
- **详解**：热更新不默默改在途 Run。  
- **追问**：Flag 要快照吗？  
- **常见错误**：全局可变单例。  
- **相关**：§4.2。

**Q8. 密钥能否进 Git？**  
- **短答**：不能；配置与密钥分家。  
- **详解**：Secret Manager + 轮换。  
- **追问**：BYOK 如何隔离？  
- **常见错误**：`.env` 进镜像。  
- **相关**：§4.4。

**Q9. 什么是吵闹邻居？**  
- **短答**：单租户耗尽共享资源拖垮他人。  
- **详解**：分队列/配额/熔断。  
- **追问**：检测信号？  
- **常见错误**：只扩容不隔离。  
- **相关**：§7。

**Q10. 死信队列用途？**  
- **短答**：隔离毒丸并告警。  
- **详解**：避免无限重试占容量。  
- **追问**：哪些错误进 DLQ？  
- **常见错误**：DLQ 无人看。  
- **相关**：§3、§10。

### Intermediate

**Q11. 设计 Run 取消端到端路径。**  
- **短答**：写 cancel → Worker 安全点停 → 取消 Gateway → 释租约。  
- **详解**：工具按可取消性处理。  
- **追问**：支付类工具？  
- **常见错误**：只断 SSE。  
- **相关**：§3.3、§8.3。

**Q12. 如何防双 Worker 执行同一 Run？**  
- **短答**：租约 + fencing；副作用幂等键。  
- **详解**：租约过期可接管。  
- **追问**：时钟漂移？  
- **常见错误**：只靠可见性超时。  
- **相关**：§3.3。

**Q13. 租户配额最小集合？**  
- **短答**：并发/排队 Run、token、工具、步数墙钟、成本单位。  
- **详解**：超额策略可配。  
- **追问**：用户级如何叠？  
- **常见错误**：只有日调用次数。  
- **相关**：§6.2。

**Q14. 背压如何传到客户端？**  
- **短答**：配额/Gateway → 少发租约 → 入站 429/Retry-After。  
- **详解**：勿只堵内存。  
- **追问**：交互 vs 批处理？  
- **常见错误**：静默变慢。  
- **相关**：§3.5。

**Q15. 逻辑隔离如何强制 `tenant_id`？**  
- **短答**：鉴权注入；存储强制谓词；缓存命名空间。  
- **详解**：透传到 Tools/Gateway。  
- **追问**：防 ORM 漏过滤？  
- **常见错误**：信任业务自觉。  
- **相关**：§7。

**Q16. Feature Flag 与 ACL 边界？**  
- **短答**：Flag 管发布/降级；ACL 管能不能做。  
- **详解**：Flag 打开也不能越权。  
- **追问**：错误用法例？  
- **常见错误**：用 Flag 当授权。  
- **相关**：§4.3。

**Q17. 多队列优先级策略？**  
- **短答**：interactive/batch/replay + 档位权重。  
- **详解**：防饿死；有界 steal。  
- **追问**：公平性如何度量？  
- **常见错误**：单一 FIFO。  
- **相关**：§3.5。

**Q18. stage 影子流量注意？**  
- **短答**：无生产副作用；隔离写；采样；标记 shadow。  
- **详解**：工具录制或只读。  
- **追问**：如何防双计费？  
- **常见错误**：原样重放写操作。  
- **相关**：§5。

**Q19. checkpoint 太大怎么治？**  
- **短答**：大对象外置；存指针与摘要。  
- **详解**：恢复时再拉。  
- **追问**：与 Context 压缩关系？  
- **常见错误**：全 transcript 进行存储。  
- **相关**：§10。

**Q20. Loop Budget 与 Platform Quota？**  
- **短答**：前者单 Run；后者跨 Run；Gateway 护后端。  
- **详解**：三者缺一不可。  
- **追问**：冲突谁优先？  
- **常见错误**：只设模型 RPM。  
- **相关**：§6.3。

**Q21. Worker 如何安全取 BYOK？**  
- **短答**：按 secret_scope 短时取用；不落盘。  
- **详解**：记 key_id。  
- **追问**：轮换窗口？  
- **常见错误**：注入全部租户密钥。  
- **相关**：§4.4、§7。

**Q22. 为何依赖 `model_alias`？**  
- **短答**：路由/降级/换厂商不改业务。  
- **详解**：实际模型写入轨迹。  
- **追问**：别名如何灰度？  
- **常见错误**：硬编码具体模型 ID。  
- **相关**：§8。

**Q23. 计量为何要 outbox？**  
- **短答**：与状态变更同意图，防丢计费。  
- **详解**：异步投递；幂等去重。  
- **追问**：重复计量？  
- **常见错误**：日志当账本。  
- **相关**：§7.4。

**Q24. waiting_human 平台行为？**  
- **短答**：释放执行并发；保留状态；超时；通知。  
- **详解**：恢复时校验 ACL/配置。  
- **追问**：超时拒绝还是降级？  
- **常见错误**：线程阻塞占坑。  
- **相关**：§3.2。

**Q25. 配置校验应挡什么？**  
- **短答**：预算上限、非法别名、工具悬空、跨环境密钥。  
- **详解**：schema + 语义；CI 跑。  
- **追问**：谁负责？  
- **常见错误**：只靠人看 JSON。  
- **相关**：§4.5。

### Advanced

**Q26. 多区域 Active-Active 难点？**  
- **短答**：状态亲和、驻留、跨区取消、幂等。  
- **详解**：通常按租户/会话钉扎区域。  
- **追问**：故障转移防双执行？  
- **常见错误**：全局共享队列无分区。  
- **相关**：§7、§9。

**Q27. Gateway 依赖治理？**  
- **短答**：超时、舱壁、多后端、混沌、产品降级路径。  
- **详解**：网关故障时怎么办。  
- **追问**：是否允许应急直连？  
- **常见错误**：单点无超时。  
- **相关**：§8、§12。

**Q28. 租户级熔断状态机？**  
- **短答**：closed/open/half-open；信号=429/错误/重试风暴。  
- **详解**：与全局熔断分层。  
- **追问**：误熔断恢复？  
- **常见错误**：一刀切停全集群。  
- **相关**：§6.4、§7。

**Q29. 至少一次投递下副作用安全？**  
- **短答**：幂等键 + 副作用日志；条件写。  
- **详解**：平台约定标准 key。  
- **追问**：非幂等第三方？  
- **常见错误**：假设恰好一次。  
- **相关**：§3.4。

**Q30. State Store 分片键？**  
- **短答**：常用 `run_id`；租户列表用二级索引或按 tenant 分区。  
- **详解**：防热点大租户。  
- **追问**：「最近 Run」列表 QPS？  
- **常见错误**：全表扫租户。  
- **相关**：§9。

**Q31. SSE 与 Worker 扩展耦合？**  
- **短答**：事件经 fanout；Worker 写总线，边缘订阅。  
- **详解**：断线续传用事件序号。  
- **追问**：百万连接？  
- **常见错误**：Worker 直持 WS。  
- **相关**：§1.3、§2。

**Q32. 「配置即代码」的限度？**  
- **短答**：schema/策略可入仓；密钥与高频开关走快通道。  
- **详解**：分层发布。  
- **追问**：Prompt 是否 GitOps？  
- **常见错误**：一切热更新无审计。  
- **相关**：§4。

**Q33. 如何阻止绕过 Gateway？**  
- **短答**：出站策略、只注入 Gateway 凭证、依赖扫描。  
- **详解**：计量空洞即信号。  
- **追问**：sidecar 方案？  
- **常见错误**：只靠文档规范。  
- **相关**：§8、§10。

**Q34. 批处理百万 Run 调度？**  
- **短答**：独立低优先队列、配额借还、分段准入。  
- **详解**：保护 interactive SLO。  
- **追问**：如何估完成时间？  
- **常见错误**：与对话混池无权重。  
- **相关**：§3.5、§9。

**Q35. 在途 Run 变更工具 ACL？**  
- **短答**：在途冻结旧清单；新 Run 用新；紧急 revoke 可中断。  
- **详解**：收紧与放宽不同策略。  
- **追问**：如何审计？  
- **常见错误**：无版本瞬时全局替换。  
- **相关**：§4.2。

### Expert

**Q36. 租约 + fencing 如何防脑裂？**  
- **短答**：单调 fencing token；存储拒旧 token 写。  
- **详解**：旧 Worker 写失败。  
- **追问**：与 etcd 类比？  
- **常见错误**：只靠壁钟超时。  
- **相关**：§3.3。

**Q37. 跨租户容量控制系统？**  
- **短答**：测 tokens/并发 → 预测 → 准入 → 弹性 → 告警。  
- **详解**：以队列年龄与 429 为误差。  
- **追问**：扩缩振荡？  
- **常见错误**：周会拍脑袋扩容。  
- **相关**：§6、§9。

**Q38. 数据驻留如何约束拓扑？**  
- **短答**：租户钉扎区域；禁跨区 state/记忆；Gateway 选区内后端。  
- **详解**：备份与日志同约束。  
- **追问**：全球低延迟折中？  
- **常见错误**：日志汇单区无约束。  
- **相关**：§5、§7。

**Q39. 控制面与数据面故障隔离？**  
- **短答**：控制面只管元数据；执行/模型失败不拖垮查询创建只读路径。  
- **详解**：舱壁；配额宕机时的降级。  
- **追问**：创建是否可降级放行？  
- **常见错误**：单体混部所有职责。  
- **相关**：§2、Architecture。

**Q40. 如何 review Agent 云平台成熟度？**  
- **短答**：异步 Run、租约幂等、三重预算、Gateway、环境隔离、租户传播、取消、计量、死信。  
- **详解**：对照本文清单。  
- **追问**：企业客户不可缺哪三项？  
- **常见错误**：只看是否 K8s+GPU。  
- **相关**：全文。

### System Design

**Q41. 设计 SaaS Coding Agent 生产拓扑。**  
- **短答**：API+队列+Worker；Sandbox 池；Gateway；租户配额；事件 fanout。  
- **详解**：仓库凭证按 scope；交互队列优先。  
- **追问**：长 repo 索引放哪？  
- **常见错误**：API 进程内跑 docker 工具。  
- **相关**：§2、Sandbox、llm-inference。

**Q42. 设计带 HITL 退款的客服平台路径。**  
- **短答**：waiting_human；幂等支付工具；审计；超时。  
- **详解**：ACL 与配置冻结。  
- **追问**：防重复退款？  
- **常见错误**：模型直连支付。  
- **相关**：§3、§7、Tool Use。

**Q43. Demo 同步 → 多租户异步里程碑。**  
- **短答**：P0 外置状态→P1 队列租约→P2 Gateway 计量→P3 配额租户→P4 多环境与死信。  
- **详解**：每阶段可回滚。  
- **追问**：企业销售门禁是哪步？  
- **常见错误**：先上 Multi-Agent。  
- **相关**：§2.1、§9。

**Q44. 大促模型容量降级方案。**  
- **短答**：交互优先；batch 推迟；别名降级；砍非关键工具；入站公告。  
- **详解**：预置开关与演练。  
- **追问**：防小模型雪崩？  
- **常见错误**：无预案现场改配置。  
- **相关**：§6.4、§8。

**Q45. 多租户 Inference 计量与对账。**  
- **短答**：Gateway usage + outbox + 幂等账本；alias/concrete 双记。  
- **详解**：差异告警；重放补记。  
- **追问**：缓存命中 token 怎么计？  
- **常见错误**：只信厂商月报。  
- **相关**：§7.4、§8、llm-inference。

---

## 14. Cross References

| 方向 | 模块 |
|------|------|
| 上游 | [Agent Loop](../../core/agent-loop/agent-loop.md)、[Agent Architecture](../../core/agent-architecture/agent-architecture.md)、[Tool Use](../../core/tool-use/tool-use.md) |
| 模型平面 | [LLM Inference](../../runtime/llm-inference/README.md) |
| 执行隔离 | [Sandbox](../../runtime/sandbox/README.md) |
| 观测 | [Observability](../observability/README.md) |
| 安全深文 | [Security](../security/README.md) |
| 评测 / 模式 | [Evaluation](../evaluation/README.md)（不写方法）、[Design Patterns](../design-patterns/README.md) |
| 导航 | [learning-paths](../../docs/learning-paths.md)、[reading-paths](../../docs/reading-paths.md) |

---

## 15. References

- [Agent Loop](../../core/agent-loop/agent-loop.md)、[Agent Architecture](../../core/agent-architecture/agent-architecture.md)、[LLM Inference](../../runtime/llm-inference/README.md)
- [article-anthropic-building-effective-agents-2024](../../references/articles.md#article-anthropic-building-effective-agents-2024)
- [doc-openai-function-calling](../../references/docs.md#doc-openai-function-calling)、[doc-anthropic-how-tool-use-works](../../references/docs.md#doc-anthropic-how-tool-use-works)
- TODO: Verify — 云托管队列租约语义；厂商 Gateway 字段映射

---

## 16. Self-review / TODOs

- [x] 服务化、队列/异步 Run、配置密钥、多环境、限流配额、多租户、Gateway 衔接；45 题
- [x] 非目标成立：无评测方法论正文；无 Security 威胁模型正文
- [ ] TODO: Verify — SQS/PubSub/NATS 租约对照；Gateway 产品字段映射
