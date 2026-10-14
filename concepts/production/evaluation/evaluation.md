# Agent Evaluation（Canonical）

> **一句话主题**：Agent Evaluation 回答 **如何用任务结果、轨迹过程、成本/延迟与安全信号，证明系统变好或变坏**——它是生产闭环的度量与门禁，不是刷预训练榜，也不是 RL 训练本身。
>
> **状态**：Canonical · **Last reviewed**: 2026-10  
> **受众**：负责 Agent 发布质量、回归门禁与实验设计的工程师 / Tech Lead  
> **硬前置**：[Agent Loop](../../core/agent-loop/agent-loop.md)、[Tool Use](../../core/tool-use/tool-use.md)  
> **建议前置**：[Agent Architecture](../../core/agent-architecture/agent-architecture.md)  
> **边界对照**：[Observability](../observability/README.md) · [`runtime/reinforcement-learning`](../../runtime/reinforcement-learning/README.md) · [Security](../security/README.md)

---

## 范围 / 非目标

**范围**：测什么（成功/质量/成本/延迟/安全）；Offline vs Online；数据集与评分标准（rubrics）；轨迹（trajectory）评估；工具使用评估；回归门禁与切片；反模式；与 RL / Observability 的边界；面试。

**非目标**：

| 主题 | 去哪里 |
|------|--------|
| Traces / spans / 回放实现、日志隐私深文 | [`observability`](../observability/README.md)（本文只消费信号） |
| RLHF / RLAIF / 轨迹级 RL / 奖励黑客训练细节 | [`runtime/reinforcement-learning`](../../runtime/reinforcement-learning/README.md) |
| 威胁模型、注入防护全文 | [`security`](../security/README.md) |
| MMLU / GSM8K / 通用预训练榜刷分 | **禁止作为本文主体** |
| 编码 Agent 专用基准清单深文 | [`coding-agent`](../../agent-types/coding-agent/README.md) |
| Prompt 措辞配方 | [`prompt-engineering`](../../core/prompt-engineering/README.md) |

**核心不等式**：

```text
Agent Eval ≠ Model Benchmark（预训练/通用能力榜）
Agent Eval ≠ Observability（测「好不好」≠ 怎么采/存/查 trace）
Agent Eval ≠ RL（度量与门禁 ≠ 用反馈更新权重/策略）
Demo 成功 ≠ 生产可发布
最终答案对 ≠ 轨迹可接受
```

---

## 1. Definitions

### 1.1 Agent Evaluation

**Agent Evaluation** = 在明确任务分布与约束下，对 Agent **run / 实验 / 版本** 的：

1. **结果**（是否完成用户目标）  
2. **过程**（轨迹是否合规、高效、可恢复）  
3. **资源**（成本、延迟、工具配额）  
4. **风险**（安全、合规、副作用）

进行 **可复现度量、对比、切片与门禁** 的工程实践。

### 1.2 与相邻概念

| 概念 | 主问题 | 本文关系 |
|------|--------|----------|
| **Model Benchmark** | 模型通用能力如何 | 可作粗筛；**不替代**任务评测 |
| **Observability** | 如何看见与诊断一次 run | 提供轨迹原料；指标语义在 Eval |
| **Experimentation** | 如何因果归因变更 | Online Eval 的子集（A/B、影子） |
| **RL / Preference Learning** | 如何用信号改进策略 | Eval **产出**可作奖励/偏好数据；训练在 RL |
| **QA / CI** | 变更是否破坏已知行为 | Regression Gates 是 Eval 落地形态 |

### 1.3 评测单元

| 单元 | 含义 | 典型用途 |
|------|------|----------|
| **Case** | 单条任务样本（输入+约束+期望） | 离线集原子 |
| **Run** | 一次完整 Agent 执行（含多步） | 轨迹与资源统计 |
| **Slice** | 按属性分组的子集 | 发现局部崩坏 |
| **Version** | Prompt/工具/模型/策略组合 | 对比与门禁 |
| **Experiment** | 受控流量上的版本对比 | Online |

---

## 2. What to Measure（必须五维）

生产 Agent 至少同时报告五维；缺一维会系统性误导发布决策。

### 2.1 Success（任务成功）

**定义**：在任务契约下，最终状态是否满足验收条件。

| 形态 | 例子 | 判分方式 |
|------|------|----------|
| **可验证成功** | 测试通过、字段写入正确、订单状态 | 程序断言 / 环境 oracle |
| **结构化成功** | JSON schema、必填字段、引用存在 | 校验器 |
| **开放成功** | 报告是否回答问题、是否满足政策 | Rubric + 人工/LLM-judge |
| **部分成功** | 多子目标完成比例 | 分项权重 |

工程要点：

- 把「用户满意」拆成 **可观测验收**（acceptance criteria），避免只留 vibe。  
- 区分 **任务失败** vs **系统失败**（超时、工具 5xx、预算耗尽）——后者进可靠性切片，勿与能力混报。  
- 多步任务优先定义 **终态断言**，再叠加过程约束。

### 2.2 Quality（质量，超出二元成功）

成功是地板；质量描述「好到什么程度」。

