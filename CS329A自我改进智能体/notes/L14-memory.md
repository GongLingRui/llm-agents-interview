# L14 — Augmenting Agents with Memory

> **来源**：**无录音 / 论文整理**（仓库无本讲完整录音；客座 **Junchen Jiang**，LMCache / UChicago）  
> **日期**：Fri Nov 7（Autumn 2025）  
> **必读**：Cartridges · MemGPT · CacheBlend  
> **相关**：客座摘要见 [guest-lectures.md](guest-lectures.md)（Junchen / LMCache 节）

---

## 1. 一句话总览

Agent 要跑长程任务，必须在「有限上下文窗口」下管理**工作记忆、长期档案与跨请求知识复用**。三条技术路线互补：**OS 式虚拟内存（MemGPT）**、**离线把语料蒸馏进可加载 KV Cartridge（Cartridges）**、**服务端融合多段预计算 KV（CacheBlend / LMCache）**——分别偏 Agent 控制流、参数化记忆对象、与推理系统层。

---

## 2. 核心概念与定义

### 2.1 为什么 Memory 是 agent 讲次

呼应 L01 / L05 / L17：

- 多步工具调用后上下文爆炸  
- 目标、进度、环境模型（如 codebase 先验）要跨回合保留  
- Claude Code 等产品的 compaction = 工程版记忆管理  
- L20 Q&A：continual learning 可以靠「更大有效上下文 / Cartridge」，不必事事改权重  

### 2.2 三个层次的「记忆」

| 层次 | 问题 | 代表 |
|------|------|------|
| **Agent 逻辑记忆** | 模型如何主动换页、归档、召回？ | MemGPT |
| **语料级压缩记忆** | 同一大语料被反复问，如何不再次 prefill？ | Cartridges |
| **Serving 层 KV 复用** | RAG 多 chunk 非前缀时如何复用 KV？ | CacheBlend / LMCache |

### 2.3 KV cache 为何成为记忆载体

- Transformers 的「已读上下文」物理上就是 KV  
- 显存随长度线性涨 → 吞吐塌陷（Cartridges 文：8B 模型 1k→120k 峰值吞吐可降数十倍）  
- 故「记忆系统」常 = **如何存、压、融、迁 KV**

---

## 3. 方法 / 机制

### 3.1 MemGPT：把 LLM 当成操作系统

**论文**：*MemGPT: Towards LLMs as Operating Systems* (arXiv:2310.08560)

#### 核心类比

| OS | MemGPT |
|----|--------|
| 物理内存 | Main context（prompt tokens） |
| 磁盘 | External context（archival / recall DB） |
| 页故障 / 换页 | 函数调用检索与驱逐 |
| 虚拟内存幻觉 | 「无限上下文」幻觉 |

#### Main context 三段

1. **System instructions**（只读）：控制流、记忆层级用法、函数 schema  
2. **Working context**（固定大小读写）：人设、用户偏好、关键事实——经函数写入  
3. **FIFO queue**：对话、系统警告、函数 I/O；队首可有 **recursive summary**  

#### Queue Manager

- 新消息入队 → 拼 prompt → 推理 → 写入 recall storage  
- **Memory pressure**：达 warning token count（如 70%）插入系统警告，促模型把重要信息写入 working / archival  
- **Flush**：达上限驱逐部分消息，更新递归摘要；被驱逐内容仍可经函数召回  

#### Function executor

- 补全解析为函数调用；错误回灌  
- `request_heartbeat=true` 等机制允许**函数链式多步检索**再对用户说话  
- 自我导向：何时换页、改写 working context，由模型决定（prompt 教 OS 规则）

#### 评测域

- **文档分析**：超窗口长文档  
- **多会话聊天**：跨 session 人设一致与长期记忆  

#### 工程要点

- 记忆编辑必须 **token-aware**（分页检索防撑爆）  
- 控制流事件化：用户消息 / 系统警告 / 函数返回  
- 与「纯 RAG 每次检索拼进 prompt」相比：MemGPT 强调**持久工作集 + 自我管理**

#### 局限（整理视角）

- 依赖模型可靠 function calling  
- 摘要与换页策略错误 → 沉默遗忘  
- 不直接解决「技能写入权重」（L20：技能迁移仍可能要 FT）

---

### 3.2 Cartridges：Self-Study 蒸馏出的可加载记忆盒

**论文**：*Cartridges: Lightweight and general-purpose long context representations via self-study* (arXiv:2506.06266)

#### 动机

用户反复把同一语料（代码库、法律、病历、教材）塞进窗口做 ICL → **每次预填同一 KV 极贵**。  
若准备成本可摊销到多次查询，可离线训练一个更小的 KV——**Cartridge**——推理时加载。

#### 为何不能 naive NTP？

- 对语料做 next-token prediction 训 KV：**能背**，记忆占用可降两个数量级  
- 但会破坏对**多样提问**的应答 → 不像 ICL 那样通用  

#### Self-Study 两步

1. **合成对话**：模型就语料「自问自答」生成 conversation traces；超长语料分块；**seed prompts** 偏向全局 / 结构推理  
2. **Context distillation**：让「仅加载 Cartridge」的下一 token 分布对齐「全文在上下文中的 ICL 教师」  

训练时冻结 LLM，主要反传到 Cartridge（prefix-tuning 式虚拟 token KV）。

#### 结果声明（论文）

- 挑战性长上下文基准上匹配 ICL，同时约 **38.6×** 更少记忆、**26.4×** 更高吞吐（多用户不同语料服务场景）  
- **上下文外推**：如 MTOB，从 484k textbook 做 Cartridge，可超过仅 ICL 前 130k  
- **可组合**：多 Cartridge 推理时拼接，无需联合重训  
- KV 参数化优于 LoRA 式注入（文内消融）  

