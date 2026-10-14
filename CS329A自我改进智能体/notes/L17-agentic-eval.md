# L17 — Agentic Evaluations & Long-Horizon Tasks

> **来源**：有录音 · `raw-transcripts/8.evalandlonghorizontask.md`  
> **日期**：Mon Nov 17（CS329A Autumn 2025）  
> **主讲**：Aakanksha Chowdhery  
> **必读**：METR 长任务时间地平线 · GDPVal · DeepScholar-Bench

---

## 1. 一句话总览

传统单轮 QA / chatbot 基准正在饱和；agentic 评测要同时回答三件事：**任务能做多长**（METR time horizon）、**对真实有经济价值的工作是否够好**（GDPVal win rate）、以及**在需要检索与引用知识库时质量是否可信**（DeepScholar-Bench）。三者趋势并不一致——时长指数翻倍 ≠ 经济任务胜率指数增长 ≠ 研究综合质量已够用。

---

## 2. 核心概念与定义

### 2.1 为什么需要新一代评测

课堂开场点名：几乎没人在项目里认真跑 agentic eval，但作业与项目其实已经在用多跳 QA、私有 Wiki QA、coding agent 等。讲师强调：评测的难点不在「能不能出分」，而在**相对旧范式，agentic 评测难在哪里**。

历史对照：

| 年代 | 「进步」长什么样 |
|------|------------------|
| ~4–5 年前 | 模型能生成任何像样解释都算大事 |
| 今天 | 要预报对经济与 safety 的影响 |
| 传统基准 | Chatbot / 单轮 QA / 上下文内已有答案 → **快速饱和** |

评测必须同时覆盖：

1. **Capability**：多复杂 / 多长的任务能完成？
2. **Economic impact**：能否做人类专业人士正在做的真实工作？

### 2.2 两条互补但不同的度量

| 度量 | 来源 | 问什么 | 课堂强调的趋势形状 |
|------|------|--------|-------------------|
| **Time horizon** | METR | 人类专家完成该任务要多久？模型在某可靠性下能覆盖多长？ | ~每 7 个月翻倍（50% 成功） |
| **Win rate vs experts** | GDPVal | 模型输出相对十年经验行业专家的胜率 | 近两年更接近**线性** |

讲师原话要点：两者「同样有效、洞察相似，但测到的趋势很不一样」。看到指数图时不要直接外推「下一阶段就能做几天」。

### 2.3 可靠性 vs 能力

- **Capability axis**：能完成多长的任务（哪怕只有 50% 成功）
- **Reliability axis**：在更高成功阈值（如 80%）下，时间地平线会**大幅缩短**

课堂类比：50% 成功 = 「把任务交给实习生，一半时候能做完」——有不确定性；部署真实工作更关心 80%+。

---

## 3. 方法 / 机制

### 3.1 METR：用人类时间校准「任务地平线」

#### 3.1.1 任务套件结构

约 **170** 个任务，覆盖从秒级到约 8 小时：

| 套件 | 规模（课上口播） | 人类时间尺度 | 任务性质 |
|------|------------------|--------------|----------|
| **SWAA** | ~66 | 1–30 秒 | 原子动作（如打开文件、找 shell 脚本） |
| **HCAST** | ~97 | 1 分钟–30 小时 | 多样软件 / 研究工程任务 |
| **RE-Bench** | ~7 | 可到 ~8 小时 | 完整 ML 研究风格（与课上 AI Scientist 同类） |

示例任务（课上列举）：

- 找一个 shell 脚本（~3 秒）
- Wikipedia research（接近「写 lit review」类）
- 在仿真输入文件中检测并修 bug（人类 ~10 分钟）
- JSON 数据 munging（格式转换）
- 自定义 CUDA kernel / backtesting 工具，瞄准性能提升（人类可达 ~8 小时）

#### 3.1.2 标定流程（课上步骤）