| 维度 | 含义 | 信号 |
|------|------|------|
| **正确性深度** | 关键事实、引用、计算 | 事实核对、引用校验 |
| **完整性** | 覆盖用户约束与边界条件 | Rubric 分项 |
| **忠实性 / Grounding** | 输出可追溯到证据或工具结果 | Citation / 证据对齐 |
| **风格与格式** | 产品要求的语气、结构 | 规则 + 抽样人工 |
| **澄清与拒答质量** | 该问则问、该拒则拒 | 专门 case 集 |
| **用户可感知价值** | 是否省时、是否可执行 | 在线反馈（弱标签） |

**禁止**：用单一「有用性」分数掩盖失败模式；质量分数必须可分解到 rubric 维度。

### 2.3 Cost（成本）

| 计量 | 说明 |
|------|------|
| **$/run、$/成功** | 成功分母防「多跑到对」 |
| Token（in/out/cache） | 分模型、分步骤 |
| 工具调用次数 / 外部 API $ | 常主导真实账单 |
| 人审成本（若含 HITL） | 别漏 |

报告习惯：同时给 **均值、p95、按任务切片**；单独报「平均成本下降」而无成功约束 → 反模式。

### 2.4 Latency（延迟）

| 计量 | 说明 |
|------|------|
| **TTFT / 首包** | 交互体验 |
| **端到端 wall time** | 完成任务 |
| **每步模型延迟、工具延迟** | 归因（细节采集链 Obs） |
| **步数 / 轮次** | 与延迟强相关的过程代理指标 |

SLO 思路：按产品路径定义 p95 上限；探索型 Agent 可放宽端到端，但必须有 **预算硬顶**（步数/时间/钱）。

### 2.5 Safety（安全与合规）

| 类别 | 测什么 |
|------|--------|
| **策略违规** | 越权工具、禁用动作、泄漏敏感字段 |
| **注入与越狱** | 用户/检索/工具结果诱导 |
| **副作用失控** | 误删、误发、重复扣款 |
| **拒答正确性** | 该拒却执行 / 该做却拒 |
| **审计完备** | 高危动作是否留下可追责记录（信号来自 Obs，门禁在 Eval） |

安全评测必须有 **红队集 + 生产影子高危采样**；不可用「总成功率高」替代。

### 2.6 五维联合读法

```text
只看 Success ↑     → 可能成本爆炸或更危险
只看 Cost ↓        → 可能更常拒答或偷工
只看 Latency ↓     → 可能跳步、少工具、质量崩
只看 Safety 严     → 可能可用性崩（过度拒答）
只看 Quality 主观  → 可能不可复现、不可门禁
```

发布决策用 **约束优化** 表述：在 Safety 与 Cost/Latency 硬约束下最大化 Success/Quality。

---

## 3. Offline vs Online

### 3.1 Offline Evaluation

**定义**：在冻结的任务集、环境或模拟器上，对固定版本跑批评测。

| 优点 | 局限 |
|------|------|
| 可复现、可 CI、便宜迭代 | 分布易过时 |
| 可穷举危险边界 | 缺真实用户意图漂移 |
| 适合回归门禁 | 模拟器/桩不等于生产工具 |

**适用**：PR 门禁、工具 schema 变更、提示词大改、模型切换预筛。

### 3.2 Online Evaluation

**定义**：在真实（或影子）流量上收集结果与过程信号，做版本对比。

| 模式 | 含义 | 风险 |
|------|------|------|
| **Shadow / Dark launch** | 新版本并行跑，不对外生效 | 双倍成本；副作用工具需隔离 |
| **A/B / 多臂** | 流量分流比较 | 需样本量与护栏 |
| **Canary** | 小流量渐进 | 需自动回滚指标 |
| **Human-in-the-loop 采样** | 对线审核 | 延迟与成本 |

**适用**：确认 Offline 增益在真实分布成立；发现长尾失败。

### 3.3 对照表

| | Offline | Online |
|--|---------|--------|
| 数据 | 策展集 | 生产分布 |
| 标签 | 金标 / rubric / oracle | 隐式反馈 + 抽样金标 |
| 因果 | 版本可控 | 需实验设计 |
| 安全 | 可含红队 | 必须护栏与熔断 |
| 角色 | **门禁主场** | **确认与发现主场** |

### 3.4 推荐闭环

```text
变更 → Offline 回归门禁 → Shadow（无副作用或沙箱）→ Canary → 全量
                ↑                              │
                └──── 失败 case 回流数据集 ←─────┘
```

Online 发现的失败必须 **晋升** 为 Offline case（带切片标签），否则门禁会永久漏检。

### 3.5 标签与反馈强度

| 信号 | 强度 | 用途 |
|------|------|------|
| 环境 oracle / 测试 | 强 | 成功主指标 |
| 人工 rubric | 强–中 | 质量校准、judge 校准 |
| LLM-as-judge | 中 | 规模化；需抽样人工锚定 |
| 显式点赞/投诉 | 中–弱 | Online 趋势 |
| 隐式（重试、编辑、弃用） | 弱 | 假设生成，勿单独门禁 |

---

## 4. Datasets & Rubrics

### 4.1 任务集设计原则

