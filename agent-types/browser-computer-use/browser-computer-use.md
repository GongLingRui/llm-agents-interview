# Browser / Computer Use（Canonical）

> **一句话主题**：Browser / Computer Use Agent 是在 **GUI / 浏览器 / 桌面环境** 中，通过感知（DOM / a11y / 截图）选择动作、执行副作用并靠反馈闭环完成任务的垂直形态——核心难点是 **定位脆弱、状态异步、安全面极大**，而非「再加几个点击工具」。
>
> **状态**：Canonical · **Last reviewed**: 2026-10  
> **受众**：设计或加固 GUI / 浏览器操控 Agent 的工程师  
> **硬前置**：[Agent Loop](../../core/agent-loop/agent-loop.md)、[Tool Use](../../core/tool-use/tool-use.md)  
> **强烈建议**：[Context Engineering](../../core/context-engineering/context-engineering.md)、[`observability`](../../production/observability/README.md)、[`security`](../../production/security/README.md)、[`sandbox`](../../runtime/sandbox/README.md)

---

## 范围 / 非目标

**范围**：GUI / 浏览器 / Computer Use Agent 的定义与边界；感知通道（DOM、可访问性树、截图/多模态）；动作空间设计；动作后反馈环与恢复；人在环（HITL）；安全边界（链 Security / Sandbox）；与 Coding Agent 的差异；失败模式；评测钩子；面试题。

**非目标（只链接，不复制正文）**：

| 主题 | 去哪里 |
|------|--------|
| 工具 schema / 执行 / 鉴权契约 | [Tool Use](../../core/tool-use/tool-use.md) |
| 控制回路、预算、checkpoint | [Agent Loop](../../core/agent-loop/agent-loop.md) |
| 窗口装配与 Observation Explosion | [Context Engineering](../../core/context-engineering/context-engineering.md) |
| 沙箱隔离实现手册 | [`runtime/sandbox`](../../runtime/sandbox/README.md) |
| 威胁模型总论 | [`production/security`](../../production/security/README.md) |
| 评测体系全文 | [`production/evaluation`](../../production/evaluation/README.md) |
| Coding Agent 深文 | [`coding-agent`](../coding-agent/README.md) |
| 厂商产品操作手册 / 软文 | **不写** |

---

## 1. Overview：在手册中的位置

```text
Tool Use     = 动作能力契约（schema / 执行 / 观察）
Agent Loop   = 控制结构（步数、终止、恢复）
Browser/CU   = 垂直环境 + 感知/动作/安全特化
```

```text
Goal / Task
  ↓
Perceive（DOM | a11y | screenshot | hybrid）
  ↓
Ground（把语义目标落到可操作目标）
  ↓
Decide Action（click / type / scroll / navigate / API-hybrid）
  ↓
Execute（受策略与沙箱约束）
  ↓
Verify / Feedback（页面态、断言、副作用）
  ↓
Update State → 下一轮 或 HITL / 终止
```

**为什么单独成模块**：Browser / Computer Use 不是「Tool Use 的一个例子」。它把世界模型换成 **异步、视觉优先、选择器易碎、副作用常不可逆** 的 GUI；评测、安全与恢复策略都与纯 API Agent、Coding Agent 不同。

