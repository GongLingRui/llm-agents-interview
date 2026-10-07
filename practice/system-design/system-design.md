# Agent System Design（Practice）

> **一句话主题**：用可评分的系统设计题，训练「目标→约束→架构→Loop→Tools/Memory/Context→Eval→Obs→Security→权衡」的完整叙事。
>
> **状态**：Practice Canonical · **Last reviewed**: 2026-10  
> **受众**：准备系统设计面试与落地架构评审的工程师  
> **硬前置**：[Agent Architecture](../../core/agent-architecture/agent-architecture.md)、[Agent Loop](../../core/agent-loop/agent-loop.md)

---

## 1. 作答框架（强制顺序）

```text
1. Goal & Non-goals
2. Users / Trust boundaries
3. Constraints（SLA、成本、合规、工具权限）
4. High-level architecture（Control vs Data Plane）
5. Loop & Tool surface
6. State / Memory / RAG / Context 策略
7. Evaluation & Observability
8. Security & Sandbox
9. Failure modes & degradation
10. Trade-offs & evolution path（P0→P7）
```

**评分维度**：边界清晰、可演进、可观测、可控制成本、安全默认、不默认 Multi-Agent。

机制细节链回 Canonical，本题只考组合与权衡。

---

## 2. 题库（10）

### SD1. 内部知识问答 Agent（带行动）

**题**：客服知识库问答，偶发需要建工单/退款查询。  
**追问**：何时 RAG、何时工单工具？注入如何防？  
**好答要点**：Evidence untrusted；高危工具 HITL；Eval 分 retrieval vs action。

### SD2. 个人助理（跨会话）

**题**：日历+邮件摘要+偏好记忆。  
**追问**：记什么/不记什么？压缩策略？  
**好答要点**：写闸；PII；Context budget；不用窗口当 DB。

### SD3. Coding Agent（单体仓）

**题**：本地/CI 修测失败。  
**追问**：上下文爆炸？沙箱？  
**好答要点**：JIT 文件；测试反馈环；Sandbox；禁止整仓。

### SD4. 数据分析 Agent

**题**：连数仓只读查询并出图表说明。  
**追问**：SQL 注入式危害？大结果集？  
**好答要点**：只读角色；结果抽样+句柄；政策扫描。

### SD5. 浏览器购票助手

**题**：在指定站点完成查询与下单草稿。  
**追问**：支付？验证码？  
**好答要点**：HITL 支付；凭证隔离；轨迹审计；脆弱 UI。

### SD6. 多工具企业 Agent 平台

**题**：百个内部 API 要以 MCP/本地工具接入。  
**追问**：工具爆炸？权限？  
**好答要点**：工具目录治理；最小权限；路由/检索工具；Eval 门禁。

### SD7. 研究型长程 Agent

**题**：数小时文献调研产出备忘录。  
**追问**：上下文焦虑/compaction？  
**好答要点**：阶段目标；compaction+句柄；引用保留；成本防火墙。

### SD8. 实时语音/低延迟助手

**题**：TTFT 敏感。  
**追问**：与 Agent Loop 冲突？  
**好答要点**：短工具面；流式；投机/小模型路由；少检索。

### SD9. 安全敏感：代码执行租户

**题**：用户提交代码让 Agent 跑。  
**追问**：逃逸？供应链？  
**好答要点**：Sandbox 深度；网络默认关；镜像供应链；Sec 威胁模型。

### SD10. 「看起来需要 Multi-Agent」的客服

**题**：售前/售后/技术三技能。  
**追问**：一定要三 Agent 吗？  
**好答要点**：先单 Agent+路由工具/技能包；拆分需独立权限或并行证据。

---

## 3. 示例提纲（SD3 缩略）

```text
Goal: 修红测；Non-goal: 自动大规模重构
Constraints: 无生产密钥；沙箱无外网默认；p95 成本上限
Arch: Loop + repo tools + test runner in sandbox
Context: 失败日志摘要 + 相关文件句柄 + diff
Eval: 固定失败用例集；修复率/步数/token
Sec: 路径允许列表；密钥扫描
Evolve: P0 tools → P5 sandbox → P6 eval
```

---

## 4. Cross References

- [Architecture](../../core/agent-architecture/agent-architecture.md)  
- [Evaluation](../../production/evaluation/evaluation.md) · [Observability](../../production/observability/observability.md) · [Security](../../production/security/security.md) · [Infrastructure](../../production/infrastructure/infrastructure.md)  
- [interview-questions.md](../interview-questions/interview-questions.md)  
- [design-patterns.md](../../production/design-patterns/design-patterns.md)