1. **收集并 vet** 任务进入各 suite  
2. **人类基线**：约 **5 年经验**的熟练专业人士记录**成功尝试**的完成时间；用**几何平均**估计任务难度评分  
3. **Agent 运行**：多轮 attempt，测是否可靠完成  
4. **拟合曲线**：success rate vs 人类时间估计 → 推出模型在 50% / 80% 等阈值下的 **time horizon**  
5. **对照发布日期**：画出「模型发布日期 → 时间地平线」趋势

#### 3.1.3 人类基线的偏差（课堂 Q&A）

学生问：人类完成时间差异很大，极端是一个主评员 vs 100 次分布？

回答要点：

- 请大量专家很贵 → 通常**少数专家** + 看 **inter-rater / inter-human agreement**  
- 数字差很远就再采；大致一致就推进  
- **经验者常低估难度**（已习惯「什么叫成功」），而模型眼中的难度可能不同  
- 对内部 codebase：contractor 可比 maintainer **慢 5–18×**；模型表现更接近 **低上下文 contractor**，而非「有完整仓库上下文的专家」

#### 3.1.4 成功曲线的形状（课上描述）

- 横轴：人类完成时间；纵轴：模型成功率 0–1  
- **SWAA（绿 / 短任务）**：成功率偏高  
- **HCAST**：从 ~0.8 到 0 **散开**  
- **RE-Bench**：偶有成功，整体偏低  
- 结论：**任务越长，模型越难保持在轨完成**——这正是 agentic eval 变难的核心原因之一

#### 3.1.5 历史趋势数字（课上口播，务必记住）

| 模型 / 时期 | 约 50% 成功时间地平线 |
|-------------|----------------------|
| GPT-2（2019） | ~**2 秒** |
| GPT-4（~2023） | ~**几分钟 / ~8 分钟** |
| Claude 3.7 Sonnet、o1 等（课上 2025 图） | ~**几小时**；Claude 3.7 提到 **~59 分钟（50%）** |

- 粗趋势：**约每 7 个月翻倍**  
- **80% 可靠性**：同模型地平线大幅缩短——课上对比 Claude 3.7：**~15 分钟（80%）vs ~59 分钟（50%）**；或「高可靠仍只有几分钟到十几分钟」  
- 预测外推（课上）：若趋势延续，**2028–2031** 可能接近「一个月」量级的任务（仍带 50% 可靠性 caveat）

#### 3.1.6 驱动进步的能力（课上清单 + 学生补充）

讲师列出：

1. 更好的 **logical / multi-step reasoning**（前几讲）  
2. 更强的 **code generation**  
3. **Tool use**（ReAct 等）  
4. 更少重复、更可靠  
5. **从错误中恢复** + 对最终目标的意识  
6. **状态 / 记忆**：目标是什么、进展到哪一步  

学生补充（课堂互动，讲师复述）：

- **Context engineering**：更好组织多步上下文；Claude Code 的 compaction window  
- **用户日志反馈**回灌到工作流 / 训练（类 Cursor tab 在线反馈）  
- **显式规划 + 再规划**：复杂问题先 plan，执行中基于新数据 replan，减少盲目试错  
- **用户反馈纠错**：像作业打回重做  
- **Memory / codebase 先验**：atomic memory 帮助理解环境；与 **L14 Memory** 直接相连

#### 3.1.7 失败模式（GPT-4 vs o1，课上分类）

| 失败类型 | 说明 |
|----------|------|
| Poor planning | 不会把任务拆成步骤 |
| Poor tool choice | 工具选错 |
| Incorrect mental math / reasoning | 推理链算错 |
| Premature abandonment | 重复行为后过早放弃，不清楚「成功」定义 |
| Repetitive loops | 失败后仍以高概率重复同一动作（GPT-4 很常见；o1 有所降低） |
| Other | 仍有其他类别 |

工程含义：**failure mode analysis** 是评测标配——作业里做 self-iterative refinement 时也要先定位失败类别。

#### 3.1.8 METR 的局限（课上明确列出）

