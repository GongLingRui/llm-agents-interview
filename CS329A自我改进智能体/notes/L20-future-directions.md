# L20 — Future Research Areas

> **来源**：有录音 · `raw-transcripts/9.未来研究方向.md`  
> **日期**：Fri Dec 5（CS329A Autumn 2025）  
> **主讲**：Aakanksha Chowdhery、Azalia Mirhoseini  
> **性质**：学期收束 + 前瞻（自改进瓶颈 × 智能能效）

---

## 1. 一句话总览

本讲把整季课程收束为两条研究前线：**(A) 如何继续推动 self-improvement 飞轮**（推理链多样性、元验证、任务自提出），以及 **(B) 如何以更低能耗交付智能**（Intelligence per Watt、本地推理、混合 serving）。核心判断：Verifier 驱动的合成数据飞轮已经证明有效，但多样性、验证与数据选择仍是瓶颈；同时云端算力爆炸要求把「智能 / 瓦特」变成一等公民指标。

---

## 2. 学期地图回顾（课上口述）

| 阶段 | 内容 | 代表思想 |
|------|------|----------|
| 前半 | LLM + test-time / train-time scaling | Verifier → reward → RL / search；数学与代码爬坡 |
| 开放进化 | Open-endedness | 不只靠 reward；可探索空间里演化（AlphaEvolve 等） |
| Agent 工作流 | Tool-use 端到端 | 环境交互驱动流程 |
| 知识与记忆 | Retrieval / memory | 真实多步工作流需要 KB |
| 规划与评测 | Multi-step + agentic eval | 下一代能力与评测 |
| 客座 | Post-training、符号 / tool、多模态机器人、推理等 | 产业与前沿交叉 |

**Agent 再定义（呼应 L01）**：有目标；与环境交互；收集反馈；用反馈纠正步骤；编排 LLM、verifier（含 LLM-as-judge）、tool call、并行搜索。当前许多场景仍是**人手写 workflow**；coding agents 已开始更多「自我驱动」。Self-improvement = 规划 + 多步推理 + 纠错 + 能力持续提升。

客座补充（Azalia）：符号技术落在 tool-use 伞下，并反哺 **synthetic data** 趋势。

---

## 3. 核心概念：两条前线

### 前线 A — Self-improvement 的下一跳

课上选了三类论文思想（未必逐页精读，抓 key idea）：

1. **多样推理链** → Multi-agent finetuning  
2. **验证瓶颈 / 元验证** → DeepSeekMath-V2 式 self-verification  
3. **训练数据 / prompt 选择瓶颈** → Absolute Zero（proposer–solver，零外部题库）

### 前线 B — Intelligence efficiency

1. **Intelligence per Watt (IPW)**：任务准确率 / 平均功耗  
2. 本地小模型已覆盖大量真实查询 → 流量可从云重分配到边  
3. 混合 routing、能效内核 / 架构、以能量为一等优化目标  

---

## 4. 方法 / 机制详解

### 4.1 Multi-agent finetuning：用多样性续命自改进

#### 问题

- Pretraining = 压缩互联网级多样数据  
- Instruction tuning 常需人类偏好数据  
- 替代路径：合成数据 + rejection sampling / STaR 类迭代微调  
- **单模型**即使高温采样，轨迹仍同质 → 迭代数十轮后性能平台 / 崩溃  

#### 机制（课上算法要点）

- 训练多类专家：  
  - **Generation agents**：产生多样初始解  
  - **Critic agents**：评估与精炼  
- 多轮「辩论」流程：  
  1. 各 generation agent 给初始答案  
  2. **汇总**（模型摘要或直接拼接）  
  3. Critic 对更新后的答案集合做 critique  
  4. 各 agent 基于摘要再生成 → majority vote → 再摘要……  
- Generation 模型从同一 base 微调；过滤与 majority 一致的 (prompt, response) 做 SFT  
- Critic 在「一开始就对」与「辩论中被纠正」的轨迹上微调，学习对比对错答案  
- 「穷人版」：直接用不同模型 / 不同 prompt 混采样（课上承认实践中常用）

#### 结果要点（课上）

- 横轴：微调迭代次数；纵轴：NLL（性能代理）或 embedding dissimilarity（多样性）  
- Multi-agent FT：性能可继续升；**多样性保持**  
- Single-agent FT：准确率塌缩或不提升  
- 在 math 上训，对相邻域（如 GSM8K，课上称较旧数字不高）仍有迁移  
- Llama 对该技术更敏感（课上观察）

#### Takeaway

> 没有多样性的合成数据飞轮会自我耗尽。多 agent / 多专长是制造多样性的一条工程路径。

