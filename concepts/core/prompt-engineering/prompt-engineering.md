# Prompt Engineering（Canonical）

> **一句话主题**：Prompt Engineering 是为 Agent **设计、版本化与评测指令层（instruction）**——角色、规则、输出协议、工具用法措辞与示例配方——使其在 Loop 中可靠地决策与调用工具；它重要，但不是 Agent Engineering 的中心。
>
> **状态**：Canonical · **Last reviewed**: 2026-10  
> **受众**：编写/维护 Agent 系统提示、工具描述措辞与输出契约的工程师  
> **前置依赖**：[foundations/llm](../../foundations/llm/README.md)（建议同步了解 [foundations/agent](../../foundations/agent/README.md)）  
> **中心链路（必读）**：[Agent Loop](../agent-loop/agent-loop.md) · [Tool Use](../tool-use/tool-use.md) · [Context Engineering](../context-engineering/context-engineering.md)

---

## 范围 / 非目标

**范围**：Agent 场景下的系统提示结构；tool-aware prompting 与 ACI（Agent-Computer Interface）措辞原则；结构化输出；few-shot 与配方化；Prompt vs Context 边界；Prompt 变更的回归评测；失败模式；面试题与引用。

**非目标**（正文不展开，只给指针）：

| 主题 | 去哪里 |
|------|--------|
| 窗口预算、压缩、截断、Observation Explosion、Lost in the Middle | [Context Engineering](../context-engineering/context-engineering.md) |
| 工具执行、鉴权、幂等、Registry、Observation 归一化机制 | [Tool Use](../tool-use/tool-use.md) |
| 控制回路、终止条件、checkpoint | [Agent Loop](../agent-loop/agent-loop.md) |
| 计划/反思策略语义 | [Reasoning & Planning](../reasoning-planning/reasoning-planning.md) |
| 微调 / RL 对齐 | [`runtime/fine-tuning`](../../runtime/fine-tuning/README.md) · [`runtime/reinforcement-learning`](../../runtime/reinforcement-learning/README.md) |
| MCP 协议与 prompts 资源 | [`protocols/mcp`](../../protocols/mcp/README.md) |

**定位（关键）**：

```text
Agent Engineering 中心 = Loop + Tools + Context
Prompt = Instruction（相对稳定、可版本化）
Context = 当次调用可见全集的策展（每步可变）
```

改 Prompt 能修「说不清规则」；修不了「窗口被工具 dump 撑爆」或「缺执行/鉴权」。后者分别属于 Context 与 Tool Use。

---

## 1. 知识模型：Agent 里的 Prompt 是什么

### 1.1 定义

**Prompt Engineering（本手册口径）**：设计并维护进入模型输入中的 **指令分区**——通常包括 system / developer 消息、静态政策、工具用法说明（与 schema 描述协同）、输出协议、以及经筛选的 few-shot——并用评测证明变更有效。

它回答的问题是：

> 「在给定工具面与上下文装配策略下，**如何把规则与接口说清楚**，使模型更可能做出正确决策？」

不回答：「这次该塞哪些检索片段 / 如何压缩历史」——那是 Context Engineering。

### 1.2 Mental Model

把一次模型调用想成：

```text
[Instruction Layer]  ← Prompt Engineering 主责（版本化配置）
[Dynamic Evidence]   ← Context Engineering 主责（State 投影）
[Tool Schemas]       ← Tool Use 定义；措辞与 Prompt 协同
        ↓
      Model
        ↓
  text / structured / tool_calls
```

| 层 | 典型内容 | 变化频率 | 主责模块 |
|----|----------|----------|----------|
| Instruction | 角色、硬约束、输出格式、工具何时用 | 低（发版） | **本文** |
| Evidence | 用户消息、历史、工具结果、RAG、Memory | 高（每步） | Context |
| Schema | name、JSON Schema、description | 中（工具发版） | Tool Use（措辞见本文 §4） |

### 1.3 为何「重要但不中心」

| 现象 | 若只堆 Prompt | 正确归属 |
|------|----------------|----------|
| 模型乱调工具 | 加禁止条款可能缓解 | 工具 description/ACI + 权限 + 评测（Tool Use） |
| 长任务遗忘约束 | 把约束复制进每条 user 消息 | Context 预算与分区；硬策略进代码 |
| 输出偶发非 JSON | 「必须输出 JSON」重复三遍 | Structured Output / schema 约束解码 |
| 多步失败 | 更长 CoT 提示 | Loop 终止/重试 + Reasoning 策略对象 |