1. 对「messy / 无唯一正确解」任务表现更差；但若有一定 messiness，趋势在不同类别仍可类似  
2. **SWE-bench** 趋势类似，但标注者常**低估**人类时间；且模型多半「见过」许多公开仓库 → 翻倍时间可能**偏短**（相对未见仓库）  
3. 内部 PR / 内部代码库：模型 ≈ 低上下文 contractor，**不是**领域专家  
4. 自动打分、**单 agent**、**不惩罚**错误成本、**几乎无资源约束** → 与真实世界仍有差距

#### 3.1.9 50% 成功如何计算？（课末澄清）

- 每个任务：多次 agent run → 该任务的成功率 = 成功 attempt 比例  
- 对某一时间地平线：用这些点拟合；「50% horizon」≈ 在该人类时间尺度上，模型约一半时间能交回好答案（「抛硬币」）  
- 不是「每个任务都必须至少 50%」那种更严的 per-task 保证

---

### 3.2 GDPVal：真实经济任务上的专家胜率

#### 3.2.1 问题重述

不是「AI 能不能做这个」，而是：

> 把任务交给模型而不是人类，**输出够不够好**？胜率直接对标**行业专家**。

任务来自 **>10 年经验**专业人士的真实工作。

#### 3.2.2 覆盖范围（课上数字）

- 瞄准 **GDP top ~5%** 相关的高价值工作  
- 约 **9 个 sector**、**44 occupations**、约 **1,320 tasks**  
- 公开 gold set 约 **200–220**（Hugging Face）  
- 以 **数字 / 计算机任务**为主；O*NET 中约 **60%** computer-based digital 任务进入  
- 平均完成时间可达 **~7 小时**，有些要**数周**  
- 模态：文本 + CAD / video / audio / spreadsheet / presentation  
- Gold 子集任务平均价值约 **$400/task**（也有更高价值；对比 Upwork 等）  
- **~70%** 任务需与 **reference file** 交互  
- **~89%** 经专家 vet 为 well-specified（避免因「题面烂」而难）

职业举例（课上）：房地产、政府、制造、专业科技服务、医疗、金融、零售批发、信息（含影视剪辑）等。

示例任务：

- 制造工程师：为装配线设计 cable reel stand 的 3D 模型  
- 金融分析师：做竞争格局研究  
- 注册护士：根据组织图像写会诊报告  
- 影视剪辑：按脚本做 intro reel  
- 客服：给不满客户起草邮件  
- Concierge：为一家四口做行程  
- 审计：多份采购订单找定价不一致  
- 地产经纪人：新盘销售手册  
- Recreation：优化 vendor fair 布局  

#### 3.2.3 成功指标与主观性

- 成功 = **pairwise win rate**（模型 vs 专家），含 wins / ties  
- 学生点出：很多任务极主观 → 讲师承认：真实工作本来就**上下文 + 主观**；所以用胜率而非「唯一正确答案」  
- 伦理 / 合规「该不该用」与「模型能力够不够」要**分开讨论**

#### 3.2.4 趋势数字（课上口播）

横轴：模型（GPT-4o → GPT-5 high → Claude Opus 4.1 等）；纵轴：相对行业专家胜率。

粗量级（课上）：

- GPT-4o 时代约 **12.4%** 量级起步  
- 中间升到 **~25 → 30s**  
- Claude Opus 4.1 约 **47.6%**（wins+ties 口径课上对比）  

与 METR 对比：**GDPVal 更接近线性**，不是每 7 个月翻倍那种指数叙事。

模型风格差异（课上观察）：

- 某些模型更强：美学、文档排版、PDF / spreadsheet / presentation  
- GPT-5 更强：instruction following、正确计算、纯文本任务  

#### 3.2.5 失败模式（GDPVal）

常见：

1. **Instruction following** 错误（多模型蓝条最高的一类）  
2. **承诺读 reference 却没读**，用幻觉覆盖  
3. **Formatting** 错误  
4. GPT-5 在该基准上 instruction-following 错误相对更少  