#### 与压缩基线

Prompt / KV drop 类方法在 >2× 压缩时常质量陡降；Cartridge 用**更多离线算力**换在线记忆——适合「语料稳定、查询多」的 agent 记忆。

#### 工程要点

- 识别「可摊销语料」（repo、政策库、客户 KB）  
- 建 self-study 数据管线 + 蒸馏目标  
- Serving 侧把 Cartridge 当 per-user / per-corpus KV 对象（与现有 paged attention 服务模型契合）  
- L20：作为 **不改权重的 continual / 长记忆** 路径被点名  

---

### 3.3 CacheBlend：RAG 多段 KV 的选择性重算融合

**论文**：*CacheBlend: Fast Large Language Model Serving for RAG with Cached Knowledge Fusion* (arXiv:2405.16444；EuroSys’25；Jiang 等)

#### 痛点

- Prefill 决定 TTFT，随长度超线性变贵  
- **Prefix caching**：只有前缀 KV 可安全复用；RAG 常是**多 chunk 非单一前缀** → 复用率低  
- **Full KV reuse**：强行复用非前缀 KV，忽略与前文的 **cross-attention** → 质量伤在「必须联合理解多段」的问题上  

#### 方法：Selective KV recompute

- 仍分层做 prefill，但每层只**更新一小部分 token 的 KV**（经验上常 <15%），其余复用预计算  
- 利用注意力稀疏：少量更新可近似恢复跨段依赖  
- 重算与从 CPU/磁盘取下一层 KV **流水线化** → 可把 KV 放慢存储而不增加延迟  

#### 效果（论文）

- 相对 full recompute：TTFT **2.2–3.3×**↓，吞吐 **2.8–5×**↑，质量接近  
- 相对 prefix caching：多 chunk 场景显著更快  
- 相对 full reuse：QA / 摘要质量指标更高  

#### LMCache 生态（客座主线）

Junchen Jiang 组将 CacheBlend 等纳入 **LMCache**：企业级 KV 层，支持 offload、跨引擎共享、非前缀 reuse、可观测性。Agent / RAG / 多轮对话是核心受益负载。详见客座摘要。

---

## 4. 三条路线如何拼进同一个 Agent

```text
用户目标
  │
  ├─ MemGPT 控制环：working / FIFO / archival 换页
  │     │
  │     └─ 工具：检索、写档案、调代码执行…
  │
  ├─ 稳定大语料（repo / 手册）──离线 Self-Study──► Cartridge 热加载
  │
  └─ RAG 多文档 ──预计算 chunk KV──► CacheBlend / LMCache 融合 prefill
```

设计问题清单：

1. 哪些状态必须在 **working memory**（目标、约束、当前假设）？  
2. 哪些知识是 **稳定语料**（适合 Cartridge）vs **一次性检索**（适合 CacheBlend）？  
3. 换页失败时的用户可观测性（MemGPT warning）？  
4. Serving 是否允许 KV 出 GPU（LMCache）？  

---

## 5. 与课程其他讲的连接

| 讲次 | 连接 |
|------|------|
| L02 | 多采样放大 KV 成本 → 更需复用 / Cartridge |
| L05 | 多步计划依赖状态记忆 |
| L08 Deep Research | 多源检索 + 综合 = CacheBlend + 工作记忆 |
| L13 SWE | 全库扫描摊销、prompt cache；repo Cartridge 想象空间 |
| L17 | 长时程失败常含「丢目标 / 无环境模型」；compaction |
| L20 | Continual ≈ 记忆 vs 改权重；IPW 与本地 KV |

---

## 6. 工程可落地 checklist

- [ ] 画出 agent 的记忆层级图（工作集 / 档案 / 原料语料 / 检索库）  
- [ ] 对超窗口对话：实现 pressure 警告 + 摘要 + 可召回存储（MemGPT 模式）  
- [ ] 对高频稳定语料：评估 Cartridge / 离线 KV 训练 ROI  
- [ ] RAG 服务：测量 prefix hit rate；多 chunk 时试点 CacheBlend  
- [ ] KV 分层存储（GPU / CPU / disk）+ 与 compute 流水线  
- [ ] 监控：TTFT、缓存命中、换页次数、摘要导致的事实丢失率  
- [ ] 别用「只增大 context window」替代记忆架构——L20：百万上下文仍会用不动  

---

## 7. 本讲论文速查

| 论文 | 卡片 |
|------|------|
| MemGPT | [papers/memgpt.md](../papers/memgpt.md) |
| Cartridges | [papers/cartridges.md](../papers/cartridges.md) |
| CacheBlend | [papers/cacheblend.md](../papers/cacheblend.md) |
| LMCache（扩展） | [papers/lmcache.md](../papers/lmcache.md) |

---

## 8. 若你只记 5 条

1. 有限窗口下要 **虚拟上下文**：换页、归档、召回（MemGPT）。  
2. 反复使用的大语料应 **离线压成 Cartridge**，别每次 ICL 全文。  
3. Naive 背诵式训练会毁通用性；要用 **Self-Study + context distillation**。  
4. RAG 多 chunk：**选择性重算**才能又快又保跨段注意力（CacheBlend）。  
5. 记忆是 **agent 策略 + 模型对象 + serving 系统** 的共建，不是单一模块。

---

## 9. 无录音说明

本讲为客座 + 论文课，仓库无逐字稿。笔记覆盖三篇必读机制与和 Junchen / LMCache 公开材料的衔接；**课堂现场 demo、口播超参、Q&A 细节可能缺失**，已在 guest-lectures 中标注「信息来源有限」。