1. **代表生产意图**，不是代表论文榜。  
2. **分层**：smoke（快） / core（门禁） / long-tail / red-team / cost-stress。  
3. **每条 case 有元数据**：域、难度、工具需求、安全标签、是否含副作用。  
4. **版本化**：dataset@semver；变更写 changelog（增删改理由）。  
5. **防泄漏**：训练/调参用过的 case 不得混入盲测门禁集（见反模式）。

### 4.2 Case 最小字段

```text
id, version
input: user_message / state_seed / env_fixtures
constraints: policy, tools_allowed, budget
acceptance: assertions | structured_schema | rubric_id
labels: slice_tags[]
gold_trajectory?: optional process constraints
notes: known_ambiguities
```

开放任务可以没有唯一金答，但必须有 **rubric_id** 或 **oracle 程序**。

### 4.3 Rubrics（评分标准）

**Rubric** = 可分解、可教给人工与自动判分器的质量契约。

好 rubric：

- 分项独立、可观察（「有无引用 ID」优于「看起来专业」）  
- 权重显式；关键项可设 **一票否决**（安全、硬约束）  
- 附带正/反例边界  
- 定期与人工金标做 **校准报告**（agreement）

坏 rubric：单句「1–5 分有用性」；与产品政策冲突；随模型口味漂移无版本。

### 4.4 判分器选型

| 判分器 | 何时用 | 风险 |
|--------|--------|------|
| **程序断言 / 测试** | 可验证任务 | 覆盖面受限 |
| **规则 / schema** | 格式、必填、禁词 | 过脆或过松 |
| **人工** | 校准、争议、安全 | 贵、慢 |
| **LLM-judge** | 开放质量规模化 | 偏好偏见、可被套路；需金标锚定 |
| **交叉模型 judge** | 降同源偏袒 | 仍非真相 |

**校准协议（最低）**：每季度对核心切片抽 N 条，算人工 vs judge 一致性；一致性崩则冻结自动门禁权重。

### 4.5 环境与夹具

Agent 评测常依赖工具与世界状态：

- **Deterministic fixtures**：固定 API mock、固定文件树、固定时钟  
- **Simulators**：浏览器/终端/店铺模拟——保交互结构，承认逼真度缺口  
- **录制回放**：工具响应金样；注意协议变更使回放失效  

不可复现的「连真实第三方随便打」→ 只适合 Online / 手工探查，不进 CI 硬门禁。

---

## 5. Trajectory Evaluation（过程评估）

### 5.1 为什么不能只看最终答案

Agent 是多步决策系统。仅终态正确仍可能：

- 调用了越权工具后「碰巧」改对  
- 泄漏密钥后再完成任务  
- 空转 40 步烧预算后蒙对  
- 破坏性中间态已被执行（不可回滚）

故：**过程约束是一等公民**。

### 5.2 轨迹是什么

**Trajectory** = 一次 run 中按时间排列的决策与观察序列，至少含：

```text
(step, thought?/plan?, model_output, tool_calls[], observations[],
 state_delta?, budget_counters, termination_reason)
```

采集与存储形态见 Observability；本节定义 **如何评**。

### 5.3 过程指标族

| 族 | 例子 | 用途 |
|----|------|------|
| **效率** | 步数、冗余工具调用、重复搜索 | 成本/延迟根因 |
| **合规** | 是否违反工具白名单、是否跳过确认 | 安全 |
| **恢复** | 错误后是否修正、是否死循环 | 鲁棒 |
| **计划一致性** | 是否按声明计划执行、无谓偏离 | 推理解释（慎用自动） |
| **信息使用** | 是否忽略已返回的关键观察 | Context/工具设计问题 |
| **终止质量** | 过早 final / 该停不停 | Loop 预算与策略 |

### 5.4 过程约束的编码方式

1. **硬断言**：禁止工具集合、必须调用 `policy_check`、不得出现 PII 明文出站。  
2. **软评分**：冗余步惩罚、路径次优但可接受。  
3. **金轨迹对齐**：只在短、决定性任务上用；长任务用「关键里程碑」而非逐步对齐。  
4. **反事实检查**：若去掉某次危险调用，终态是否仍成功——成功则标记「幸运成功」。

### 5.5 轨迹评测伪代码

```text
function score_run(run, case, rubric):
  success = evaluate_acceptance(run.final_state, case.acceptance)
  safety  = check_hard_safety(run.trajectory, case.constraints)
  if safety.veto:
    return Fail(reason=safety), metrics...

  process = {
    steps: len(run.trajectory),
    illegal_tools: count_illegal(run),
    loops: detect_loops(run),
    recovery: score_recovery(run),
  }
  quality = score_rubric(run.output, rubric, judges...)
  cost, latency = summarize_resources(run)

  return Aggregate(success, quality, process, cost, latency, safety)
```

### 5.6 与 Reasoning 痕迹

显式 CoT / plan 文本 **不是** 可靠的过程金标：模型可写漂亮计划却做错工具。优先评 **可观察动作与状态变化**；语言痕迹仅作调试辅助（链 [Reasoning](../../core/reasoning-planning/reasoning-planning.md)）。

---

## 6. Tool-Use Evaluation

深文机制见 [Tool Use](../../core/tool-use/tool-use.md)；此处给 **度量契约**。

### 6.1 工具面必须单独切片