可接受性分层（课上）：

- ~**50%** 任务：可接受但偏差（acceptable but subpar）  
- ~**20%**：模型真的更好  
- ~**29%**：差 / catastrophic  
- 人类 grader 之间也有分歧 → 分析有噪声  

#### 3.2.6 自改进环与成本（与本课主题直接挂钩）

课上强调：parallel / sequential 多试 + 让模型修自己，常能明显提升。

- GPT-5：**~1.6× cost**、**~1.4× speed** 相对「无助专家」的某种改进口径（课上表格）  
- 成功时成本仍常 **< 人类专家薪资的 10%**  
- 与 L02 test-time scaling / L04 feedback 同一逻辑

#### 3.2.7 按行业 / 时长 / 模态的变化

- **近平价或表现强**的例子（课上点名）：counter & rental clerks、real estate brokers、shipping/receiving/inventory clerks、buyers & purchasing agents、computer/IT managers、software developers；政府行政 / compliance；医疗健康服务管理；个人理财顾问、客服；零售一线主管；编辑；调查 / 新闻分析；批发制造销售代表等  
- **按时长**：短任务（数小时内）更强，长任务明显下降（与 METR 一致）  
- **按模态**：有的模型多模态强，有的文本强  

软件开发讨论：学生质疑「14 年经验」口径；讲师：软件侧更可能偏 **repo 维护**而非「十年领域直觉」；工业 / 机械工程师则十年经验更关键。**熟知 codebase** 本身就是关键上下文。

#### 3.2.8 欠指定 prompt 与「人类架构问题」

- 去掉上下文后：win rate 掉几个点，但更关键的是模型**不知道该干什么**  
- 真实工作大量是：**优先做什么、问题是什么、资源放哪**——这些往往在专家脑子里，未写入 prompt  
- 近端含义：**AI assistance + 人类 oversight**（人类 architect 问题，模型执行）已经有价值；不同职业影响不同；要**按任务选模型**

#### 3.2.9 美元价值问题（课堂 Q&A）

学生想用「AI 现在能做多少美元的工作」量化。讲师：关键的不是单点美元数字，而是**哪类任务**模型天然擅长（deep research、代码、多源信息综合），以及这些任务落在哪些职业——GDPVal 提供的是可量化 taxonomy。

「模型胜过人类」时务必回去看 prompt：往往是「人类把脑子里上下文全写进 prompt 之后」模型才能做——**不是**模型已能自己获取那些上下文。

---

### 3.3 DeepScholar-Bench：生成式研究综合（live）

#### 3.3.1 动机

课堂早有同学想「让 AI 写 lit review」——本基准正式测：

> AI 能否做 **deep research synthesis**：读一批文献、检索相关信息、综合成连贯报告，且引用可核验？

产业原型对照：OpenAI / Gemini / Perplexity Deep Research、Stanford STORM、OpenScholar；作业 HW3 也做过 deep research agent。

#### 3.3.2 数据集设计

- 任务：为学术论文生成 **Related Work**  
- 材料：近期 arXiv、PhD 难度、跨 **~22 domains**  
- **每月用新论文重跑** → live，抗污染  
- 只用**主模型训练截止之后**的论文  

#### 3.3.3 三轴评测（均有人类验证，agreement ~70–80%）

1. **Knowledge synthesis**  
   - 组织是否连贯  
   - 是否抓住 **key facts / nuggets**  
2. **Retrieval quality**  
   - 相关性  
   - Citation counts / 权威源  
   - Reference coverage / 是否找到重要论文（document importance）  
3. **Verifiability**  
   - 引用是否支持 claims（precision）  
   - claims 是否被引用覆盖（coverage）  

#### 3.3.4 关键结果（课上）