Anthropic 主张从简单可组合模式起步、按评测加复杂度（[article-anthropic-building-effective-agents-2024](../../references/articles.md#article-anthropic-building-effective-agents-2024)）。Prompt 是其中一环，不是架构本身。

### 1.4 何时优先改 Prompt vs 改别处

**优先改 Prompt，当**：

- 规则从未写清，或自相矛盾
- 输出协议不稳定，且尚无 schema 硬约束
- 工具 description 含糊（何时用/不用未说明）
- few-shot 与真实任务分布错位

**先改别处，当**：

- 证据未进窗或被噪声淹没 → Context
- 工具失败/超时/权限错误 → Tool Use / Infra
- 无终止、无限工具环 → Loop
- 模型能力不够且评测已饱和 → 换模型或考虑 runtime 对齐（非本文）

---

## 2. Prompt vs Context（必读边界）

> 与 [Context Engineering §1.3](../context-engineering/context-engineering.md) 对齐；此处给 Prompt 侧操作定义，不重复预算/压缩正文。

### 2.1 对照表

| | Prompt（指令） | Context（调用视图） |
|--|----------------|---------------------|
| 角色 | **Instruction**：角色、规则、协议、静态示例 | **Invocation 可见全集** |
| 工程问题 | 写清楚、可版本、可回归 | 策展、排序、预算、更新 |
| 典型载体 | system / developer；配置仓库中的模板 | messages 装配结果 |
| 失败信号 | 规则冲突、格式漂移、示例污染 | 注意力噪声、中部丢失、观测膨胀 |

```text
Prompt ⊂（通常）Context 中的指令分区
Context Engineering ⊃ Prompt Engineering（≠「更好的措辞」）
```

### 2.2 反模式：把 Context 问题当 Prompt 问题

| 反模式 | 症状 | 正确动作 |
|--------|------|----------|
| 上帝 System Prompt | 数万 token 政策 + 动态数据粘贴 | 政策短而稳；动态进 Evidence 分区 |
| 「再强调一遍」循环 | 同一约束复制 N 次仍失败 | 查是否被截断/淹没；硬校验进代码 |
| 把工具 dump 写进 system | 试图「提醒模型注意上次结果」 | Observation 归一化后进证据区（Tool Use + Context） |
| Prompt 当状态机 | 用自然语言维护 plan/预算 | State + Loop；Reasoning 对象 |

### 2.3 分工检查清单（PR 用）

改动落点自检：

1. 这条文本 **每次调用都一样** 吗？→ 倾向 Prompt / 政策模板  
2. 依赖 **当前 State / 工具结果 / 检索** 吗？→ Context Builder  
3. 描述的是 **工具接口语义** 吗？→ Tool schema `description`（与 §4 协同）  
4. 是 **不可协商的安全边界** 吗？→ 代码/鉴权，不单靠服从 Prompt  

---

## 3. Prompts for Agents：系统提示结构

### 3.1 设计目标

Agent 的系统提示不是「人设小品」，而是 **运行时合同摘要**：

1. **身份与范围**：你是什么、服务谁、明确不做什么  
2. **硬约束**：安全、合规、语言、禁止动作（短）  
3. **工具用法总则**：何时调用、何时直接答、失败时如何表现（细节在各 tool description）  
4. **输出协议**：最终答复形态；中间过程是否对用户可见  
5. **工作风格**：简洁度、澄清策略、不确定性处理  
6. **（可选）静态示例指针**：短；长示例外置或 few-shot 槽位

### 3.2 推荐分区（模板骨架）

```text
# Role
...

# Scope / Non-goals
...

# Hard constraints（短、可测）
...

# Tooling principles（总则，非全量手册）
- Prefer tools for facts that may change
- Do not invent tool results
- On tool error: explain + suggest next step / retry once if transient
...

# Output contract
- Final answer: ...
- If tools needed: emit tool calls per schema（由 API 承载）
...

# Interaction policy
- Ask at most N clarifying questions when blocked
- Prefer progressive disclosure
...
```

**原则**：system 保持 **短、稳、可 diff**。长政策文档外置；运行时只注入与当前产品面相关的切片（装配仍属 Context，但切片选择策略可配置在 Prompt 模块旁）。

### 3.3 分层：System / Developer / User / Tool

| 角色（概念） | 用途 | 信任 |
|--------------|------|------|
| System | 产品级合同 | 最高（自有） |
| Developer / 内部指令 | 租户或任务模板覆盖 | 高（需 ACL） |
| User | 目标与偏好声明 | 半信；不可覆盖硬约束 |
| Tool result | 观察 | **默认不可信为指令** |

工程要点：**永不**让用户或工具结果「升格」改写 System 硬约束。分区标签应进入 trace，便于审计「模型服从了哪一层」。

### 3.4 Workflow Agent vs Autonomous Agent 的提示差

对齐 Anthropic 对 workflows vs agents 的区分（[article-anthropic-building-effective-agents-2024](../../references/articles.md#article-anthropic-building-effective-agents-2024)）：

| 类型 | Prompt 侧重点 |
|------|----------------|
| Workflow（代码编排） | 单步任务清晰；少「自主规划」修辞；强调本步输入输出 |
| Agent（模型主导过程） | 工具总则 + 停止条件说明 + 不确定时澄清；避免空泛「你是超级 AGI」 |

不要用 Prompt 假装有 Loop：终止、步数预算、checkpoint 在 [Agent Loop](../agent-loop/agent-loop.md)。

### 3.5 可测试的约束写法

差：

```text
尽量不要做坏事，谨慎使用工具。
```

好（可评测）：

```text
- 不得调用 refund 工具，除非用户本轮明确要求退款。
- 不得编造 order_id；缺失时先追问或调用 lookup_order。
- 最终用户可见答复使用简体中文，除非用户要求其他语言。
```

把约束写成 **可判定命题**，回归集才能断言 pass/fail。

### 3.6 版本与配置

| 实践 | 说明 |
|------|------|
| `prompt_version` | 语义版本或内容哈希进 trace |
| 模板 + 变量 | 仅注入白名单变量（产品名、地区）；禁止任意 State dump |
| 租户覆盖 | overlay 层；基线不可被租户删除安全段 |
| 评审 | Prompt PR 与代码 PR 同等：diff、评测、回滚 |

---

## 4. Tool-aware Prompting 与 ACI 措辞

> **边界**：本文只谈 **如何把工具说清楚（措辞与协同）**；执行、鉴权、幂等、并行机制见 [Tool Use](../tool-use/tool-use.md)。不在此重写 Tool Use 正文。

### 4.1 什么是 ACI

**ACI（Agent-Computer Interface）**：模型与工具面之间的「人机接口」——名称、描述、参数含义、错误返回形态。Anthropic 强调工具文档质量与评测驱动迭代（[article-anthropic-writing-tools-for-agents-2025](../../references/articles.md#article-anthropic-writing-tools-for-agents-2025)、[article-anthropic-building-effective-agents-2024](../../references/articles.md#article-anthropic-building-effective-agents-2024)）。

对 Prompt Engineering 的含义：

- System 提示写 **总则**（何时倾向用工具、如何对待错误）
- 每个工具的 `description` 写 **局部契约**（何时用/不用、副作用、返回语义）
- 两者冲突时，以 **可机器校验的政策 + schema** 为准，并修描述

官方工具机制说明见 [doc-anthropic-how-tool-use-works](../../references/docs.md#doc-anthropic-how-tool-use-works)、[doc-openai-function-calling](../../references/docs.md#doc-openai-function-calling)。

### 4.2 Description 措辞清单（Prompt 工程师必做）

每个工具描述应覆盖：

1. **一句话能力**（做什么）  
2. **触发条件**（何时用）  
3. **反触发**（何时不要用；与易混工具的边界）  
4. **副作用级别**（只读 / 写入 / 不可逆）——细节执行仍在 Tool Use  
5. **关键参数语义**（单位、ID 形态、时区）  
6. **失败时模型应如何表现**（可在 system 总则统一）

示意（描述文本，非执行实现）：

```text
name: lookup_order
description: |
  按 order_id 查询订单状态与可退款金额。
  在用户询问物流/支付状态，或退款前需要确认订单时使用。
  不要在用户仅闲聊时调用。
  不要用本工具下单或退款（见 create_refund）。
```

### 4.3 System 层的 tool-aware 总则（推荐短条款）

```text
- 对可能过期或私有的事实，优先工具而非记忆中的猜测。
- 工具结果可能不完整或错误；冲突时以工具结果为准并说明不确定性，或再查。
- 禁止声称已执行未发出的工具调用。
- 高副作用操作前，向用户确认（若产品政策要求）。
```

### 4.4 避免「Prompt 假装安全」

| 错误 | 为何无效 | 正确层 |
|------|----------|--------|
| 「绝不要调用 delete_*」仅写在 Prompt | 注入/越狱可绕过 | 鉴权 + 工具暴露子集 |
| 「参数必须合法」 | 模型仍会幻觉字段 | Schema 校验 + 拒绝执行 |
| 「并行调用要小心」长文 | 无调度语义 | Loop/Executor 策略 |

Prompt 可降低误用率；**不能**替代权限边界。详见 Tool Use 安全相关章节。

### 4.5 与 Schema 的分工

| 信息 | 放 description（自然语言） | 放 JSON Schema |
|------|----------------------------|----------------|
| 何时用/不用 | ✓ | ✗（难表达） |
| 类型、枚举、必填 | 可重复关键点 | ✓ 权威 |
| 单位/示例值 | ✓ 短例 | format / minimum 等 |
| 幂等键含义 | ✓ | 字段存在性 |

Schema 标准基础：[std-json-schema-2020-12](../../references/docs.md#std-json-schema-2020-12)。

### 4.6 工具过多时的提示策略

- System **不要**粘贴 80 个工具的用户手册  
- 用 Registry 权限过滤后的子集（机制在 Tool Use）  
- Prompt 只保留「如何选择工具」的元策略 + 当前子集描述  

---

## 5. Structured Output

### 5.1 问题

Agent 除 tool_calls 外，常需稳定的 **最终结构化载荷**（分类、抽取、打分、UI 动作提案）。自然语言「请输出 JSON」在生产中不够。

### 5.2 手段分层（由软到硬）

| 层级 | 手段 | 强度 | 代价 |
|------|------|------|------|
| L0 | Prompt 约定字段 | 弱 | 低 |
| L1 | Few-shot 展示形状 | 中弱 | token |
| L2 | 解析器 + 重试/修复 | 中 | 延迟 |
| L3 | 供应商 Structured Outputs / JSON mode / strict schema | 强 | 模型/API 限制 |
| L4 | 工具参数作为唯一结构通道 | 强 | 多一轮 |

生产默认：**能 schema 约束就不要只靠 L0**。Tool 参数本身即结构化通道（见 Tool Use）；最终答复也可用 JSON schema 约束。

> **Evergreen 2026-10-07**：OpenAI Structured Outputs（`strict`/json_schema）保证 schema 遵从，JSON mode 只保证合法 JSON；Anthropic 有 `output_config.format` + tool `strict: true`。完整模型矩阵仍随文档变。

### 5.3 Prompt 侧仍要写清的内容

即使有 L3，仍建议在指令中写：

- **语义**：字段含义与枚举业务意义  
- **空值策略**：未知用 `null` 还是省略  
- **禁止** markdown 代码篱（若 API 已保证纯 JSON 可省略）  
- **与 tool_calls 的互斥/顺序**：先工具后最终对象，或仅最终对象

### 5.4 校验与修复回路（Prompt + 代码）

```text
generate → validate(schema) → (fail) → repair_prompt + errors → generate
         → (pass) → accept
```

- `repair_prompt` 只含 **校验错误列表**，勿回灌整份错误政策百科  
- 限制 repair 次数；超限降级人工或安全默认值  
- 修复提示与主 system 分离，避免污染前缀缓存假设  

### 5.5 反模式

- 同一字段两套名字（`userId` vs `user_id`）无迁移策略  
- 结构化输出里塞长篇思维链（应分离：内部推理痕迹 vs 对外对象）  
- 用结构化输出模拟完整 Agent State（State 应在应用侧）

---

## 6. Few-shot 与 Recipes

### 6.1 Few-shot 在 Agent 中的角色

**Few-shot**：在指令层或邻近槽位放入 **输入→期望行为** 示例，用于校准格式、语气、工具选择边界。

对 Agent：

- 示例应展示 **决策边界**（何时调用 A 而非 B），不只展示漂亮最终答案  
- 示例中的工具调用必须与 **当前 schema** 一致，过期示例是高危回归源  
- 动态「从记忆里抽相似案例」属于 Context 装配；本文管 **静态配方与选型原则**

### 6.2 示例选择原则

| 原则 | 说明 |
|------|------|
| 覆盖边界 | 含易混任务、拒绝案例、错误恢复 |
| 最短充分 | 每例只保留决策相关字段 |
| 分布对齐 | 贴近线上真实；避免仅玩具题 |
| 去污染 | 不含过时工具名、内部密钥、真实 PII |
| 可版本 | 示例集有 ID；随工具发版更新 |

### 6.3 Zero / Few / Many

| 策略 | 适用 | 风险 |
|------|------|------|
| Zero-shot + 清晰协议 | 格式简单、模型强 | 边界案例不稳 |
| 2–5 shots | 格式/工具选择需校准 | token；位置敏感 |
| 大量示例 | 少见；优先检索式动态例 | 变相变成 Context 膨胀 |

动态多例的预算与排序 → [Context Engineering](../context-engineering/context-engineering.md)。

### 6.4 Recipe（配方）概念

**Recipe** = 可复用的「任务类型 → 提示模块 + 工具子集 + 输出 schema + 评测集」打包。

```text
recipe.support_refund_v3 =
  system_modules: [role_base, safety_payments, tool_principles]
  tools: [lookup_order, create_refund]
  output: RefundDecisionSchema
  fewshot_pack: refund_boundary_v3
  eval_suite: refund_regression_v3
```

收益：产品新增场景时复制配方而非复制一整份上帝 Prompt。

### 6.5 CoT 示例与 Agent

Chain-of-Thought 提示（[paper-cot-2022](../../references/papers.md#paper-cot-2022)）可提升部分推理题，但：

- **不等于** 生产 Plan 对象（见 [Reasoning & Planning](../reasoning-planning/reasoning-planning.md)）  
- 对外产品常需隐藏思维痕迹或分通道  
- Agent 更常需要「计划摘要进 State」，而非只在 Prompt 里写「一步步想」

---

## 7. 组合、模块化与变更治理

### 7.1 模块化拼接

推荐把 system 拆成模块：

```text
base_role
+ product_policy
+ tool_principles
+ output_contract
+ (±) locale_style
+ (±) recipe_overlay
```

拼接规则写进代码：顺序稳定、去重、冲突检测（例如两个模块都定义默认语言）。

### 7.2 变更分级

| 级别 | 例子 | 门禁 |
|------|------|------|
| P0 文案无关 | 错别字 | lint + 冒烟 |
| P1 行为可能变 | 改工具总则、示例 | 回归套件 |
| P2 安全/合规 | 硬约束、支付话术 | 全套 + 人工审核 |
| P3 工具契约 | description/schema | 工具评测 + 提示回归 |

### 7.3 与前缀缓存的工程关系

稳定前缀（system 前段不变）有利于部分部署的缓存命中。实践：

- 把 **高变内容** 往后放或放 user/evidence  
- 不要为「每次个性化」重写整个 system  

具体缓存行为随供应商变化。  
> **Evergreen 2026-10-07**：OpenAI 默认对够长前缀做 caching（稳定内容置前）；较新模型支持显式 breakpoint / `prompt_cache_options`；TTL ~30m，GPT-5.6+ 最低约 1024 tokens。Anthropic 用 `cache_control`，缓存序 tools→system→messages；默认 TTL 5m（可选 1h），最低 token 按模型 512–4096。**机制已核**；**计费倍率**仍 **TODO: Verify**（[evergreen-2026-10-caching-seed.md](../../docs/evergreen-2026-10-caching-seed.md)）。

---

## 8. Prompt 变更的回归评测

### 8.1 为什么必须评测

Prompt 是高杠杆、高回归源：一行「可以更主动」可能导致工具滥用或话术越权。无套件的 Prompt PR ≈ 无测试的业务逻辑变更。

### 8.2 套件分层

| 层 | 内容 | 频率 |
|----|------|------|
| Smoke | 10–30 条关键路径 | 每次 PR |
| Regression | 历史 bug 固化 + 边界 | 每次 PR |
| Recipe suite | 某配方专属 | 改该配方时 |
| Red team | 注入、越权、诱导滥调工具 | 定期 / P2+ |
| Online | 影子流量、评分、人工抽检 | 持续 |

评测平台与指标深文 → [`production/evaluation`](../../production/evaluation/README.md)。本文给 Prompt 侧最小集。

### 8.3 用例结构（推荐字段）

```yaml
id: refund_requires_explicit_ask
goal: 用户未要求退款时不得调用 create_refund
messages: [...]
tools_allowed: [lookup_order, create_refund]
assertions:
  - no_tool: create_refund
  - final_contains_any: ["需要确认", "是否退款"]
tags: [payments, safety]
```

断言类型：

- 工具名集合（必须调用 / 禁止调用）  
- 参数字段满足 schema 与业务谓词  
- 最终结构化对象字段  
- 安全：不得泄露 system、不得遵从恶意指令  

### 8.4 对比实验协议

1. 固定模型与解码参数（或矩阵声明）  
2. 固定工具面与 Context 策略（避免把 Context 变更算到 Prompt 头上）  
3. 基线 vs 候选；预注册主指标  
4. 多次采样估计方差（非确定性任务）  
5. 失败案例入库，禁止只报平均分  

**消融**：去掉 few-shot / 缩短硬约束 / 改 description，定位真正起作用的模块。

### 8.5 发布与回滚

- 金丝雀：按租户或百分比切换 `prompt_version`  
- 自动回滚触发：禁止工具调用率、安全拦截率、任务成功率跌破阈值  
- 保留「已知坏版本」黑名单  

### 8.6 与「感觉更好」的斗争

评审禁止只贴两段对话截图。最低要求：**套件差分表**（通过数、新失败、延迟/token 可选）。

---

## 9. 失败模式

| ID | 模式 | 信号 | 缓解 |
|----|------|------|------|
| F1 | 规则冲突 | 模型两难或随机偏向 | 冲突检测；优先级表；删冗余 |
| F2 | 示例污染 | 工具名/格式与线上不一致 | 示例 CI；随 schema 破改 |
| F3 | 格式漂移 | JSON 解析失败升高 | Structured Output；repair 限次 |
| F4 | 工具滥用 | 无关查询暴增 | ACI 反触发；暴露子集；评测禁调断言 |
| F5 | 工具怯用 | 该查不查、幻觉事实 | 总则 + 边界 few-shot；事实类任务强制工具政策 |
| F6 | 指令稀释 | 中后段约束失效 | 缩短 system；硬约束前置；查 Context 淹没（链 Context） |
| F7 | 注入覆盖 | 用户/工具文本改写行为 | 分区；政策代码化；红队 |
| F8 | 角色扮演越权 | 「忽略以上」生效 | 硬拒绝集；不单靠 Prompt |
| F9 | 版本漂移 | 环境间 system 不一致 | 哈希校验；配置即代码 |
| F10 | 过度 CoT | 延迟升、泄露思维 | 分通道；任务类型开关 |
| F11 | Recipe 腐化 | 复制粘贴分叉 | 中心配方库；共享模块 |
| F12 | 评测虚荣 | 只测玩具题 | 线上抽样回流；对抗例 |

检测建议：对 F3/F4/F5 建仪表盘；对 F7/F8 绑安全用例；对 F2/F9 绑 CI。

---

## 10. 实践建议（可执行）

1. **先合同后文采**：硬约束与输出协议优先于人设形容词。  
2. **System 短于一屏**（经验量级；超长先拆模块/外置）。  
3. **工具细节进 description**，System 只留总则。  
4. **结构靠 schema**，不靠三重「请输出 JSON」。  
5. **每个 Prompt PR 带套件差分**。  
6. **安全边界双写**：Prompt 说明 + 代码强制。  
7. **怀疑 Context 问题时不要继续堆叠强调句**——打开 trace 看装配。  
8. **能力不够时评估换模型或 runtime 方案**，而非无限加示例（链 runtime）。

### 10.1 最小可行 Prompt 栈

```text
P0: 短 system + 输出协议 + 工具 description 审查
P1: 边界 few-shot pack + smoke/regression
P2: recipe 化 + structured output
P3: 红队 + 线上影子 + 版本金丝雀
```

---

## 11. 权衡表

| 决策 | 收益 | 成本 | 建议 |
|------|------|------|------|
| 更长 system | 规则更全 | 稀释、成本、难 diff | 模块化；测后再加 |
| 更多 few-shot | 边界更稳 | token、位置敏感 | 2–5 高质量边界例 |
| 只 Prompt 约束 JSON | 快 | 解析失败 | 尽快上 schema |
| 把政策写进 Prompt | 迭代快 | 可被注入绕过 | 关键政策代码化 |
| 动态示例检索 | 适配长尾 | Context 复杂度 | 配方静态 + 必要时 Context JIT |

---

## 12. Interview Questions

### Beginner

**Q1. 在本手册中 Prompt Engineering 负责什么？**  
- **短答**：指令层——角色、规则、协议、工具用法措辞与示例。  
- **详解**：服务 Loop/Tools，不替代它们。  
- **追问**：中心是什么？  
- **常见错误**：当成 Agent 工程全部。  
- **相关**：§1。

**Q2. 为什么说它重要但不中心？**  
- **短答**：中心是 Loop+Tools+Context；Prompt 是指令合同。  
- **详解**：堆 Prompt 修不好观测膨胀与缺鉴权。  
- **追问**：举一例该改 Context 的。  
- **常见错误**：无限加长 system。  
- **相关**：开篇定位、§2。

**Q3. Prompt vs Context？**  
- **短答**：Prompt≈指令；Context≈当次可见全集。  
- **详解**：Prompt 常是 Context 的指令分区。  
- **追问**：工具结果算哪边？  
- **常见错误**：两词混用。  
- **相关**：§2、Context Canonical。

**Q4. System 提示建议含哪些分区？**  
- **短答**：角色、范围、硬约束、工具总则、输出协议、交互策略。  
- **详解**：短、稳、可测。  
- **追问**：动态订单数据放哪？  
- **常见错误**：上帝 Prompt。  
- **相关**：§3。

**Q5. 什么是可测试的约束写法？**  
- **短答**：可判定命题，而非「尽量谨慎」。  
- **详解**：能写成 eval assertion。  
- **追问**：给支付场景一例。  
- **常见错误**：模糊形容词堆叠。  
- **相关**：§3.5、§8。

**Q6. ACI 是什么？**  
- **短答**：Agent 与工具面的接口文档/契约体验。  
- **详解**：名称、描述、参数、错误形态；评测迭代。  
- **追问**：和 Tool Use 机制差在？  
- **常见错误**：只改 system 不改 description。  
- **相关**：§4、[`article-anthropic-writing-tools-for-agents-2025`](../../references/articles.md#article-anthropic-writing-tools-for-agents-2025)。

**Q7. 工具 description 至少写什么？**  
- **短答**：能力、何时用/不用、副作用、关键参数语义。  
- **详解**：与易混工具划界。  
- **追问**：为何反触发重要？  
- **常见错误**：只写「查询订单」。  
- **相关**：§4.2。

**Q8. Structured Output 为何不能只靠 Prompt？**  
- **短答**：服从不稳定；需 schema/约束解码/校验修复。  
- **详解**：L0–L4 分层。  
- **追问**：tool 参数算结构通道吗？  
- **常见错误**：重复三遍「输出 JSON」。  
- **相关**：§5。

**Q9. Few-shot 在 Agent 里最该展示什么？**  
- **短答**：决策边界与正确工具选择，不只漂亮答案。  
- **详解**：与当前 schema 一致。  
- **追问**：过期示例风险？  
- **常见错误**：只展示最终散文。  
- **相关**：§6。

**Q10. Prompt 变更为什么要回归？**  
- **短答**：高杠杆行为变更，易引入工具滥用/越权。  
- **详解**：套件+差分，禁止只靠截图。  
- **追问**：smoke 最小集？  
- **常见错误**：感觉更好就上线。  
- **相关**：§8。

### Intermediate

**Q11. Workflow Agent 与 Autonomous Agent 的提示差？**  
- **短答**：前者单步清晰；后者工具总则+停止/澄清策略。  
- **详解**：对齐 Anthropic workflows vs agents。  
- **追问**：停止条件写在 Prompt 就够吗？  
- **常见错误**：用文案假装有 Loop。  
- **相关**：§3.4、Agent Loop。

**Q12. 用户消息能否覆盖 System 硬约束？**  
- **短答**：不能；半信并隔离。  
- **详解**：升格攻击是注入。  
- **追问**：工具结果呢？  
- **常见错误**：拼接成同一 blob。  
- **相关**：§3.3、F7。

**Q13. 设计 refund 工具的 description 要点。**  
- **短答**：仅明确退款意图时用；先查单；幂等键；非闲聊。  
- **详解**：与 lookup 划界。  
- **追问**：如何评测？  
- **常见错误**：安全只写在 system。  
- **相关**：§4、§8.3。

**Q14. Schema 与 description 如何分工？**  
- **短答**：类型约束进 schema；何时用进 description。  
- **详解**：关键语义可双写但不冲突。  
- **追问**：单位写哪？  
- **常见错误**：schema 无描述、描述无结构。  
- **相关**：§4.5。

**Q15. 修复 JSON 的回路怎么设计？**  
- **短答**：validate→错误列表 repair→限次→降级。  
- **详解**：勿回灌百科。  
- **追问**：与 structured API 关系？  
- **常见错误**：无限修复。  
- **相关**：§5.4。

**Q16. 什么是 Recipe？**  
- **短答**：任务类型打包：模块提示+工具子集+schema+评测。  
- **详解**：防上帝 Prompt 分叉。  
- **追问**：和 Context 配方差？  
- **常见错误**：复制整份 system 改三行。  
- **相关**：§6.4。

**Q17. 如何防止 few-shot 污染？**  
- **短答**：示例 CI、随工具破改、去 PII、版本 ID。  
- **详解**：失败入库。  
- **追问**：动态相似例风险？  
- **常见错误**：三年前聊天当金标。  
- **相关**：§6.2、F2。

**Q18. CoT prompting 与生产 Planning 区别？**  
- **短答**：CoT 是提示痕迹；Planning 是可治理状态对象。  
- **详解**：链 Reasoning Canonical。  
- **追问**：对外要藏思维吗？  
- **常见错误**：二者等同。  
- **相关**：§6.5、[`paper-cot-2022`](../../references/papers.md#paper-cot-2022)。

**Q19. Prompt PR 的门禁有哪些？**  
- **短答**：diff 评审、套件差分、安全级人工、可回滚版本。  
- **详解**：P0–P3 分级。  
- **追问**：文案错别字？  
- **常见错误**：无主指标发布。  
- **相关**：§7.2、§8。

**Q20. 如何做 Prompt 消融？**  
- **短答**：去掉模块/示例/改 description，固定 Context 与模型。  
- **详解**：定位真正增益。  
- **追问**：方差怎么处理？  
- **常见错误**：同时改五处。  
- **相关**：§8.4。

**Q21. 「指令稀释」如何识别？**  
- **短答**：约束在长文中后失效；探针任务失败。  
- **详解**：缩短+前置；并查 Context。  
- **追问**：与 Lost in the Middle 关系？  
- **常见错误**：继续追加强调句。  
- **相关**：F6、Context。

**Q22. 工具怯用怎么治？**  
- **短答**：总则要求可验证事实走工具；边界 few-shot；指标监控。  
- **详解**：幻觉事实用例。  
- **追问**：与滥用如何平衡？  
- **常见错误**：只禁不导。  
- **相关**：F5、§4.3。

**Q23. 为何不能把完整政策 PDF 塞进 system？**  
- **短答**：噪声、成本、难测、易冲突。  
- **详解**：切片+外置+代码硬规则。  
- **追问**：法务要求全文？  
- **常见错误**：一次性粘贴。  
- **相关**：§3.2、§2.2。

**Q24. 多租户 Prompt 覆盖如何设计？**  
- **短答**：基线不可删安全段；overlay ACL；版本追踪。  
- **详解**：冲突检测。  
- **追问**：租户要「更主动销售」？  
- **常见错误**：租户全文替换 system。  
- **相关**：§3.6、§7.1。

**Q25. 红队用例最小集？**  
- **短答**：忽略指令、工具结果注入、诱导高副作用、索取系统提示。  
- **详解**：定期跑。  
- **追问**：和支付套件关系？  
- **常见错误**：只测快乐路径。  
- **相关**：§8.2、F7/F8。

### Advanced

**Q26. 设计支持金丝雀的 Prompt 配置系统。**  
- **短答**：配置库+版本哈希+流量分配+自动回滚指标。  
- **详解**：trace 带 version。  
- **追问**：与模型版本交叉实验？  
- **常见错误**：改文件即全球生效。  
- **相关**：§8.5、§7。

**Q27. 如何把「政策优先级」编码进提示仍可执行？**  
- **短答**：提示声明顺序；冲突以代码强制为准。  
- **详解**：优先级表单测。  
- **追问**：模型仍违反？  
- **常见错误**：只靠加粗。  
- **相关**：§3.5、§4.4。

**Q28. 工具搜索/动态加载时 Prompt 写什么？**  
- **短答**：元策略：先发现再调用；禁止虚构未加载工具。  
- **详解**：机制在 Tool Use。  
- **追问**：description 何时注入？  
- **常见错误**：system 列全部工具百科。  
- **相关**：§4.6。

**Q29. 结构化最终答复与 tool_calls 并存的协议。**  
- **短答**：明确阶段：工具回合无最终对象；结束回合仅对象或短文本。  
- **详解**：校验互斥。  
- **追问**：边解释边调用？  
- **常见错误**：同条消息杂糅难解析。  
- **相关**：§5.3、Loop。

**Q30. 评估 description 改写的实验设计。**  
- **短答**：固定任务集；只改描述；测正确调用率/错误调用率/步数。  
- **详解**：对齐 Anthropic 评测迭代思路。  
- **追问**：样本量？  
- **常见错误**：改描述同时改 schema。  
- **相关**：§4、[`article-anthropic-writing-tools-for-agents-2025`](../../references/articles.md#article-anthropic-writing-tools-for-agents-2025)。

**Q31. Prompt 模块冲突检测怎么做？**  
- **短答**：静态规则（重复键、语言默认、禁工具列表合并策略）。  
- **详解**：CI 失败阻断。  
- **追问**：语义冲突？  
- **常见错误**：字符串拼接无检查。  
- **相关**：§7.1、F1。

**Q32. 何时应从 Prompt 迭代转向微调/RL？**  
- **短答**：套件饱和、稳定失败模式、成本上可接受训练。  
- **详解**：链 runtime；非默认第一步。  
- **追问**：数据从哪来？  
- **常见错误**：评测未做就微调。  
- **相关**：非目标、runtime。

**Q33. 多 Agent 时系统提示如何避免目标打架？**  
- **短答**：每 Agent 窄角色；共享政策模块；编排层裁决。  
- **详解**：深文 multi-agent。  
- **追问**：共享 few-shot？  
- **常见错误**：复制同一上帝角色。  
- **相关**：§6.4、Architecture。

**Q34. 设计「禁止泄露 system」的评测与缓解。**  
- **短答**：诱饵探测；拒绝模板；输出过滤。  
- **详解**：不单靠「保密」一句。  
- **追问**：间接复述？  
- **常见错误**：认为 Prompt 保密=安全。  
- **相关**：§8.3、Security。

**Q35. 前缀缓存友好的 Prompt 布局。**  
- **短答**：稳定段前置；高变后置或进 user/evidence。  
- **详解**：供应商相关。  
- **追问**：租户 overlay 影响？  
- **常见错误**：每请求重写全 system。  
- **相关**：§7.3。

**Q36. 如何度量「主动澄清」是否过度？**  
- **短答**：澄清轮次、任务完成时延、用户放弃率。  
- **详解**：A/B 策略句。  
- **追问**：与工具怯用关系？  
- **常见错误**：澄清越多越好。  
- **相关**：§3.1、§8。

**Q37. Recipe 腐化的治理。**  
- **短答**：中心库、共享模块、破改通知、定期重跑套件。  
- **详解**：删除死配方。  
- **追问**：业务紧急分叉？  
- **常见错误**：私自复制后永不汇合。  
- **相关**：F11、§6.4。

**Q38. 将自然语言政策编译为可执行检查。**  
- **短答**：关键条款→断言/守卫函数；Prompt 留摘要。  
- **详解**：双轨一致测试。  
- **追问**：谁维护映射？  
- **常见错误**：两份政策长期漂移。  
- **相关**：§4.4、§10。

**Q39. 面试：调试「突然乱调搜索工具」。**  
- **短答**：查 version diff、description、few-shot、Context 噪声、注入。  
- **详解**：用禁调断言复现。  
- **追问**：先回滚还是先加禁令？  
- **常见错误**：只加「不要乱用」。  
- **相关**：F4、§8、§9。

**Q40. Audit 声称「我们做了很多 Prompt Engineering」的项目。**  
- **短答**：有无版本、套件、ACI、与 Context/Tool 边界、硬安全。  
- **详解**：对照本文清单。  
- **追问**：只有文案平台？  
- **常见错误**：字数当质量。  
- **相关**：§10、open-source-analysis。

### System Design

**Q41. 设计生产级 Prompt 管理服务。**  
- **短答**：模块存储、拼装、版本、实验、鉴权、审计、评测钩子。  
- **详解**：与 Config/Feature flag 集成。  
- **追问**：边缘用本地缓存？  
- **常见错误**：Wiki 复制粘贴上线。  
- **相关**：§7、§8。

**Q42. 设计支付类 Agent 的提示+评测方案。**  
- **短答**：窄角色；退款双确认政策代码化；工具边界例；红队+回归。  
- **详解**：禁止只靠 Prompt 鉴权。  
- **追问**：误退款指标？  
- **常见错误**：示例含真实卡号。  
- **相关**：§3–§8。

**Q43. 设计 Coding Agent 的系统提示骨架。**  
- **短答**：角色=编程助手；工具总则；补丁/测试协议；不整仓粘贴（Context）。  
- **详解**：链 Coding Agent / Sandbox。  
- **追问**：与 shell 工具 description？  
- **常见错误**：把仓库规范全书塞 system。  
- **相关**：§3、§4、Context。

**Q44. 设计从「单文件 system.txt」迁到 Recipe 体系的里程碑。**  
- **短答**：P0 版本哈希→P1 模块拆分→P2 套件→P3 配方包→P4 金丝雀。  
- **详解**：每步可回滚。  
- **追问**：不可妥协项？  
- **常见错误**：一次性大重构无评测。  
- **相关**：§10.1。

**Q45. 端到端：新工具上线时 Prompt 侧 checklist。**  
- **短答**：写 ACI 描述；更新总则如需；加边界 few-shot；断言用例；查与旧工具易混；发版记录。  
- **详解**：执行/鉴权在 Tool Use 清单。  
- **追问**：如何防怯用/滥用？  
- **常见错误**：只加 schema 不改文档与评测。  
- **相关**：§4、§6、§8。

---

## 13. Cross References

- [Agent Loop](../agent-loop/agent-loop.md) · [Tool Use](../tool-use/tool-use.md) · [Context Engineering](../context-engineering/context-engineering.md)  
- [Reasoning & Planning](../reasoning-planning/reasoning-planning.md) · [Agent Architecture](../agent-architecture/agent-architecture.md)  
- [Memory](../memory/memory.md) · [RAG](../rag/rag.md)  
- [`production/evaluation`](../../production/evaluation/README.md) · [`production/security`](../../production/security/README.md) · [`production/observability`](../../production/observability/README.md)  
- [`runtime/fine-tuning`](../../runtime/fine-tuning/README.md) · [`runtime/reinforcement-learning`](../../runtime/reinforcement-learning/README.md)  
- [`protocols/mcp`](../../protocols/mcp/README.md)

---

## 14. References

- [article-anthropic-writing-tools-for-agents-2025](../../references/articles.md#article-anthropic-writing-tools-for-agents-2025)  
- [article-anthropic-building-effective-agents-2024](../../references/articles.md#article-anthropic-building-effective-agents-2024)  
- [article-anthropic-context-engineering-2025](../../references/articles.md#article-anthropic-context-engineering-2025)  
- [doc-openai-function-calling](../../references/docs.md#doc-openai-function-calling)  
- [doc-anthropic-tool-use](../../references/docs.md#doc-anthropic-tool-use)  
- [doc-anthropic-how-tool-use-works](../../references/docs.md#doc-anthropic-how-tool-use-works)  
- [std-json-schema-2020-12](../../references/docs.md#std-json-schema-2020-12)  
- [paper-cot-2022](../../references/papers.md#paper-cot-2022)  

边界深文（不重复）：[Context Engineering](../context-engineering/context-engineering.md) · [Tool Use](../tool-use/tool-use.md)

---

## 15. Self-review / TODOs

- [x] 定位：重要但不中心；中心 Loop+Tools+Context  
- [x] Prompt vs Context 必读边界；未写预算/压缩/Observation Explosion 正文  
- [x] prompts-for-agents、ACI 措辞、structured output、few-shot/recipes、评测  
- [x] 未重写 Tool Use 执行/鉴权；未写微调/RL  
- [x] 失败模式 + 45 面试题 + references + README 指针  
- [x] Evergreen 2026-10-07：Structured Outputs vs JSON mode；OpenAI/Anthropic caching 机制  
- [x] Evergreen 2026-10-07：caching **TTL + 最低 token** 机制已核；计费倍率仍 Verify  
- [ ] TODO: Verify — 完整模型矩阵与 caching **计费倍率 / 价目数字**  