工具错误常被「最终自然语言圆回来」掩盖。至少报告：

| 指标 | 含义 |
|------|------|
| **Tool selection accuracy** | 该调谁 vs 实调谁（在有金标或规则时） |
| **Argument validity** | schema 校验通过率；语义正确率（需金参或 oracle） |
| **Execution success** | 工具返回 ok vs 应用/下游错误 |
| **Observation use** | 后续步骤是否依赖返回值（启发式：实体 ID 是否续用） |
| **Unauthorized attempts** | 越权调用尝试率（即使被运行时拦截） |
| **Retry / thrash** | 同工具同参重复、无新信息重试 |

### 6.2 评测设计要点

- **金工具序列**：适合短流程（「查单 → 退款」）；长探索任务改用约束 + 终态。  
- **参数容错**：语义等价（日期格式、ID 别名）要进比对器，否则假阴性。  
- **副作用工具**：Offline 必须 mock；评「是否发出正确意图」，执行层用沙箱（链 Sandbox）。  
- **工具描述变更**：用同一任务集做前后对比——Anthropic 强调用评测驱动工具文档迭代（[article-anthropic-writing-tools-for-agents-2025](../../references/articles.md#article-anthropic-writing-tools-for-agents-2025)）。

### 6.3 典型失败模式 → 指标映射

| 失败 | 看什么 |
|------|--------|
| 选错工具 | selection + 混淆矩阵 |
| 参数幻觉 | arg validity + 实体接地检查 |
| 忽略错误观察 | recovery / observation use |
| 平行乱调 | 并行冲突、重复副作用计数 |
| 该用工具却空想 | 「本可验证却未调用」率 |

### 6.4 最小工具评测集构成

1. Happy path（单工具 / 短链）  
2. 歧义选择（两工具易混）  
3. 强制澄清（缺槽位）  
4. 错误注入（工具 4xx/5xx/空结果）  
5. 权限拒绝  
6. 大规模观察（爆窗）后的再调用  
7. 禁调用红队  

---

## 7. Regression Gates（回归门禁）

### 7.1 门禁层级

| 层级 | 触发 | 内容 | 失败后果 |
|------|------|------|----------|
| **L0 Smoke** | 每次 PR | 极小集，分钟级 | 阻断合并 |
| **L1 Core** | PR / nightly | 核心切片 + 安全硬项 | 阻断发布候选 |
| **L2 Full** | nightly / 发版前 | 全量离线 + 成本报表 | 阻断发版 |
| **L3 Online guards** | canary | 实时成功率/安全/成本护栏 | 自动回滚 |

### 7.2 门槛怎么定

用 **相对基线 + 绝对地板**：

```text
pass iff
  success >= max(floor_success, baseline_success - ε_s)
  safety_veto_rate <= floor_safety
  p95_latency <= budget_latency
  cost_per_success <= baseline * (1 + ε_c)   # 或绝对上限
  critical_slices all green
```

禁止「总分微涨但安全切片崩」仍通过——**关键切片一票否决**。

### 7.3 切片分析（强制）

至少按下列切片拆开报，禁止只报全局均值：

- 任务类型 / 域  
- 是否需要工具、工具族  
- 难度 / 多跳  
- 新用户 vs 回流  
- 安全敏感 vs 普通  
- 模型路由（若多模型）  
- 语言 / 租户（若多租户）  

全局 90% 成功、支付切片 40% → **发布失败**。

### 7.4 变更归因清单

每次门禁报告绑定：

- 变更类型：模型 / prompt / 工具 schema / 检索 / 策略代码 / 预算  
- 预期影响切片  
- Offline diff（case 级翻车列表）  
- 是否更新了数据集（防「改集充好」）

### 7.5 与发布系统的接口

Eval 门禁输出应是机器可读：

```text
GateResult {
  version_id,
  dataset_id,
  metrics_by_slice,
  vetoes[],
  flaky_cases[],
  artifact_uris[]   // 轨迹包，存储细节属 Obs
}
```

CD 只消费 `vetoes` 与阈值；人看切片仪表盘。

---

## 8. Anti-Patterns（反模式）

| # | 反模式 | 为什么有害 | 纠正 |
|---|--------|------------|------|
| 1 | **只看 Demo / 少量金样** | 过拟合叙事 | 分层集 + 盲测 |
| 2 | **刷预训练榜代替 Agent 任务评测** | 测错对象 | 任务成功与轨迹 |
| 3 | **只评最终答案** | 掩盖危险过程 | 过程硬约束 |
| 4 | **指标漂移无版本** | 不可比 | metric@semver |
| 5 | **训练集泄漏进门禁集** | 虚假进步 | 隔离 + 哈希审计 |
| 6 | **调参时反复看盲测** | 盲测变训练集 | 真正 holdout |
| 7 | **LLM-judge 无人工锚定** | 自我偏好闭环 | 校准协议 |
| 8 | **全局均值掩盖切片崩坏** | 错误放行 | 强制切片否决 |
| 9 | **成本优化无成功约束** | 学会拒答/偷懒 | $/成功 + 成功率地板 |
| 10 | **Online 无护栏** | 事故 | canary 熔断 |
| 11 | **用用户点赞当唯一门禁** | 噪声大、滞后 | 弱信号辅助 |
| 12 | **环境不可复现却进 CI** | 碎绿 | fixtures/mock |
| 13 | **把 Obs 仪表盘当 Eval 结论** | 看见≠证明变好 | 显式对比实验 |
| 14 | **为过门禁删难例** | 自欺 | changelog 审计 |
| 15 | **安全只靠「模型更对齐」叙事** | 不可证 | 红队集 + 拦截率 |
| 16 | **一步到位 RL，跳过 Eval 基建** | 优化未知目标 | 先度量后学习 |

---

## 9. 与 RL 的边界

详见 [`runtime/reinforcement-learning`](../../runtime/reinforcement-learning/README.md)。此处钉死分工：

| | Evaluation | Reinforcement Learning |
|--|------------|------------------------|
| 目标 | **度量、对比、门禁、归因** | **用反馈改进策略/模型** |
| 输出 | 指标、切片报告、失败 case | 更新后的政策/权重/偏好模型 |
| 成功标准 | 可复现证明版本差异 | 期望回报提升且泛化 |
| 风险焦点 | 测错、泄漏、虚假门禁 | 奖励黑客、不安全探索 |
| 数据流向 | 生产/离线 → 指标 | Eval 信号/偏好 → 训练 |

```text
Eval 回答：这个版本能上线吗？比上一版好在哪？
RL  回答：如何把反馈变成更好的策略？
```

**可衔接但不混淆**：

- Offline 可验证奖励（测试通过、格式正确）可作 RL 的 verifiable reward——**奖励定义仍先在 Eval 站得住**。  
- 轨迹偏好对（chosen/rejected）来自 rubric/人工——**标注规范属 Eval；优化算法属 RL**。  
- Online 探针可采集探索数据——**生产护栏与门禁仍是 Eval/发布职责**。

Reflexion 等「语言强化、不改权重」属 Loop/Memory 运行时技巧，**不是**本模块的 RL；也勿当成已完成评测基建。

---

## 10. 与 Observability 的边界

| | Evaluation | Observability |
|--|------------|---------------|
| 主问题 | 好不好、能否发 | 看见什么、如何诊断 |
| 典型产物 | 数据集、rubric、门禁、实验结论 | trace、日志、回放 UI |
| 消费关系 | **消费**轨迹与资源事件算指标 | **生产**这些事件 |

本文不展开 span 模型、采样率、PII 红线、调试 playbook——一律链 [Observability](../observability/README.md)。约定：没有稳定 trace schema，Eval 的轨迹指标会碎；但有了 dashboards ≠ 已做评测。

---

## 11. 生产评测架构（机制级）

```text
                    ┌──────────────┐
  Cases / Rubrics ─►│ Eval Runner  │◄── Version (model, prompt, tools, policy)
                    └──────┬───────┘
                           │ runs
                           ▼
                    ┌──────────────┐
                    │  Scorers     │  assertions / rubric / judges / safety
                    └──────┬───────┘
                           │
              ┌────────────┼────────────┐
              ▼            ▼            ▼
         Gate API    Slice Report   Case Triage
              │            │            │
              ▼            ▼            ▼
           CD/Release   Dashboard    Dataset回流
```

组件职责：

1. **Eval Runner**：对 case 启 Agent Loop，注入 fixtures，收束预算。  
2. **Scorers**：纯函数优先；LLM-judge 异步可重跑。  
3. **Gate API**：给 CI/CD 布尔+否决码。  
4. **Triage**：翻车 case 打标签、负责人、是否晋升核心集。  
5. **实验服务**（Online）：分流、护栏、样本量估算——可与投放平台集成。

---

## 12. 实践建议（可执行）

1. **第一周**：10–30 条核心可验证 case + 5 条红队；L0 进 PR。  
2. **第一个月**：rubric 分项、工具切片、成本/延迟报表；失败回流流程。  
3. **上线前**：Shadow 或 Canary；安全否决自动回滚。  
4. **常态**：月度泄漏审计、judge 校准、删改集走评审。  
5. **指标语言**：对内统一「Success / Quality / Cost / Latency / Safety」五维看板，禁止私有黑话总分。  
6. **变更默认假设**：任何工具描述或路由变更都可能毁选择准确率——用工具评测集挡。  
7. **写作验收**：每个产品能力文档附「如何测」。

---

## 13. Trade-offs

| 张力 | 偏向 A | 偏向 B |
|------|--------|--------|
| 覆盖 vs 速度 | 全量稳 | Smoke 快 |
| 自动 vs 人工 | 规模 | 校准真相 |
| 逼真环境 vs 可复现 | Online/模拟高保真 | CI fixtures |
| 过程严苛 vs 探索自由 | 合规域 | 研究/创意域 |
| 门禁严 vs 迭代速 | 高风险 | 早期产品 |

选择应写进团队「评测政策」一页纸，并随风险升级收紧。

---

## 14. Interview Questions

### Beginner

**Q1. Agent Evaluation 是什么？**  
- **短答**：在任务分布上度量结果、过程、成本、延迟与安全，并用于对比与门禁。  
- **详解**：生产闭环，不是刷通用榜。  
- **追问**：与 model benchmark 区别？  
- **常见错误**：把 MMLU 当发布依据。  
- **相关**：§1。

**Q2. 必须测哪五维？**  
- **短答**：Success、Quality、Cost、Latency、Safety。  
- **详解**：缺一维会误导。  
- **追问**：为何 $/run 不够？  
- **常见错误**：只报准确率。  
- **相关**：§2。

**Q3. Offline vs Online？**  
- **短答**：冻结集可复现 vs 真实流量确认。  
- **详解**：门禁主场在 Offline；online 发现长尾。  
- **追问**：只有 online 行吗？  
- **常见错误**：用线上感觉代替回归。  
- **相关**：§3。

**Q4. 什么是 rubric？**  
- **短答**：可分解的质量评分契约。  
- **详解**：分项+否决项+例子。  
- **追问**：和单一 Likert 分差别？  
- **常见错误**：不可观察的「专业度」。  
- **相关**：§4.3。

**Q5. 为何要轨迹评估？**  
- **短答**：终态对仍可能过程危险或极贵。  
- **详解**：硬约束+效率指标。  
- **追问**：举例幸运成功。  
- **常见错误**：只 diff 最终字符串。  
- **相关**：§5。

**Q6. Tool-use eval 测什么？**  
- **短答**：选工具、参数、执行、观察利用、越权尝试。  
- **详解**：防自然语言掩盖。  
- **追问**：无金参数怎么办？  
- **常见错误**：只看任务成功率。  
- **相关**：§6。

**Q7. Regression gate 是什么？**  
- **短答**：版本变更必须通过的自动门槛。  
- **详解**：相对基线+绝对地板+切片否决。  
- **追问**：L0/L1 差在哪？  
- **常见错误**：只拦编译错误。  
- **相关**：§7。

**Q8. Eval 与 Observability？**  
- **短答**：Obs 看见；Eval 证明好坏与能否发。  
- **详解**：Eval 消费 trace。  
- **追问**：有仪表盘算评测吗？  
- **常见错误**：等同两者。  
- **相关**：§10。

**Q9. Eval 与 RL？**  
- **短答**：度量门禁 vs 用反馈改进策略。  
- **详解**：信号可衔接，职责分离。  
- **追问**：可验证奖励两边怎么分？  
- **常见错误**：上 RL 替代评测基建。  
- **相关**：§9。

**Q10. 何谓可验证成功？**  
- **短答**：程序/环境可判定的验收。  
- **详解**：测试、字段、状态机。  
- **追问**：开放写作任务呢？  
- **常见错误**：凡事靠 LLM-judge。  
- **相关**：§2.1。

### Intermediate

**Q11. 设计 30 条起步任务集。**  
- **短答**：核心业务 15 + 工具链 8 + 红队 5 + smoke 2。  
- **详解**：元数据与验收齐全。  
- **追问**：如何避免全是 happy path？  
- **常见错误**：全抄公开榜。  
- **相关**：§4、§12。

**Q12. 如何校准 LLM-judge？**  
- **短答**：抽样人工金标、算一致性、设失效阈值。  
- **详解**：分维度校准。  
- **追问**：一致性掉了怎么办？  
- **常见错误**：上线后永不校准。  
- **相关**：§4.4。

**Q13. Shadow 流量注意什么？**  
- **短答**：副作用隔离、双倍成本、对比指标对齐。  
- **详解**：写操作不得打真。  
- **追问**：只读工具？  
- **常见错误**：影子版真实下单。  
- **相关**：§3.2。

**Q14. $/成功 为何优于平均成本？**  
- **短答**：防止靠少做事降成本。  
- **详解**：与成功率联合。  
- **追问**：拒答算成功吗？  
- **常见错误**：优化平均 token。  
- **相关**：§2.3、§8。

**Q15. 切片一票否决举例。**  
- **短答**：支付/PII/越权切片失败则全局再高也否决。  
- **详解**：写进 gate 政策。  
- **追问**：如何选关键切片？  
- **常见错误**：只看 macro average。  
- **相关**：§7.2–7.3。

**Q16. 工具描述改了如何回归？**  
- **短答**：跑工具选择/参数集 + 核心任务集。  
- **详解**：看混淆矩阵。  
- **追问**：和模型升级同 PR？  
- **常见错误**：只靠手工试两个问题。  
- **相关**：§6、[`article-anthropic-writing-tools-for-agents-2025`](../../references/articles.md#article-anthropic-writing-tools-for-agents-2025)。

**Q17. 开放域质量怎么门禁？**  
- **短答**：rubric 分项 + 锚定 judge + 人工抽检配额。  
- **详解**：安全仍用硬规则。  
- **追问**：分数波动大？  
- **常见错误**：单一自动分卡死。  
- **相关**：§4。

**Q18. 数据泄漏如何发现？**  
- **短答**：case 哈希、调参日志、训练关联审计。  
- **详解**：holdout 物理隔离。  
- **追问**：无意复制近义例？  
- **常见错误**：同一条又练又测。  
- **相关**：§8。

**Q19. 过程「死循环」怎么量化？**  
- **短答**：重复状态/重复工具调用指纹超阈值。  
- **详解**：进过程指标与否决。  
- **追问**：合法重试呢？  
- **常见错误**：只靠最大步数。  
- **相关**：§5.3。

**Q20. Canary 护栏设哪些？**  
- **短答**：成功率地板、安全事件率、p95 延迟、成本倍率。  
- **详解**：自动回滚。  
- **追问**：样本量不够？  
- **常见错误**：只盯均值无安全。  
- **相关**：§3、§7。

### Advanced

**Q21. 幸运成功检测方法。**  
- **短答**：检查非法/危险中间动作；反事实去掉后是否仍成功。  
- **详解**：记为 process fail。  
- **追问**：如何自动化？  
- **常见错误**：终态绿就过。  
- **相关**：§5.4。

**Q22. 多模型路由下的评测。**  
- **短答**：按路由切片；门禁含错误路由率与贵模型滥用率。  
- **详解**：成本质量联合。  
- **追问**：级联回退？  
- **常见错误**：只评最终混合总分。  
- **相关**：§2、§7.3。

**Q23. 模拟器逼真度不足时怎么办？**  
- **短答**：Offline 保契约；关键路径 Online/影子；缺口写进已知限制。  
- **详解**：勿假装高保真。  
- **追问**：何时值得重做模拟器？  
- **常见错误**：CI 打真实付费 API。  
- **相关**：§4.5。

**Q24. 设计工具参数语义等价器。**  
- **短答**：规范化（时间、ID、金额、枚举别名）再比。  
- **详解**：单测覆盖等价类。  
- **追问**：自由文本参数？  
- **常见错误**：字符串全等。  
- **相关**：§6.2。

**Q25. 指标版本化方案。**  
- **短答**：metric_id@semver；改定义发新版本，旧报告不可默比。  
- **详解**：兼容说明。  
- **追问**：权重调整算 major？  
- **常见错误**：同名指标偷换公式。  
- **相关**：§8。

**Q26. 红队集维护策略。**  
- **短答**：按威胁建模分类；事故驱动新增；定期退役失效项。  
- **详解**：与 Security 联动。  
- **追问**：被模型「背答案」？  
- **常见错误**：红队十年不变。  
- **相关**：§2.5、§8。

**Q27. 轨迹金标过拟合风险。**  
- **短答**：长任务改里程碑/约束；允许多合法路径。  
- **详解**：避免强迫单一工具序。  
- **追问**：合规必须固定序？  
- **常见错误**：逐步强制对齐探索任务。  
- **相关**：§5.4。

**Q28. Online 隐式反馈如何用？**  
- **短答**：生成假设与切片预警；不单独做硬门禁。  
- **详解**：抽样升级为金标。  
- **追问**：重试率上升？  
- **常见错误**：点赞率当 KPI 唯一。  
- **相关**：§3.5。

**Q29. 评测失败回流流程。**  
- **短答**：triage→标签→复现夹具→晋升数据集→负责模块。  
- **详解**：关闭需门禁验证。  
- **追问**：偶发 flaky？  
- **常见错误**：Slack 一说就过。  
- **相关**：§3.4、§11。

**Q30. 成本门禁与产品免费额度冲突。**  
- **短答**：分层预算；免费档低成本模型+更严步数；评测分档。  
- **详解**：防跨档泄漏贵路由。  
- **追问**：如何测？  
- **常见错误**：一套预算打天下。  
- **相关**：§2.3、§13。

### Expert

**Q31. 形式化 Gate 为约束满足问题。**  
- **短答**：硬约束（安全、合规切片）+ 软目标（成功/质量）+ 资源上限。  
- **详解**：否决码枚举。  
- **追问**：多目标帕累托？  
- **常见错误**：加权总分一把糊。  
- **相关**：§2.6、§7。

**Q32. Judge 被套路（adversarial output）怎么办？**  
- **短答**：多样 judge、规则预滤、人工抽检、隐藏评分细则。  
- **详解**：定期对抗集。  
- **追问**：与奖励黑客关系？  
- **常见错误**：公开完整判分提示。  
- **相关**：§4.4、§9。

**Q33. 跨租户政策不同如何组织数据集？**  
- **短答**：共享核心 + 租户政策覆盖层；切片含 policy_id。  
- **详解**：门禁按租户矩阵。  
- **追问**：配置即代码？  
- **常见错误**：单一全球集。  
- **相关**：§4、§7.3。

**Q34. 将 Eval 信号导出给 RL 的契约。**  
- **短答**：稳定 reward schema、provenance、安全过滤、训练/门禁数据隔离。  
- **详解**：链 RL 模块。  
- **追问**：在线学习护栏？  
- **常见错误**：直接拿可刷的 judge 分当奖励。  
- **相关**：§9。

**Q35. 证明「变好」的最小实验设计。**  
- **短答**：预注册指标、功效/样本量、护栏、固定窗口、切片计划。  
- **详解**：防 peeking。  
- **追问**：中途改指标？  
- **常见错误**：看盘到显著再定义成功。  
- **相关**：§3。

**Q36. 多 Agent 系统如何评？**  
- **短答**：系统级验收 + 角色级契约（消息schema、越权）+ 成本汇总。  
- **详解**：禁只评子 Agent 单测代替端到端。  
- **追问**：责任归因？  
- **常见错误**：忽略通信膨胀成本。  
- **相关**：§5、Multi-Agent README。

**Q37. 评测基础设施的 SLO。**  
- **短答**：L0 时长、flaky 率、报告新鲜度、数据集可用性。  
- **详解**：评测挂了要当事故。  
- **追问**：谁 oncall？  
- **常见错误**：评测坏了改 skip。  
- **相关**：§7、§11。

**Q38. 长程任务（小时级）评测策略。**  
- **短答**：检查点断言、里程碑 rubric、资源积分；全长跑夜间。  
- **详解**：短代理指标辅助。  
- **追问**：如何做 PR 级？  
- **常见错误**：每次 PR 跑满小时。  
- **相关**：§5、§7.1。

**Q39. 安全「过度拒答」与「漏拒」双指标。**  
- **短答**：敏感集召回（漏拒）+ 良性集误拒率。  
- **详解**：联合门禁。  
- **追问**：如何造良性易混集？  
- **常见错误**：只优化拒答率。  
- **相关**：§2.5。

**Q40. Audit「我们有 Eval」的项目。**  
- **短答**：查五维、数据集版本、门禁、切片、回流、与 Obs/RL 边界。  
- **详解**：对照反模式表。  
- **追问**：只有 notebook 算吗？  
- **常见错误**：有一次榜分数就宣称。  
- **相关**：§8、§12。

### System Design

**Q41. 设计公司级 Agent Eval 平台。**  
- **短答**：Case 仓库、Runner、Scorer 插件、Gate API、实验、Triage、权限。  
- **详解**：多团队共享核心安全集。  
- **追问**：如何多区域数据驻留？  
- **常见错误**：每团队自造互不可比指标。  
- **相关**：§11。

**Q42. 设计 Coding Agent 发布门禁。**  
- **短答**：可验证测例 + 仓库夹具 + 越权命令红队 + 成本/时长；长基准夜间。  
- **详解**：链 coding-agent；禁只报 SWE 公开分。  
- **追问**：网络隔离？  
- **常见错误**：连真仓库乱写当 CI。  
- **相关**：§6、§7。

**Q43. 设计客服 Agent 的 Online+Offline 闭环。**  
- **短答**：离线剧本集；线上影子意图分类；投诉晋升；rubric 含政策。  
- **详解**：HITL 抽检配额。  
- **追问**：多语言？  
- **常见错误**：只盯 CSAT。  
- **相关**：§3、§4。

**Q44. 设计工具生态（第三方 MCP 工具）准入评测。**  
- **短答**：schema 质量、描述混淆、副作用声明、沙箱行为、红队、成本封顶。  
- **详解**：准入门禁≠信任模型自觉。  
- **追问**：工具更新强制复测？  
- **常见错误**：市场上下载即用。  
- **相关**：§6、MCP README。

**Q45. 从 0 到生产的 Eval 里程碑。**  
- **短答**：P0 可验证核心+安全硬门 → P1 五维报表+切片 → P2 工具/轨迹 → P3 Online 护栏+回流 → P4 校准与实验平台。  
- **详解**：RL 放在度量稳定后。  
- **追问**：不可妥协项？  
- **常见错误**：P0 直接上复杂偏好模型训练。  
- **相关**：§12、§9。

---

## 15. Cross References

- [Agent Loop](../../core/agent-loop/agent-loop.md) · [Tool Use](../../core/tool-use/tool-use.md) · [Agent Architecture](../../core/agent-architecture/agent-architecture.md)  
- [Reasoning & Planning](../../core/reasoning-planning/reasoning-planning.md) · [Context Engineering](../../core/context-engineering/context-engineering.md)  
- [Observability](../observability/README.md) · [Security](../security/README.md) · [Infrastructure](../infrastructure/README.md)  
- [`runtime/reinforcement-learning`](../../runtime/reinforcement-learning/README.md) · [`runtime/sandbox`](../../runtime/sandbox/README.md)  
- [`coding-agent`](../../agent-types/coding-agent/README.md) · [`protocols/mcp`](../../protocols/mcp/README.md)

---

## 16. References

- [article-anthropic-writing-tools-for-agents-2025](../../references/articles.md#article-anthropic-writing-tools-for-agents-2025) — 评测驱动工具描述迭代  
- [article-anthropic-building-effective-agents-2024](../../references/articles.md#article-anthropic-building-effective-agents-2024) — 可观测与按需复杂度；评测为生产前提  
- [paper-react-2022](../../references/papers.md#paper-react-2022) — 推理-行动交错轨迹的思想源头（评的是交互过程，非榜）  
- 边界模块：[Observability](../observability/README.md) · [RL](../../runtime/reinforcement-learning/README.md) · [Security](../security/README.md)

---

## 17. Self-review / TODOs

- [x] 五维度量 + Offline/Online + datasets/rubrics  
- [x] Trajectory / Tool-use / Regression gates / Anti-patterns  
- [x] 与 RL、Observability 边界明确；未刷预训练榜；未写 Obs 深文  
- [x] 45 面试题 + references + 交叉链接  
- [ ] TODO: Verify — 具体厂商实验平台/自动评测产品 API（有意不绑死）  
- [ ] TODO: Verify — 团队 LLM-judge 一致性基线需用自有金标标定  