- **没有系统整体超过 ~19%**（相对综合指标）→ **巨大 headroom**，适合下学期作业 / 期末项目  
- Knowledge synthesis：OpenAI Deep Research 写得很连贯，但**几乎所有系统都漏关键事实**（「好英文 ≠ 全事实」）  
- Retrieval：普遍难找全面且重要的源；document importance **< ~12.5%**  
- Verifiability：DeepScholar base 精度可到 **~90%**；OpenAI Deep Research 可验证性更低，尽管文笔更好  

#### 3.3.5 失败模式（学习重点）

1. 找不到全面源；找到相关文档也常漏**奠基论文**（专家靠多年 bookmark / 领域记忆）  
2. Agent 难判断「重要性」——只会判断「是否与 query 相关」  
3. 即使检索不错，也难高效抽出关键信息  
4. **Oracle 源**（直接给该读的论文）时，key fact coverage 也大约只有 **~50%**；无 oracle 更低  
5. **没有系统同时在 synthesis 质量与 verifiability 上都很强**  

#### 3.3.6 为何单独讲这个「窄任务」？

学生问：比 METR / GDPVal 窄，为何单独一讲？

讲师：本讲主题是 agentic eval 关心什么——长度、经济价值之外，真实任务还要求**会用地用知识库**。DeepScholar 是该类的代表任务。

另外：不只数「命中了专家列表里多少论文」，还要看 related work 里「我的工作与别人有何不同」等 **claims + 引用支撑**——检索与写作事实覆盖是两件事。

---

### 3.4 三基准交叉总结（课上收束）

| 维度 | METR | GDPVal | DeepScholar |
|------|------|--------|-------------|
| 核心问题 | 能做多长？ | 相对专家够不够好？ | 综合 + 引用是否可信？ |
| 研究综合任务位置 | 人类 30 分钟–8 小时；能力区 ~50 分钟量级但质量可能差 | 金融研究等有明确经济价值 | 直接测合成质量缺口 |
| 主要缺口 | 长程可靠性、messy 任务 | 上下文、instruction、reference 使用 | 全面检索、关键事实、可验证性 |
| 方法学特征 | 自动打分、单 agent、少惩罚 | 任务过度 well-specified、偏 one-shot、依赖已有 tacit knowledge | 测「从知识库取上下文」的能力缺口 |

**Headroom 共识（课上）**：

模型已较强：隔离、well-specified、软件工程 / ML 研究等「我们自己在自动化自己工作」的领域；强模型能产出**组织良好**的文本（即使部分错误）。

模型仍弱：缺上下文时不知做什么；歧义 prompt；对抗环境；**95% 可靠性**；软件与知识工作之外的泛化；从知识库找全面高质量源 + 抽关键事实 + 高精高召回验证。

最终需要**三套指标并存**：duration-based、economic-value-based、synthesis-quality-based，并持续对人校验。

---

## 4. 关键数字与实验结论（速查卡）

| 数字 | 含义 |
|------|------|
| ~170 tasks | METR 总任务量 |
| 97 / 66 / 7 | HCAST / SWAA / RE-Bench |
| 每 ~7 个月 ×2 | 50% time horizon 翻倍 |
| ~59 min vs ~15 min | Claude 3.7：50% vs 80% 地平线（课上） |
| 5–18× | contractor vs maintainer 慢多少 |
| 9 sectors / 44 occupations / ~1320 tasks | GDPVal 规模 |
| ~47.6% | Claude Opus 4.1 专家胜率量级（课上） |
| ~50% / 20% / 29% | 可接受偏差 / 更好 / catastrophic |
| <19% | DeepScholar 现有系统综合上限 |
| <12.5% | document importance |
| ~50% | 给完美源时 key fact coverage |
| 2028–2031 | 外推「月级」任务（50% caveat） |

---

## 5. 课堂 Q&A 与易错点