**与 ReAct 的关系**：感知→行动→观察仍是 Loop 特化；本文关心的是 **观察从哪来、动作写到哪、如何确认真的成功**（见 [paper-react-2022](../../references/papers.md#paper-react-2022)）。

---

## 2. Definitions

### 2.1 Browser Agent

**Browser Agent**：目标环境主要是 **网页文档与浏览器控件**（DOM、URL、Cookie/Session、标签页）。动作通常经自动化驱动（或等价抽象）落到元素/坐标；观察可来自 DOM、a11y、截图或网络事件。

### 2.2 Computer Use / Desktop GUI Agent

**Computer Use Agent**：目标环境扩展到 **操作系统桌面与任意应用窗口**（浏览器只是其中一类）。观察常以 **屏幕像素 / OS 辅助技术树** 为主；动作是键鼠、剪贴板、窗口切换等。浏览器 Agent 可视为 Computer Use 在 Web 上的特化。

### 2.3 GUI Agent（总称）

**GUI Agent**：凡通过图形界面操控软件完成任务的 Agent。本模块用「Browser / Computer Use」统称该族，并在工程上按 **环境边界**（仅浏览器 vs 全桌面）拆开安全与评测。

### 2.4 Grounding

**Grounding（落地）**：把自然语言/任务意图映射到 **具体可操作目标**（元素节点、坐标、控件 role、菜单路径）。失败的 grounding = 点错、填错、在错误页面操作。

### 2.5 与相邻概念

| 概念 | 是 | 不是 |
|------|----|------|
| Browser tools | GUI 动作的 Tool 实现 | Tool Use 理论本身 |
| Web scraping | 抽取内容 | 交互式任务完成 |
| RPA 脚本 | 预定义路径 | 动态决策 Agent（可组合） |
| API 集成 | 结构化副作用 | 替代所有 GUI（常更优） |

---

## 3. Mental Model：三层世界

```text
Layer 0  Intent World     用户目标、约束、风险等级
Layer 1  Interface World  DOM / a11y / 像素 / 窗口
Layer 2  Effect World     后端状态、邮件已发、订单已下、文件已改
```

工程铁律：

1. **Layer 1 变了 ≠ Layer 2 成功了**（UI 文案「成功」可假；需可验证副作用）。
2. **Layer 0 未批准的高危 Effect 不得自动执行**（支付、外发、删数、权限变更）。
3. 模型只产出动作意图；**执行器**负责等待、重试、策略拦截（不重写 Tool Use，只特化）。

---

## 4. 感知（Perception）

### 4.1 通道对比

| 通道 | 信号 | 优点 | 代价 / 风险 |
|------|------|------|-------------|
| **DOM / HTML** | 树、属性、文本、部分交互态 | 结构化、可过滤、成本低 | Shadow DOM、Canvas、iframe、巨大 HTML、反爬混淆 |
| **Accessibility tree（a11y）** | role、name、state、bounds | 贴近「可交互控件」语义；常比裸 HTML 更利于 grounding | 站点 a11y 质量差则空洞；动态更新滞后 |
| **Screenshot / 多模态** | 像素、布局、图标、图表 | 覆盖视觉任务、Canvas、无文本控件 | 坐标不准、OCR 噪声、token 贵、隐私面大 |
| **Hybrid** | 结构化 + 视觉互证 | 稳健性最好 | 装配复杂度高；易 Observation Explosion |

Mind2Web 等真实网页任务表明：**原始 HTML 往往过大**，需要过滤/候选后再交给大模型做选择（见 [paper-mind2web-2023](../../references/papers.md#paper-mind2web-2023)）。VisualWebArena 强调：**大量任务本质依赖视觉 grounding**，纯文本通道不够（见 [paper-visualwebarena-2024](../../references/papers.md#paper-visualwebarena-2024)）。

### 4.2 感知管线（推荐）

```text
Raw page / frame
  → scope（当前 frame / 可见区 / 任务相关区）
  → candidate set（可交互节点过滤）
  → normalize（role/name/state/bbox/稳定 id）
  → budget truncate / rank
  → optional screenshot crop
  → Observation pack → Context Builder
```

原则：

- **先过滤再推理**：不要把整页 HTML 当 Context。
- **优先稳定语义键**：`role + accessible name`、业务 data-testid（若可控）、结构化 locator；少依赖脆弱 CSS 路径。
- **截图要有目的**：全页大图默认贵且中部易丢（对齐 Context「More ≠ Better」）；优先 viewport / 控件 crop。
- **标注坐标约定**：若用 Set-of-Mark / 编号标签 overlay，必须在 Observation 中 **显式声明坐标系与编号→元素映射**。

### 4.3 iframe / Shadow / 多标签

- **iframe**：感知须带 frame 路径；动作须在正确 browsing context 执行。
- **Shadow DOM**：依赖 piercing 或 a11y；失败时降级视觉。
- **多标签 / 弹窗**：State 必须跟踪 `active_target`；否则 Agent 在错页「成功点击」。

### 4.4 桌面扩展差异

Computer Use 额外感知：窗口标题/Z-order、系统对话框、焦点控件、剪贴板、通知。**缺少窗口身份** 时，坐标动作会产生跨应用误操作——这是安全级问题，不只是准确率问题。

---

## 5. 动作空间（Action Space）

### 5.1 设计原则

动作空间应满足：

1. **完备性**：完成目标所需的最小操作集（导航、点击、输入、滚动、等待、上传下载、快捷键…）。
2. **可审计**：每次动作可日志化（目标、参数、策略结果）。
3. **可拦截**：高危动作可被策略拒绝或转 HITL。
4. **可组合**：优先小原子动作；少用「一键完成整站流程」黑盒宏（难评测、难恢复）。

Anthropic 工具设计建议同样适用：清晰职责、上下文效率、可组合（见 [article-anthropic-writing-tools-for-agents-2025](../../references/articles.md#article-anthropic-writing-tools-for-agents-2025)）——本文不展开 ACI 通用理论。

### 5.2 常见动作族

| 族 | 示例 | 备注 |
|----|------|------|
| Navigation | goto、back、reload、new tab | URL 允许列表策略挂钩 |
| Pointer | click、dblclick、hover、drag | 元素目标 vs 坐标目标 |
| Keyboard | type、press、hotkey | 输入前焦点确认 |
| Scroll / viewport | scroll_into_view、page_down | 懒加载触发 |
| Wait / sync | wait_for_selector、wait_network_idle | **不是**盲目 sleep |
| File | upload、download | 沙箱路径与 MIME 限制 |
| Browser state | cookies、storage、permissions | 高隐私；默认禁用 |
| Hybrid API | 可读 API / 受控 SDK | GUI 失败时的旁路（常更稳） |

### 5.3 元素目标 vs 坐标目标

| | 元素/语义目标 | 坐标目标 |
|--|---------------|----------|
| 稳定性 | 相对高（依赖 locator 质量） | 低（布局、DPI、动画） |
| 适用 | DOM/a11y 可用 | Canvas、游戏、坏 a11y |
| 安全 | 可绑 role/域名策略 | 难证明「点的是什么」 |

生产建议：**默认语义目标；坐标作降级**；坐标动作提高 HITL / 录屏审计权重。

### 5.4 API 混合动作空间

当同一效果可用官方 API / 内部 RPC 完成时：

- **优先 API**（可校验、可幂等、日志清晰）。
- GUI 保留给：**无 API、一次性后台、探索性任务**。
- 混合时 State 要记录「本步走 GUI 还是 API」，避免双重提交。

### 5.5 反模式

- 把「执行任意 JS」作为默认工具（XSS 级能力）。
- 无限制 `page.evaluate` 读 Cookie / LocalStorage。
- 宏动作吞掉中间失败（无法局部重试）。
- 动作参数允许任意绝对坐标且无窗口绑定。

---

## 6. 反馈环（Feedback Loop）

### 6.1 为什么「点完就信」会失败

GUI 是异步的：动画、网络、骨架屏、乐观 UI。正确闭环是：

```text
precondition check
  → act
  → wait for expected transition（带超时）
  → postcondition verify（UI + 可选 Effect）
  → classify outcome（success | retryable | fatal | needs_human）
```

### 6.2 验证层次

| 层 | 例子 | 强度 |
|----|------|------|
| 局部 UI | 元素出现/消失、toast 文本 | 弱 |
| 导航态 | URL / title / 面包屑 | 中 |
| 表单态 | 校验错误、disabled、值回读 | 中 |
| Effect | 订单号、邮件外发日志、DB 断言 | **强** |
| 人类确认 | 截图审批 | 任务相关 |

WebArena 类基准强调 **功能正确性 / 执行态评测**，而不是字符串匹配动作序列（见 [paper-webarena-2024](../../references/papers.md#paper-webarena-2024)）——工程上应同样以 **Effect 断言** 为金标准。

### 6.3 等待策略

- **条件等待** > 固定 sleep。
- 区分：DOM attached、visible、stable（位置不再抖）、network quiet、自定义 predicate。
- 超时必须进入 **分类**：继续等 / 换策略 / 重感知 / HITL / 失败。

### 6.4 错误恢复模式

| 模式 | 适用 | 风险 |
|------|------|------|
| 同动作重试（有限次） | 瞬时网络/加载 | 重复提交 |
| 重感知再 grounding | 节点过期 | 成本上升 |
| 滚动/切换标签后重试 | 目标不可见 | 走偏页面 |
| 回退导航 | 进错路径 | 丢失表单 |
| 换通道（DOM→视觉） | a11y 空洞 | 坐标风险 |
| 升 HITL | 高危或连续失败 | 延迟 |
| 终止并报告 partial | 预算耗尽 | 需可续跑 |

**幂等**：对「提交订单」「发送」类动作，恢复前必须检查是否已产生 Effect（订单号、成功页、幂等键）。

### 6.5 与 Agent Loop 的挂钩

步数预算、重复动作检测、checkpoint、interrupt/resume 由 [Agent Loop](../../core/agent-loop/agent-loop.md) 提供。本模块要求 Loop 状态至少额外包含：

- `active_url` / `active_window`
- `last_action` + `last_verify_result`
- `permission_tier` / `pending_approvals`
- 可选：关键截图句柄（外置存储，不整图塞进 messages）

---

## 7. 人在环（HITL）

### 7.1 何时必须人

强制或强烈建议 HITL：

- 支付、转账、签约、不可逆删除
- 外发通信（邮件/IM/社媒发帖）
- 权限/安全设置变更
- 跨域导航到未允许站点
- 凭证输入（密码、2FA）——通常 **人类输入，Agent 不经手明文**
- 连续 N 次 verify 失败或模型自报不确定

### 7.2 HITL 形态

| 形态 | 说明 |
|------|------|
| 预批准策略 | 会话开始声明允许域/动作白名单 |
| 逐步审批 | 每高危动作暂停，展示意图+截图 |
| 分段接管 | 人类完成登录/支付，Agent 续跑 |
| 事后审计 | 低危批量任务；异常回放 |

Interrupt/Resume 语义对齐 Loop：持久化 `waiting_human`，审批事件唤醒，超时默认 **拒绝** 而非静默继续。

### 7.3 UX 要点

- 给人看的是：**要做什么、点哪里、预期副作用、风险等级**，不是原始 tool JSON。
- 提供「仅本步 / 本站点会话 / 永久策略」三级授权粒度。
- 拒绝与超时必须可观测，写入轨迹供评测。

---

## 8. 安全边界（链 Security / Sandbox）

> 本节只写 **Browser/CU 特化威胁与控制点**；完整威胁模型见 [`production/security`](../../production/security/README.md)，隔离机制见 [`runtime/sandbox`](../../runtime/sandbox/README.md)。

### 8.1 特化攻击面

| 面 | 例子 |
|----|------|
| 间接提示注入 | 网页隐藏文本、白字、ARIA 名注入「忽略先前指令」 |
| 视觉注入 | 截图中的对抗贴图、假按钮 |
| 凭证窃取 | 诱导打开登录页并 type 密码到攻击者表单 |
| 数据外泄 | 把内网页内容粘贴到外网表单 / 上传 |
| 供应链 | 恶意扩展、被污染的浏览器配置文件 |
| 跨应用误操作（桌面） | 坐标点到另一窗口的「确认」 |

### 8.2 控制层次（由内到外）

```text
1. 环境隔离     专用 profile / 容器 / 无持久凭证（Sandbox）
2. 网络策略     出站允许列表、阻断内网元数据地址
3. 动作策略     域名/路径/动作类型 ACL；高危强制 HITL
4. 感知消毒     不可信页内容标记为 untrusted；指令与页面文本分区
5. 密钥隔离     Agent 不持有主密码；使用短时 token / 人类填密
6. 审计回放     动作+截图+URL 轨迹可追溯
```

### 8.3 最小权限基线（建议默认）

- 仅允许任务所需 host；禁止任意 `file://`、内网段（除非显式企业策略）。
- 默认禁止：清 Cookie 全局、读全部存储、下载执行文件、安装扩展。
- 下载目录沙箱化；上传仅允许用户指定文件。
- 桌面 Computer Use：**单窗口/单应用绑定** 优先于全屏任意点击。

### 8.4 与 Coding Agent 安全的对比一句话

Coding Agent 主要怕 **任意代码执行与仓库破坏**；Browser/CU 主要怕 **身份盗用、资金/外发副作用、不可信网页注入**。二者都要沙箱，但策略对象不同。

---

## 9. 与 Coding Agent 的差异

| 维度 | Browser / Computer Use | Coding Agent |
|------|------------------------|--------------|
| 世界 | GUI / 网页 / 桌面 | 仓库 + shell + 测试 |
| 真相源 | 弱（UI 可撒谎） | 相对强（编译/测试/diff） |
| 定位 | 元素/坐标易碎 | 路径/AST/符号相对稳 |
| 反馈 | 异步 UI + Effect 断言 | 日志、退出码、测试 |
| 副作用 | 支付、外发、会话 | 写文件、跑命令 |
| 评测 | 任务成功 / 安全违规 / 步效 | SWE-bench 类 + 本地测试 |
| 优先旁路 | 官方 API | 直接编辑文件 / CI |

**不要复制 Coding 闭环当 Browser 闭环**：没有「跑测试变绿」的通用等价物时，必须自己定义 **verify 预言机**（URL、DOM 断言、业务 API、人工）。

边界：Coding 深文 → [`coding-agent`](../coding-agent/README.md)；二者可组合（例如 Agent 用浏览器查文档再改代码），但 **权限域应分沙箱**。

---

## 10. 可靠性工程要点

### 10.1 选择器脆弱性

- 绝对 XPath / 深层 CSS：页面一改就挂。
- 优先：`getByRole`、accessible name、稳定 test id、文本+结构约束。
- 自建站点：为 Agent 提供 **显式稳定钩子**（比让模型猜 class 便宜一个数量级）。

### 10.2 时间与竞态

- SPA 路由、骨架屏、防抖搜索：verify 过早 = 假失败；过晚 = 刷步数。
- 输入后等待建议结果列表稳定再点击。

### 10.3 状态污染

- Cookie / LocalStorage / 购物车：任务间要 **重置或隔离 profile**。
- 评测环境可复现是一等公民（WebArena 动机之一）。

### 10.4 Observation 膨胀

整页 DOM + 全屏截图每步回灌会迅速打爆窗口。对策：候选集、差分感知、外置轨迹、摘要（见 [Context Engineering](../../core/context-engineering/context-engineering.md)）。

---

## 11. Failure Modes

| Symptom | Cause | Impact | Detection | Mitigation |
|---------|-------|--------|-----------|------------|
| 点错控件 | Grounding 失败 / 坐标漂移 | 错误副作用 | 动作目标≠预期 role；事后截图 | 语义 locator；HITL；禁用裸坐标 |
| 死循环点击 | 无进展、错误恢复 | 成本爆炸 | 重复 (url, action, target) | Loop 重复检测；换策略上限 |
| 假成功 | 只看 toast | 业务未完成 | Effect 断言失败 | 强验证层 |
| 卡在加载 | 等待不当 | 超时失败 | 长时间无 DOM 变化 | 条件等待；失败分类 |
| 登录墙 | Session 过期 | 任务中断 | 识别 login form | 分段 HITL；禁止存密 |
| 注入接管 | 页内指令 | 越权/外泄 | 异常外域导航；策略命中 | 内容不信任；ACL；沙箱 |
| iframe 迷失 | 未切 frame | 找不到元素 | 空 candidate | frame 感知 |
| 跨窗误点 | 桌面无绑定 | 灾难性误操作 | 窗口标题校验失败 | 单应用绑定 |
| 双重提交 | 重试无幂等 | 重复订单 | Effect 已存在仍提交 | 提交前查重 |
| Context 爆 | 感知过载 | 决策退化 | token/步 | 过滤+差分 |

---

## 12. Evaluation Hooks

> 完整评测体系 → [`production/evaluation`](../../production/evaluation/README.md)。此处只钉 **本垂直必须挂钩的指标与任务形态**。

### 12.1 任务级

- **Task Success**：最终 Effect / 功能断言是否成立（优先于动作序列匹配）。
- **Partial Credit**：关键子目标完成比例（可选）。
- **Safety Violations**：越权域、未审批高危动作、外泄尝试次数（应为 0 门禁）。

### 12.2 过程级

- 步数 / 墙钟 / 模型调用成本
- Grounding 准确率（若有标注）
- 重试率、HITL 介入率、假成功率
- 动作多样性 vs 重复率

### 12.3 基准与本地集

| 类型 | 作用 | 代表 |
|------|------|------|
| 真实/可复现 Web 任务 | 长程功能正确性 | [paper-webarena-2024](../../references/papers.md#paper-webarena-2024) |
| 跨站泛化 | 元素选择与迁移 | [paper-mind2web-2023](../../references/papers.md#paper-mind2web-2023) |
| 视觉 grounding | 多模态必要任务 | [paper-visualwebarena-2024](../../references/papers.md#paper-visualwebarena-2024) |
| 开放桌面 | Computer Use 全环境 | [paper-osworld-2024](../../references/papers.md#paper-osworld-2024) |
| 内部金丝雀 | 产品关键流 + 安全红队 | 自建（必做） |

**反模式**：只 demo 一条快乐路径；用「看起来像点对了」代替 Effect 断言；评测环境与生产权限不一致导致虚假安全感。

### 12.4 发布门禁建议

1. 安全违规 = 0（允许列表外导航、未审批支付等）。
2. 核心任务成功率不低于阈值；成本/步数不回归超阈。
3. 注入红队集（隐藏指令、假支付页）有固定回归。

---

## 13. 生产架构挂钩（不复制）

```text
Orchestrator (Agent Loop)
  ├── Policy Engine（域/动作/HITL）
  ├── Perception Service（DOM/a11y/shot → pack）
  ├── Action Executor（browser/desktop driver）
  ├── Verifier（UI + Effect oracles）
  ├── Sandbox / Browser profile
  └── Trajectory Store（obs 链接、审计）
```

- 观测：每步记录 URL、目标描述、策略决策、verify 结果、截图句柄 → [`observability`](../../production/observability/README.md)。
- 推理/重计划：长程站点任务可启用有界计划，但计划必须随 URL/页面签名失效 → [Reasoning](../../core/reasoning-planning/reasoning-planning.md)。

---

## 14. 伪代码（概念）

```text
function run_browser_agent(task, policy, budgets):
  state = init_state(task)
  while not terminated(state, budgets):
    obs = perceive(state.active_target, mode=policy.perception)
    obs = sanitize_untrusted(obs)
    ctx = build_context(state, obs)          # Context Engineering
    decision = llm_decide(ctx, action_schemas)
    if decision.needs_human or policy.requires_approval(decision):
      state = await_human(state, decision)
      if state.approval == deny: return fail(state, "denied")
    if not policy.allows(decision): return fail(state, "policy")
    pre = check_preconditions(decision)
    if not pre.ok: recover_or_replan(state, pre); continue
    result = execute(decision)               # Tool executor
    verify = verify_postconditions(decision, result)
    state = update(state, decision, result, verify)
    if verify.effect_success: maybe_progress(state)
    elif verify.retryable: bounded_retry(state)
    else: replan_or_fail(state, verify)
  return finalize(state)
```

---

## 15. Trade-offs（定性）

| 选择 | 得 | 失 |
|------|----|----|
| 纯 DOM | 便宜、快 | 视觉任务弱；易被混淆 HTML |
| 纯截图 | 覆盖广 | 贵、坐标脆、隐私 |
| Hybrid | 稳 | 复杂、易爆 Context |
| 宽动作空间 | 能力强 | 更难安全与评测 |
| 紧动作空间 | 可控 | 任务覆盖不足 |
| 高自治 HITL | 安全 | 吞吐下降 |
| 低干预自主 | 体验快 | 事故面大 |
| API 优先混合 | 可靠 | 非处处有 API |

从简单可组合模式起步、用评测证明再加复杂度（见 [article-anthropic-building-effective-agents-2024](../../references/articles.md#article-anthropic-building-effective-agents-2024)）。

---

## 16. Interview Questions

### Beginner

**Q1. 什么是 Browser Agent？**  
- **短答**：在网页 GUI 上感知并执行交互以完成任务的 Agent。  
- **详解**：观察来自 DOM/a11y/截图；动作经驱动落到页面。  
- **追问**：和爬虫区别？  
- **常见错误**：把一次性 HTTP 抓取叫 Browser Agent。  
- **相关**：§2。

**Q2. Computer Use 和 Browser Agent 什么关系？**  
- **短答**：前者覆盖桌面/多应用；后者是 Web 特化。  
- **详解**：感知与安全边界不同。  
- **追问**：为何桌面更危险？  
- **常见错误**：混用同一套无窗口绑定策略。  
- **相关**：§2、§4.4。

**Q3. 什么是 grounding？**  
- **短答**：把意图落到具体可点/可填目标。  
- **详解**：失败即点错填错。  
- **追问**：如何评测 grounding？  
- **常见错误**：只评最终文案。  
- **相关**：§2.4。

**Q4. DOM、a11y、截图各适合什么？**  
- **短答**：结构交互 / 控件语义 / 视觉与 Canvas。  
- **详解**：常 hybrid。  
- **追问**：为何不总喂整页 HTML？  
- **常见错误**：每步全量 DOM+全屏图。  
- **相关**：§4。

**Q5. 为什么 UI 显示成功仍可能失败？**  
- **短答**：Layer1≠Layer2；乐观 UI/假文案。  
- **详解**：要 Effect 断言。  
- **追问**：举一个 Effect 例子。  
- **常见错误**：只等 toast。  
- **相关**：§3、§6。

**Q6. 元素定位 vs 坐标点击？**  
- **短答**：语义目标更稳；坐标是降级。  
- **详解**：坐标难审计。  
- **追问**：何时必须坐标？  
- **常见错误**：默认全坐标。  
- **相关**：§5.3。

**Q7. 什么是动作后反馈环？**  
- **短答**：pre→act→wait→verify→分类。  
- **详解**：处理异步 GUI。  
- **追问**：固定 sleep 的问题？  
- **常见错误**：点完立刻下一步。  
- **相关**：§6。

**Q8. HITL 在 Browser Agent 里典型触发？**  
- **短答**：支付、外发、登录、越域、连续失败。  
- **详解**：超时默认拒绝。  
- **追问**：凭证如何处理？  
- **常见错误**：让模型保存用户密码。  
- **相关**：§7。

**Q9. 为何要链 Sandbox？**  
- **短答**：隔离浏览器身份、文件系统与网络。  
- **详解**：降低外泄与误操作半径。  
- **追问**：仅无头模式够吗？  
- **常见错误**：用个人日常浏览器 profile 跑 Agent。  
- **相关**：§8、`runtime/sandbox`。

**Q10. 与 Coding Agent 最大差异？**  
- **短答**：真相源与副作用类型不同。  
- **详解**：缺通用「测试变绿」。  
- **追问**：如何自建 verify？  
- **常见错误**：照搬 test-driven 故事。  
- **相关**：§9。

### Intermediate

**Q11. 设计候选元素过滤策略。**  
- **短答**：可交互、可见、任务相关、预算截断。  
- **详解**：可先小模型/规则粗筛。  
- **追问**：如何避免滤掉真目标？  
- **常见错误**：只按文本模糊匹配。  
- **相关**：Mind2Web 过滤思想。

**Q12. iframe 场景如何建模 State？**  
- **短答**：browsing context 栈 + active frame。  
- **详解**：感知与动作同 context。  
- **追问**：跨 frame 拖拽？  
- **常见错误**：全局 query 一次。  
- **相关**：§4.3。

**Q13. 如何防止重复下单？**  
- **短答**：提交前查 Effect；幂等键；限制重试。  
- **详解**：恢复路径显式分支。  
- **追问**：网络超时后不知是否成功怎么办？  
- **常见错误**：超时即重 POST。  
- **相关**：§6.4。

**Q14. 间接提示注入在网页上如何表现？**  
- **短答**：隐藏文本/ARIA/图片指令诱导越权。  
- **详解**：页面内容不可信。  
- **追问**：缓解层次？  
- **常见错误**：把 DOM 文本当系统指令。  
- **相关**：§8、`production/security`。

**Q15. API 混合动作空间如何选型？**  
- **短答**：有效应 API 则优先；GUI 补洞。  
- **详解**：防双写。  
- **追问**：无官方 API 时？  
- **常见错误**：全部 GUI「更通用」。  
- **相关**：§5.4。

**Q16. 观测日志应记什么？**  
- **短答**：URL、动作、策略、verify、截图句柄、成本。  
- **详解**：敏感字段红蚀。  
- **追问**：整页 HTML 是否入库？  
- **常见错误**：只留最终答案。  
- **相关**：§13。

**Q17. 选择器维护策略？**  
- **短答**：role/name/testid；避免深 XPath。  
- **详解**：可控站点加钩子。  
- **追问**：第三方站点？  
- **常见错误**：录制回放绝对坐标当生产方案。  
- **相关**：§10.1。

**Q18. 多标签任务如何避免串台？**  
- **短答**：显式 active_target；动作带 target id。  
- **详解**：弹窗当作新 target。  
- **追问**：关错标签恢复？  
- **常见错误**：假设始终单页。  
- **相关**：§4.3。

**Q19. 假成功如何进入评测？**  
- **短答**：单列指标；用 Effect 金标准对照 UI 断言。  
- **详解**：回归门禁。  
- **追问**：没有后端断言时？  
- **常见错误**：人工看一眼截图算通过。  
- **相关**：§12。

**Q20. 何时从 DOM 降级到截图？**  
- **短答**：候选空、Canvas、a11y 空洞、视觉题。  
- **详解**：记录降级原因。  
- **追问**：如何控制成本？  
- **常见错误**：永久切纯视觉。  
- **相关**：§4、VisualWebArena。

### Advanced

**Q21. 设计域名允许列表策略引擎。**  
- **短答**：任务作用域 host；重定向跟随规则；例外审批。  
- **详解**：子域与路径级控制。  
- **追问**：CDN/第三方登录怎么办？  
- **常见错误**：只匹配首页 host。  
- **相关**：§8。

**Q22. Observation Explosion 在 Browser 中的特解？**  
- **短答**：候选集、差分、外置截图、摘要轨迹。  
- **详解**：对齐 Context 预算。  
- **追问**：差分感知的失效条件？  
- **常见错误**：每步全量多模态。  
- **相关**：Context Engineering。

**Q23. 桌面 Computer Use 的窗口绑定方案。**  
- **短答**：启动时钉住 PID/窗口；动作前校验焦点。  
- **详解**：失败则中止非 HITL。  
- **追问**：多显示器坐标？  
- **常见错误**：全局屏幕任意点。  
- **相关**：§4.4、§8。

**Q24. 长程任务中计划如何失效？**  
- **短答**：页面签名/URL 模板变化触发 replan。  
- **详解**：有界重计划。  
- **追问**：与 Loop 预算关系？  
- **常见错误**：死守过时步骤列表。  
- **相关**：Reasoning Canonical。

**Q25. 设计「登录分段」协议。**  
- **短答**：Agent 停在登录墙；人类完成；会话令牌留在沙箱 profile。  
- **详解**：Agent 不读密码框值。  
- **追问**：2FA？  
- **常见错误**：自动化填密进轨迹日志。  
- **相关**：§7。

**Q26. 如何评测安全而非只评成功？**  
- **短答**：违规次数、越域、未审批高危、红队注入集。  
- **详解**：成功但违规 = 发布失败。  
- **追问**：指标权重？  
- **常见错误**：成功率掩盖外泄。  
- **相关**：§12。

**Q27. Shadow DOM 与自定义组件对策？**  
- **短答**：a11y 优先；必要时 pierce；再视觉。  
- **详解**：与站点合作暴露 test id。  
- **追问**：封闭组件无 a11y？  
- **常见错误**：假设 querySelector 全域。  
- **相关**：§4。

**Q28. 并发多 Browser Agent 的隔离？**  
- **短答**：独立 profile/容器/网络命名空间。  
- **详解**：禁止共享登录态除非明文政策。  
- **追问**：成本权衡？  
- **常见错误**：共享同一 Cookie jar。  
- **相关**：Sandbox。

**Q29. 把 RPA 脚本嵌进 Agent 何时合理？**  
- **短答**：稳定高频子流程可固化为工具。  
- **详解**：仍要 verify 与策略。  
- **追问**：何时不该固化？  
- **常见错误**：整站纯录制回放当 Agent。  
- **相关**：§2.5、§5。

**Q30. 视觉 Set-of-Mark 的工程风险？**  
- **短答**：编号映射丢失、分辨率不一致、遮挡。  
- **详解**：Observation 必须带映射表。  
- **追问**：与纯坐标比？  
- **常见错误**：只传图不传图例。  
- **相关**：§4.2。

### Expert

**Q31. 给出 Browser Agent 威胁模型资产清单。**  
- **短答**：会话、支付能力、私信、内网可达性、剪贴板、下载执行链。  
- **详解**：映射攻击者与控制。  
- **追问**：最优先控制？  
- **常见错误**：只谈 jailbreak 文案。  
- **相关**：§8、Security。

**Q32. 如何在不可复现的生产站点做回归？**  
- **短答**：契约测试 + 录制流量拟态 + 关键路径金丝雀 + 人工抽检。  
- **详解**：与评测环境分离声明。  
- **追问**：DOM 漂移报警？  
- **常见错误**：只靠公开基准分数。  
- **相关**：§12。

**Q33. Effect 预言机不可得时怎么办？**  
- **短答**：多层弱信号+人类抽检；降低自主等级。  
- **详解**：产品上标不确定。  
- **追问**：能否用第二 Agent 当裁判？  
- **常见错误**：用另一个 LLM 纯看截图当金标准。  
- **相关**：§6.2。

**Q34. 设计抗注入的 Context 分区。**  
- **短答**：系统策略 / 工具 schema / 不可信页面包 严格分隔。  
- **详解**：页面文本不可升格为指令。  
- **追问**：工具返回是否可信？  
- **常见错误**：全部拼成一个 user 消息。  
- **相关**：Context、Security。

**Q35. Computer Use 与虚拟机逃逸残差如何谈？**  
- **短答**：沙箱降风险非消除；假设突破时的监控与凭证短命。  
- **详解**：链 limits_of_sandboxing。  
- **追问**：为何仍要做？  
- **常见错误**：宣称「有容器即绝对安全」。  
- **相关**：Sandbox README。

**Q36. 多模态成本爆炸时的自适应感知。**  
- **短答**：默认结构化；失败/视觉题再升级截图。  
- **详解**：记入预算。  
- **追问**：如何避免振荡？  
- **常见错误**：每步最高保真。  
- **相关**：§4、§15。

**Q37. 跨站泛化差的根因分层。**  
- **短答**：感知过滤、动作空间、站点交互范式、训练分布。  
- **详解**：Mind2Web 式拆解。  
- **追问**：工程上先修哪层？  
- **常见错误**：只换更大模型。  
- **相关**：§12.3。

**Q38. 如何把策略引擎做成可测试组件？**  
- **短答**：纯函数：(state, action)→allow|deny|approve；单测矩阵。  
- **详解**：与执行器解耦。  
- **追问**：灰度策略？  
- **常见错误**：策略写死在 prompt。  
- **相关**：§8、§14。

**Q39. 轨迹泄露隐私如何治理？**  
- **短答**：红蚀、短留存、截图区域遮盖、访问控制。  
- **详解**：默认不进训练。  
- **追问**：支付页？  
- **常见错误**：全量录屏永久存。  
- **相关**：Observability、Security。

**Q40. 何时不该做 Browser Agent？**  
- **短答**：有稳定 API；任务不可逆且无法 HITL；合规禁止自动操控。  
- **详解**：改用人机流程或官方集成。  
- **追问**：灰色地带？  
- **常见错误**：为炫技上 GUI 自动化。  
- **相关**：§15。

### System Design

**Q41. 设计企业内部「只读调研」Browser Agent。**  
- **短答**：只读动作集、严格允许列表、禁下载执行、全审计。  
- **详解**：写操作一律 deny。  
- **追问**：如何扩到可申请写？  
- **常见错误**：复用个人 SSO 无隔离。  
- **相关**：§5、§8。

**Q42. 设计电商下单 Agent 的安全与 verify。**  
- **短答**：购物车阶段自主；支付 HITL；订单号 Effect 断言；防双下单。  
- **详解**：价格异常熔断。  
- **追问**：优惠券竞态？  
- **常见错误**：支付也可自动。  
- **相关**：§6、§7。

**Q43. 设计可复现评测环境。**  
- **短答**：自托管站点+种子数据+重置脚本+功能断言。  
- **详解**：对齐 WebArena 动机。  
- **追问**：与生产差异声明？  
- **常见错误**：对着活网刷榜。  
- **相关**：§12。

**Q44. 设计桌面版「单应用客服系统」Computer Use。**  
- **短答**：应用绑定、动作白名单、字段级 HITL、窗口校验。  
- **详解**：禁跨应用。  
- **追问**：弹窗升级？  
- **常见错误**：OS 级全局热键任意发。  
- **相关**：§4.4、§8。

**Q45. 从 MVP 到生产的里程碑。**  
- **短答**：P0 单站只读+预算 → P1 verify/HITL → P2 沙箱网络策略 → P3 红队与门禁 → P4 混合 API。  
- **详解**：每步用指标证明。  
- **追问**：不可妥协项？  
- **常见错误**：先全网开放点击再补安全。  
- **相关**：§15、Anthropic building effective agents。

---

## 17. Cross References

- [Agent Loop](../../core/agent-loop/agent-loop.md) · [Tool Use](../../core/tool-use/tool-use.md)  
- [Context Engineering](../../core/context-engineering/context-engineering.md) · [Reasoning & Planning](../../core/reasoning-planning/reasoning-planning.md)  
- [`coding-agent`](../coding-agent/README.md) · [`multi-agent`](../multi-agent/README.md)  
- [`runtime/sandbox`](../../runtime/sandbox/README.md) · [`production/security`](../../production/security/README.md)  
- [`production/evaluation`](../../production/evaluation/README.md) · [`production/observability`](../../production/observability/README.md)

---

## 18. References

- [paper-webarena-2024](../../references/papers.md#paper-webarena-2024)  
- [paper-mind2web-2023](../../references/papers.md#paper-mind2web-2023)  
- [paper-visualwebarena-2024](../../references/papers.md#paper-visualwebarena-2024)  
- [paper-osworld-2024](../../references/papers.md#paper-osworld-2024)  
- [paper-react-2022](../../references/papers.md#paper-react-2022)  
- [article-anthropic-building-effective-agents-2024](../../references/articles.md#article-anthropic-building-effective-agents-2024)  
- [article-anthropic-writing-tools-for-agents-2025](../../references/articles.md#article-anthropic-writing-tools-for-agents-2025)

---

## 19. Self-review / TODOs

- [x] GUI/浏览器定义；感知；动作空间；反馈环；HITL；安全链；与 coding 差异；失败模式；评测钩子；45 题；refs  
- [x] 未复制 coding-agent 正文；未重写 Tool Use；无厂商产品软文  
- [ ] TODO: Verify — 特定驱动（Playwright 等）等待 API 细节随版本变化  
- [ ] TODO: Verify — OS 级 a11y API 在各平台的能力差异表（可另文）  