连接：L02 Monkeys / Archon；L06 STaR；L05 多路径搜索。

---

### 4.2 DeepSeekMath-V2：元验证打破「只看最终答案」

#### 问题

- 当前 RL 常假设：**最终答案 = ground truth ⇒ 奖励**  
- 足以冲饱和 AIME 等；但**证明 / 定理**需要严格逐步推导  
- LLM 生成的证明常数学无效；再让 LLM 验证，会把错证明判对 → **朴素 LLM-as-judge 不够**  
- 人类专家能看出「下一步并不从上文推出」的 gap  

#### 机制

在熟悉的 **generator ↔ verifier** 环上增加 **meta-verifier**：

1. 人类（早期）在**无参考解**条件下标出证明问题 → 训练 verifier 找 issue  
2. Verifier 给证明打分（课上提到 0 / 0.5 / 1 量表）  
3. **Meta-verifier** 审查 verifier 的分析：  
   - 指出的 issue 是否真实存在？  
   - 分数是否由这些 issue 合理推出？  
   - 减少「编造错误」的幻觉式验证  
4. 专家再标注「评估质量」  
5. 闭环：更好的 verifier → 改进 generator → 更难证明 → 再练 verifier  
6. 一旦学会找错，**可自动化标注**，减少对人依赖  

#### 结果（课上）

- 基于 DeepSeek-V3 类模型；IMO / CNML 等  
- 迭代优化下 proof score 持续爬升  
- **Best@32** 在 2024 IMO shortlist 上 proof score 约到 **42%** 量级（课上）  
- 洞见：高质证明得高验证分；generator 学会区分优劣；self-verification 可 hill-climb  

#### 可迁移配方（课上归纳）

若要用 LLM verifier：

1. 学会在**无参考解**下找问题  
2. 加 **meta-verification** 降幻觉 issue  
3. 给 generator 激励：通过深思最大化质量  

局限：仍偏「验证相对容易」的域。

连接：L03 Gen–Ver gap、ORM vs PRM；客座 Thang Luong 的 Gemini IMO / 形式化证明对照。

---

### 4.3 Absolute Zero：模型自己出题 + 做题（零外部题库）

#### 问题

- SFT 要人类推理轨迹；RLVR 要专家策展的 QA  
- 模型越强，越难找到「还能教它」的人类专家与题目 → **数据墙**  
- 思想极端化：单一模型既 **propose tasks** 又 **solve**，不依赖外部人类 prompt 分布  

#### 任务类型（代码域）

对程序 / 输入 / 输出三元组的推理模式：

| 类型 | 直觉 |
|------|------|
| **Deduction** | 给定程序+输入 → 环境执行得输出；学「往前跑」 |
| **Abduction** | 类似结构，另一方向的推断设定 |
| **Induction** | 采样已有程序，生成新输入 + 自然语言功能描述；环境校验 |

#### Proposer 奖励与课程

- 把任务交给 solver：  
  - 成功率 = 0 → proposer 奖励 0（太难 / 无效）  
  - 否则奖励 ≈ **1 − average success rate**（太简单则奖励低）  
- 目标：**中等难度**——有时对有时错 → 最大可学性  
- 模型变强后，proposer 应提出更难任务 → **自适应课程**  

#### 防黑客校验

代码域可执行验证：

- 程序完整性 / 语法  
- 安全检查  
- 多次运行输入输出一致（确定性）  

无效任务不进训练管线。

#### Buffer / 多样性

- 维护 seed triplet buffer（程序–输入–输出）  
- Proposer 条件于过去样本以促多样性  
- 记录成功 / 失败次数 → 演化课程  

#### 结果与涌现（课上）

- **无人类策展 prompt** 仍达 coding benchmark SOTA；超过用数万专家样本训练的模型  
- 复杂度指标随时间上升  
- 程序与答案多样性在环设置正确时上升  
- Proposer 随训练提出更难任务  
- 像轻微对抗的博弈，但共同进步  
- 惊人点：只在自提代码任务上爬坡，**数学基准也变强**  
- **更大模型收益更大**  

#### Azalia 补充（与 SWiRL 共鸣）

- 模型自造训练数据 → 不仅提升生成域，还有**跨工具 / 跨任务迁移**（SWiRL：多跳 QA+检索的合成数据，也能提升 Python / 数学等）  
- 大模型更会吸收 data flywheel  

连接：L05 SWiRL；L06 RLVR；L07 open-ended evolution。

---

### 4.4 自改进三条总结 + 开放问题（A 部分收束）

课上归纳要真正泛化推理，需要：