1. **不要把 METR 指数图直接当成 AGI 时间表**——GDPVal 线性胜率与 DeepScholar 质量缺口在「打脸」过度外推。  
2. **50% ≠ 可部署**——部署看 80%+；差距很大。  
3. **Win rate 高 ≠ 模型会自己找上下文**——常是 prompt 已注入专家上下文。  
4. **好文笔 ≠ 好研究综合**——DeepScholar：连贯写作与 key facts / 可验证性脱节。  
5. **SWE-bench 乐观偏差**：公开仓库污染 + 人类时间低估。  
6. **长尾瓶颈**：数据 + 基础能力；可验证域（法务 / 金融研究）进步快；极窄域与机器人更吃数据收集。  
7. **职业不会消失的论断（讲师）**：CS 需要 fundamentals；AI 需要你给上下文——「architect 问题」仍是人的工作。  
8. **AI Scientist / AGI 定义（课末）**：真正「AI 自己建下一代模型」还没到；近端是强 **co-scientist / brainstorming partner**；可靠性长尾是 last mile。

---

## 6. 本讲论文速查

| 论文 | 卡片 |
|------|------|
| Measuring AI Ability to Complete Long Tasks (METR) | [papers/metr-long-tasks.md](../papers/metr-long-tasks.md) |
| GDPVal | [papers/gdpval.md](../papers/gdpval.md) |
| DeepScholar-Bench | [papers/deepscholar-bench.md](../papers/deepscholar-bench.md) |

---

## 7. 工程可落地 checklist

- [ ] 为你的 agent 定义 **人类时间标定**（哪怕粗估）：分钟 / 小时 / 天  
- [ ] 同时报 **50% 与 80%** 成功下的能力，不要只报 best-case  
- [ ] 做 **failure taxonomy**：planning / tools / reasoning / loops / abandonment / instruction-following / reference-ignore  
- [ ] 若任务依赖文件：显式测「是否真读了 reference」  
- [ ] 研究 / RAG 类：分项测 retrieval、key-fact coverage、citation support——不要只看 ROUGE / 「读起来顺」  
- [ ] Live / 抗污染：用训练截止后的新数据重跑  
- [ ] 多试 + 自修环（test-time）作为默认基线，并报成本  
- [ ] 区分 **well-specified one-shot** 与 **真实迭代协作** 评测  
- [ ] 内部代码任务：用 maintainer 时间与 contractor 时间双基线  
- [ ] 选模型时按任务类型（多模态排版 vs 严格指令 / 计算）分流  

---

## 8. 与课程其他讲的连接

| 讲次 | 连接点 |
|------|--------|
| L02 Test-time compute | 多采样 / 自修提升 GDPVal；coverage vs selection |
| L03 Verification | DeepScholar 的 verifiability；长任务需要过程监督 |
| L04 Tools / Feedback | Tool choice 失败；execution feedback |
| L05 Planning | Poor planning / replan；复杂任务先 plan |
| L07 / L08 AI Scientist & Deep Research | RE-Bench；DeepScholar；HW3 |
| L13 SWE Agents | SWE-bench 乐观偏差；CUDA / kernel 长任务 |
| L14 Memory | 长程目标状态、codebase 记忆、compaction |
| L20 Future | 可靠性长尾、continual learning、可验证域外推 |

---

## 9. 若你只记 5 条

1. Agentic eval = **时长 × 经济价值 × 知识综合质量**，缺一不可。  
2. METR：时间地平线约 **7 个月翻倍**，但 **80% 可靠地平线短得多**。  
3. GDPVal：相对专家胜率近 **线性**；强项分职业，弱在 instruction / reference / 缺上下文。  
4. DeepScholar：现系统 **<19%**；好文笔掩盖漏事实与弱引用。  
5. 近端现实：**人类 architect + AI execute**；模型更像低上下文 contractor，不是资深 maintainer。

---

## 10. 课堂互动任务备忘（自测）

讲师让学生举例「几分钟 vs 几小时想交给模型」：

- 写 literature review（数小时 → HCAST / RE-Bench）  
- 核对 BibTeX（分钟级 → HCAST）  
- Refactor codebase（数小时到数天 + 正确性检查 → 接近 RE-Bench）  

用这些例子校准你对自己项目的「时间地平线」估计。