1. **推理链多样性**（如何系统性制造仍开放）  
2. **更好验证**（减少对人与窄域 verifier 的依赖）  
3. **数据 / 任务选择**不被人类策展卡住  

Azalia 提出的研究问题：

- 在**可自动验证**的多样任务上把飞轮推到多远，能否迁移到**不可自动验证**域并减少标注？  
- 不可验证 / 慢验证例子：芯片设计仿真（天级）、化学实验（进实验室）、科学发现；创意写作等主观域（易 reward hacking）  
- RL / test-time 需要近乎即时（或分钟–小时级）verifier；**天级 reward** 撑不起数千迭代  

慢验证的工程对策（课上）：

- 离线跑贵仿真，训 **surrogate reward model** 预测质量——但泛化受数据限制，不准会害 RL  
- KernelBench 类：编译执行可给对错与指标；但读 performance profile、改非热点代码更难 → 常需**问题分解**到模型当前能力切片（点名课上相关 project）

---

### 4.5 Intelligence per Watt：效率前线

#### 宏观背景（Azalia）

- 今天像 **mainframe 时代**：ChatGPT / Gemini 等大模型几乎都在云上跑  
- 需求爆炸：课上举例 Google Cloud 算力服务 ~**20 个月 1200×**；NVIDIA ~**10× YoY**；需约 **250 GW** 级数据中心供应想象空间  
- Token 处理量：某内部图从去年 2 月 **160T** 量级到今年 10 月 **1.3B**（录音口播数字；理解为「增长极快」的示意）——人类历史上最快的一类算力需求增长  

#### 工作负载观察

- ChatGPT 类大规模日志：约 **77%** 请求是 practical guidance / 信息查询 / 写作等  
- 这些**不需要最强前沿模型**；较小本地模型可解  
- 用户会问更难的问题，但**主体仍偏简单侧**

#### 本地硬件趋势

- 2012→今本地加速器 GPU 内存约 **126×** 提升  
- Mac 级设备可达 ~**100GB** 统一内存 → 量化后可塞很大模型  
- 与「多数查询可用小模型」趋势同向  

#### IPW 定义

\[
\text{Intelligence per Watt} \approx \frac{\text{average task accuracy}}{\text{average power draw}}
\]

研究范围（课上）：

- 「本地模型」工作定义：约 **≤20B active parameters**  
- >20 个本地模型（Qwen、GPT-OSS、Gemma3 等）  
- 企业加速器 vs 本地加速器  
- ~**100 万** ChatGPT 来源查询 + Natural Reasoning / MMLU-Pro / SuperGPQA 等  
- 指标：准确率、能耗、延迟、算力；数据将开源  

#### 主要发现（课上）

| 发现 | 数字 |
|------|------|
| 本地模型解决 chat 类查询比例提升 | 自 2023 约 **3.1×**；现可覆盖约 **88.7%** 所述查询 |
| 本地加速器 IPW vs 企业芯片 | Apple M4 Max 约比 B200 **低 1.5×** IPW（企业芯片更针对 LLM） |
| IPW 两年总提升 | 约 **5.3×** = 模型 **3.1×** × 硬件 **~1.7×** |

含义：越来越多流量**可以**在边侧解决；不是「云要消失」，而是**重新分配**。

#### 由此引出的开放方向（效率侧）

1. **混合 inference serving**：按复杂度在本地 / 云模型与加速器间平滑路由  
2. **面向本地的能效架构与 kernel**（相对云加速器研究不足）  
3. **Energy 作为最稀缺资源**；IPW 类指标应成主流优化目标（今天还太少人优化它）  

---

### 4.6 横跨全课的开放研究问题（课上列表）

1. **Test-time scaling 的第一性原理**  
   - 为何重复问 / 搜索会「冒出」正确答案？  
   - 成功轨迹 distill 回模型的最佳实践？  

2. **Continual learning**  
   - 人类边做边学；模型多为离线攒轨迹再异步微调——范式不匹配  
   - 如何把正负经验更自然地写回模型？  

3. **高吞吐低延迟的 test-time scaling 基础设施**  
   - 重复采样、多轮修订、tool calling ≠ 单轮 chatbot  
   - 需要专用系统优化（课上点名 lab 工作如 hydrogen / token SRS 等）  

4. Pretraining 本课触及少；重心在 **post-training ↔ test-time ↔ 合成数据飞轮 ↔ continual learning** 的连接带。

---

## 5. 课堂 Q&A（详尽）

### Q1. Continual learning 更该做在长期记忆系统，还是改模型权重？

- **记忆 / 无限上下文理想**：若有完美无限上下文，把一切正负经验塞进上下文也是一种 continual learning——但现实中即便百万级上下文，对 ICL 数据的推理能力会衰减  
- **Cartridges（L14）**：不改权重，把语料压进 KV / activations，延长有效上下文  
- Akanksha：若只是知识库更新 → **更新 DB** 更简单；若要教**新域推理技能** → 往往必须 **更新权重**  
- 机器人跨本体泛化（点名 Danny Driess 客座）：再多 memory 也不会自动出现技能迁移  

### Q2. Absolute Zero 的环境 ≈ post-training 数据；有没有 agent 自造环境的论文？

- 窄智能时代：游戏仿真即环境  
- 今天环境重要是因为它们是**真实任务的代理**  
- 关键是：任务集合是否有代表性、是否可模拟反馈；不一定需要「agent 写环境」这篇独立论文  
- 游戏状态空间有限 → 易用代码仿真；真实世界代理更难  

### Q3. 不可验证域怎么做？（芯片 / bio）

- Surrogate reward model  
- 分解到可验证子问题（KernelBench 式执行反馈 + 剖析）  
- 接受：慢实验域无法直接套「千步 RL」模板  

---

## 6. 关键数字速查

| 数字 | 含义 |
|------|------|
| Best@32 ~42% proof score | DeepSeekMath-V2 课上 IMO shortlist 数字 |
| 3.1× / 88.7% | 本地模型准确率进步 / 查询覆盖 |
| 5.3× IPW | 两年智能每瓦提升 |
| 1.5× | B200 相对 M4 Max 的 IPW 优势量级 |
| ≤20B active | 课上「本地模型」工作定义 |
| ~77% | 相对简单的实用查询占比 |
| 1200× / 10× / 250GW | 云需求与能源背景（课上量级） |

---

## 7. 本讲相关论文 / 思想卡片

| 思想 | 卡片 |
|------|------|
| Multi-agent finetuning（多样推理链） | [papers/multi-agent-finetuning.md](../papers/multi-agent-finetuning.md) |
| DeepSeekMath-V2 元验证 | [papers/deepseekmath-v2.md](../papers/deepseekmath-v2.md) |
| Absolute Zero Reasoner | [papers/absolute-zero.md](../papers/absolute-zero.md) |
| Intelligence per Watt（课上研究） | [papers/intelligence-per-watt.md](../papers/intelligence-per-watt.md) |

---

## 8. 工程可落地 checklist

- [ ] 合成数据管道是否监控 **diversity**（嵌入离散度 / 唯一推理策略）？塌了就上多 agent / 多模型  
- [ ] Verifier 是否只看最终答案？关键域加 **process / meta-verification**  
- [ ] 任务是否全靠人手出题？可验证域试点 **proposer–solver** 课程  
- [ ] 慢仿真域：先建 surrogate，并持续测其与真实验相关  
- [ ] Serving：对简单查询路由本地 / 小模型；难查询上云  
- [ ] 报表增加 **energy / watt / TTFT / 美元**，不只准确率  
- [ ] Test-time 路径单独做 infra（前缀缓存、并行采样、工具并发）  
- [ ] Continual：明确是「更新记忆」还是「更新技能权重」  

---

## 9. 与课程其他讲的连接

| 讲次 | 连接 |
|------|------|
| L02–L03 | test-time、verification bottleneck |
| L05–L06 | SWiRL 迁移；STaR / GRPO / RLVR |
| L07 | open-ended / 自进化任务分布 |
| L13 | KernelBench 作为「可执行但不平凡」验证 |
| L14 | Cartridges / MemGPT 作为非权重 continual 路径 |
| L17 | 可靠性长尾、经济任务线性 vs 时长指数 |
| 客座 | Melvin post-training；Denny / Thang 推理；Misha 自主 agent；Danny 机器人；Junchen 记忆 serving |

---

## 10. 若你只记 5 条

1. 自改进下一跳：**多样轨迹 × 元验证 × 自提任务**。  
2. 单模型合成数据会**同质化耗尽**；多 agent FT 是续命手段之一。  
3. Absolute Zero：在可执行环境里 **零外部题** 也能爬坡，并迁移到数学。  
4. 多数用户查询可用**本地小模型**；优化 **Intelligence per Watt**。  
5. Continual learning 未解：记忆系统 ≠ 技能写入权重；按问题选机制。

---

## 11. 结课寄语（录音原意）

领域极快，「现在教的明年变历史」，但**基本技法**（verifier 环、test-time、工具、记忆、评测）会反复复用。讲师感谢同学陪伴共同备课——很多材料备课时自己也在学。希望这是研究 / 工作的起点而非终点。
