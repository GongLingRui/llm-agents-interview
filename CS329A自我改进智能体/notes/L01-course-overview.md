# L01 — Course Overview：从 LLM Scaling 到 Self-Improving Agents

> 一句话总览：本讲把过去五年 LLM 能力跃迁拆成 **pretrain → finetune（instruction/RLHF）→ inference（test-time scaling）** 三阶段，并说明为何「固定参数、在推理时花更多算力」与「把推理时产生的可验证合成数据再喂回训练」共同构成 self-improving AI agents 的核心叙事。

---

## 1. 本讲目标与在课程中的位置

### 1.1 课程身份

| 项 | 内容 |
|----|------|
| 课程 | Stanford **CS329A** Self-Improving AI Agents（Autumn 2025） |
| 官网 | https://cs329a.stanford.edu/ |
| 授课 | **Aakanksha Chowdhery**（Adjunct；Reflection AI）、**Azalia Mirhoseini**（CS Assistant Professor） |
| 背景 | 两人相识于 Google Brain / DeepMind；Azalia 还做过 Claude（Anthropic）与 Gemini |
| 定位 | 第二次开课；面向硕士为主的混合人群（也有 PhD / undergrad） |

### 1.2 本讲要回答的问题

1. **为什么模型越来越大就越好？**（pretraining scaling laws）
2. **为什么 ChatGPT 比 GPT-3「突然好用」？**（instruction tuning + RLHF）
3. **为什么 2024 起「推理时算力」也成了新 frontier？**（Large Language Monkeys / o1）
4. **从 chatbot / reasoning model 到 agent，差在哪里？**（goal → plan → act → feedback → stop）
5. **本课要覆盖什么、怎么评分？**（三份 HW + 研究项目）

### 1.3 在课程地图中的位置

```text
L01 Overview  ──► 建立三阶段世界观 + Agent 定义
L02 Test-time Compute ──► 深挖 repeated sampling / Archon / compute-optimal
L03 Robust Verification ──► 专门打 verifier / gen-ver gap
L04 Tools & Feedback ──► ReAct / Constitutional AI / RLEF
L05 Multi-step Planning
L06 Train-time RL
L07 Open-ended Evolution
L08 Search & Deep Research
…（SE agents / Memory / Eval / Future）
```

本讲是**总纲**：后续几乎所有讲座都在回答「如何在 pretrain / finetune / inference 某一环上让系统自我改进」。

### 1.4 本讲结构（课上实际顺序）

1. LLM scaling trends（参数 / 数据 / 算力）
2. Emergent abilities：few-shot、chain-of-thought
3. ChatGPT 跃迁：instruction tuning + RLHF
4. 新 frontier：inference / test-time scaling（Monkeys 预告）
5. Reasoning models（o1 / DeepSeek）如何把 test-time 与 train-time 合在一起
6. LLM → Agents：目标、工具、工作流
7. 课程 logistics（成绩、项目、late policy）

---

## 2. 核心概念与定义（含课上强调细节、数字、图意）

### 2.1 Pretraining Scaling Laws（三轴）

课上展示的经典图：横轴是某种「资源」，纵轴是 **test loss**（越低越好）。

| 缩放轴 | 课上说法 | 直觉 |
|--------|----------|------|
| **Compute** | 投入更多训练算力 → test loss 下降 | 更长训练 / 更大集群 |
| **Dataset size** | 数据量增大 → test loss 继续下降 | 更多 tokens |
| **Parameter count** | 层数 / 参数量增大 → loss 改善 | 「Large」字面意思 |

**关键历史数字（课上口述）：**

| 模型 | 约略参数量 |
|------|------------|
| BERT | ~340M |
| GPT-2 | ~1.5B |
| GPT-3 | **175B** |
| PaLM | **540B** |
| GPT-4（估计） | **trillions** |

时间线：约 2018–2024 模型尺寸近乎指数增长；到「去年」开始出现某种 **saturation** 迹象——仅靠更大 pretrain 的边际收益变难，于是更多人把注意力转到 post-training 与 inference。

**为什么关心更大模型？课上给了三点：**

1. 在自然语言、推理等 benchmark 上**持续变好**
2. 获得 **few-shot / zero-shot learning**：不必为每个域单独 finetune，给几个例子就能跟模板走
3. **Emergent behavior**：某些能力（尤其 reasoning）只在足够大的模型上突然出现

### 2.2 Zero-shot vs Few-shot

- **Prompt**：给任务描述（如 “Translate English to French”）+ 待翻译词（cheese）
- **Zero-shot**：只有任务描述，模型从未专门为该任务训练，却能答
- **Few-shot**：任务描述 + 若干示例翻译，再让模型翻译 cheese

课上强调：这是 GPT-3 / PaLM 时代打开「易原型化」的关键能力。

### 2.3 Chain-of-Thought（CoT）作为涌现能力

**普通 one-shot：** 给「题 + 答案」，再给新题——模型可能用上也可能用不上。

**CoT one-shot：** 不只给答案，还给**推理过程**。课上经典例子：

> Roger has 5 tennis balls. Buys 2 cans, each with 3 balls. How many now?  
> 走法：5；两罐 ×3 = 6；5+6=**11**。

要点：

- 如今 **1B** 模型也能解这个玩具题，**不需要** few-shot / CoT
- 但 **CoT 性质本身**仍是 reasoning / thinking models 过去一年进步的核心
- 课上对比 LaMDA / GPT / PaLM：小模型（如 LaMDA ~8B、某 GPT ~7B）**几乎吃不到 CoT**；更大时才能利用上下文中的推理过程提升数学等任务

其他「突然出现」的能力例子：mod arithmetic、word unscrambling——在某个尺寸阈值后突然变好。

### 2.4 ChatGPT 跃迁（不只是更大）

- 上线：**2022 年 11 月**
- 到 **100 万用户约 5 天**（课上强调：比许多著名软件快得多）
- 在「更大」之外，两个关键创新：
  1. **Instruction tuning**
  2. **RLHF**（Reinforcement Learning from Human Feedback）

### 2.5 三阶段流水线（本课最重要的骨架）

```text
┌─────────────┐    ┌──────────────────────────────┐    ┌─────────────────┐
│ Pretraining │ →  │ Fine-tuning                  │ →  │ Inference /     │
│ next-token  │    │ HQ data → Instruction → RLHF │    │ Test-time scale │
│ on web-scale│    │                              │    │ (fixed params)  │
└─────────────┘    └──────────────────────────────┘    └─────────────────┘
```

#### A. Pretraining

- 「最简单」的一步：在海量文本上做 **next-token prediction**
- 模型学到世界的统计结构，但**没有**「对错 / 偏好 / 如何听话」的明确感

#### B. Fine-tuning（课上拆成多步）

1. **高质量数据上的 SFT（仍是 next-token）**  
   - 书、创意文、公司花 **数百万～数亿美元** 买的高质量语料  
   - 比 raw web 精得多

2. **Instruction tuning**  
   - 数据形态：instruction + QA 对  
   - 来源：人工、模板、**合成数据** 混合  
   - 也可含 CoT fine-tuning：展示「如何一步步走到答案」  
   - **数据质量与覆盖面**强烈影响此阶段模型质量

3. **RLHF**  
   - 不再只是 supervised prompt→label  
   - 人类（有时是专家）对模型多条回答做偏好标注 → 训练 **reward model**  
   - 再用 reward model 引导 LLM 参数，使生成更符合偏好  
   - 可加权的 reward 维度例子：correctness / helpfulness / specificity / **harmlessness**

课上结论：pretrain + 高质量 finetune + instruction + RLHF，是 ChatGPT 相对 GPT-3 等「好用」的核心配方。

#### C. Inference / Test-time scaling（新 frontier）

约一年半前起：即使**不改参数**，在推理时用更多算力也能显著变强。这打开一整片研究方向，也是 L02 主题。

### 2.6 Infinite Monkey Theorem → Large Language Monkeys（本讲预告）

- **定理直觉**（非正式）：一直打字的猴子，足够久后会打出莎士比亚全集
- 项目类比：LLM = monkey；对同一题 **反复采样**；用 **verifier** 挑出正确样本作为最终输出
- Verifier 例子：代码用 **unit tests**
- 控制多样性：用 **temperature**（课上后续答问：太高如 >~1.2 会变成 gibberish）

**课上关键实验图意（L01 预告版）：**

- 横轴：每题样本数从 **1 → 10,000**
- 纵轴：**Coverage** = 至少一个样本做对的题目比例
- 红虚线：GPT-4o 的单次表现
- 蓝/绿：更弱的开源模型（如 ~3B / ~7B）单次更差，但**加大采样后可超过 GPT-4o 单次**
- 解读：「模型其实已经会更多；你只问一次时没问出来」
- 极端：有的难题在 10,000 个解里可能只有 **3–4 个** 正确 → 必须靠大规模采样 + 好 verifier

### 2.7 Coverage / Pass@k vs Pass@1（本课贯穿术语）

| 术语 | 含义 | 典型场景 |
|------|------|----------|
| **Coverage / Pass@k** | k 次采样中至少一次正确的比例 | Monkeys 式 repeated sampling（有完美 verifier 时） |
| **Pass@1** | 最终只交一个答案时的正确率 | 产品形态；o1 展示的 test-time scaling |

课上对 reasoning models 的关键澄清：

- Repeated sampling 抬的是 **coverage / pass@k**
- Reasoning / RL 训练抬的是模型「自己知道哪条链对」→ **pass@1** 上升

### 2.8 Reasoning Models（o1 / DeepSeek 等）

- OpenAI **o1**（约去年 9 月）：在难题集（课上称 hard AIME 类）上，**test-time compute（log 轴）↑ → pass@1 ↑**，呈 log-linear
- **DeepSeek**（约 2024 年 12 月圣诞前后）：把 **finetune 与 test-time scaling** 合在一起——用可验证域（数学有标准答案、代码有测试）在 test-time 造大量合成数据，再拿去 finetune / RL → thinking models

**Thinking 过程的典型子技能（课上列举）：**

1. Problem analysis  
2. Task decomposition  
3. Self-evolution / 用反馈优化（跑测试、用计算器、自我评判）  
4. Self-correction / backtrack  
5. Alternative proposals  

部分来自人类策展的训练数据；很大一块来自合成数据上的 finetune / RL。

**o1 vs GPT-4o：** 数学、数据分析、编程等 reasoning 任务更好；个人写作 / 文本编辑等未必更好。

### 2.9 LLM → Agents

**Chatbot / 单轮 reasoning 的局限：** 好玩，但不替你 **端到端完成任务**。

**Agent 差异（课上定义）：**

1. 有 **goal**
2. **Plan** 步骤
3. 与 **environment** 交互、采取 **actions**
4. 根据 **feedback** 纠正
5. 决定 **何时停止**（完成或报告无法完成）
6. 常需 **tools**（搜索、终端、文件编辑…）与某种 **memory**

**Agentic workflows（现实中更常见的中间形态）：**

- 不是完全开放的「环境闭环」
- 而是人写好的图：LLM 调用、verifier、LLM-as-judge、tool call、编排

课上抽象的几种编排：

| 模式 | 含义 | 例子 |
|------|------|------|
| Prompt chaining | 子任务串成链 | 接近 reasoning 分解 |
| Routing | 按复杂度走不同路径 | 简单走轻量、复杂走重流程 |
| Parallelization | 多路并行再聚合 | Deep Research 多关键词 |
| Orchestrator | 中心 LLM「经理」做计划再调子调用 | Claude Code 式 plan |
| Evaluator / Judge | LLM 评判代替真实环境反馈 | 作业会练 |
| Verifier | 可执行检查（单测等） | 数学 / 代码 |

**真实产品例子：**

- **Deep Research**：查多站、汇总利弊（如租一年房）
- **Claude Code / Codex**：英文指令 → 改文件、写测、跑终端命令
- 客服：转写、Knowledge Assist、smart reply、通话摘要
- 研究报告：Olympics 开幕典礼式课题 → 找引用 → outline → 摘要 → 成文
- **AI Scientist**：想法生成 → 实验迭代 → 论文写作辅助（即使会幻觉，也有「出圈 brainstorm」价值）

### 2.10 Generator–Verifier Gap（本讲点名、L03 深挖）

- 模型很会 **生成** 大量看起来合理的内容
- 难的是 **验证** 是否真有用
- 创意写作等域：人类反馈成瓶颈
- 可验证域（数学、代码）：更容易闭环改进
- 「verification continues to be one of the bottlenecks」

---

## 3. 方法/机制（分技术；伪流程；与前后讲关系）

### 3.1 Pretrain → Align 的伪流程

```text
1. Pretrain: maximize P(next token | context) on web-scale text
2. HQ SFT: same objective on curated books/essays/...
3. Instruction SFT: (instruction, answer) pairs ± CoT demos
4. Preference data: for prompt x, humans rank y_good ≻ y_bad
5. Train reward model R(x, y)
6. RL (e.g. PPO-style): update policy π to ↑ E[R] while staying near SFT policy
```

### 3.2 Repeated Sampling（Monkeys）伪流程

```text
Input: problem x, model π, sample budget K, temperature T, verifier V
For i = 1..K:
    y_i ~ π(· | x; temperature=T)   # 可并行
Select y* = argmax / filter by V(y_i)  # e.g. all unit tests pass
Output: y*
```

与 L02 关系：L01 只建立直觉；L02 给出 coverage 的 **exponentiated power law**、长尾难题条件、以及无完美 verifier 时 majority / RM 的 plateau。

### 3.3 Reasoning Model 的「自改进」闭环（课上叙事）

```text
Test-time: 多样采样 / 长思考 → 在可验证域得到 (x, y_correct, traces)
Train-time: 用这些合成数据做 SFT / RL → 提高 pass@1
再部署 → 更强的 test-time 行为 → 更多更好的合成数据 …
```

这就是课程标题里的 **self-improving** 主线之一。

### 3.4 Agentic Coding Loop（课上图意）

```text
User goal (English)
  → LLM Agent
  → tools: navigate repo / search / view / edit / run terminal
  → observe stdout / test results
  → edit again or read more files
  → until tests pass or give up
```

课上问「为何去年不可靠、现在开始可靠？」：架构未必大变；更强模型 + **RL with verifiable rewards** + 自改进环（能生成更可靠的测试）在起作用。

### 3.5 与前后讲关系速查

| 概念 | 本讲 | 后续 |
|------|------|------|
| Scaling laws | pretrain 三轴 + 预告 inference | L02 正式推导 inference laws |
| Verifier | 用 unit test 举例 | L03 Weaver / ORM / PRM |
| Gen–Ver gap | 点名 | L03 主线 |
| Tools | Deep Research / coding | L04 ReAct 等 |
| Planning | task decomposition | L05 LATS / ADaPT / … |
| RL | RLHF 简介 | L06 STaR / GRPO / DAPO |
| AI Scientist | 应用 vignette | L07 |
| Eval | 项目里强调 hypothesis | L17 长任务 eval |

---

## 4. 关键实验结论与数字

> 说明：下列数字以**课堂口述 / 展示图**为准；更完整实验见 L02 与 papers/。

### 4.1 模型规模史

- BERT 340M → GPT-2 1.5B → GPT-3 **175B** → PaLM **540B** → GPT-4 量级 **万亿**

### 4.2 ChatGPT 增长

- 2022-11 发布；约 **5 天** 到 100 万用户

### 4.3 Repeated Sampling（L01 预告）

- 采样预算：**1 → 10,000**
- 弱模型多次采样可 **超过 GPT-4o 单次**（数学 / 代码等）
- 难题可能在 10k 样本中仅 **3–4** 个正确

### 4.4 o1 Test-time Scaling

- 横轴：test-time compute（**log**）
- 纵轴：难数学上的 **pass@1**
- 形状：**log-linear** 上升（参数不变）

### 4.5 CoT 与模型规模

- ~7–8B 量级：CoT **几乎无效**
- 更大（PaLM 等）：CoT 带来明显数学收益

### 4.6 Temperature（课堂答问）

- 通常 **> ~1.2** 多样性变成胡言；不是「温度越高越好」

### 4.7 成绩结构（Autumn 2025 官网 + 课上）

| 组成 | 占比（官网） |
|------|----------------|
| HW1 / HW2 / HW3 | 15% / 15% / 20%（合计 **50%**） |
| Project（proposal + midterm + final + poster） | 合计 **50%** |

课上口述也强调：**作业 50% + 项目 50%**。

其他 logistics：

- 团队：**2–4** 人（也可 1 人，但不推荐）
- Late days：**4** 天总额；每作业最多 **2**；期末报告 **不能**用 late day；额外每天 **-25%**
- **不允许 audit**；视频最终上 YouTube
- Poster：**Dec 12, 4–6 PM**
- 项目要有 **hypothesis / 实验**，不要纯 survey、不要纯「白写代码的 app」
- 去年有项目最终发论文

---

## 5. 课堂讨论 / Q&A / 易错点

### 5.1 Q&A 精选（来自录音稿）

**Q：Repeated sampling 和「遍历所有可能答案」有何不同？**  
A：答案空间由模型生成；全空间极大。树搜索 / 随机搜索也难「撞」到。人坐下来解题也不是无限尝试。**10,000 次采样对 IMO 级难题仍算 sample-efficient**——小模型偶尔也能解到。

**Q：延迟 vs GPT-4o 单次？**  
A：并行采样对 **latency** 较友好；真正的 frontier 是 **cost / compute**，随题难度变。

**Q：没有 verifier 的域怎么办？**  
A：LLM-as-judge、reward model、工具增强等一整片研究；**专门有一讲 verifiers**。

**Q：对任意正温度都成立吗？温度→∞？**  
A：否。太高变 gibberish；经验上 **>~1.2** 就差。还有其他提高多样性的技巧。

**Q：Reasoning 收益来自「大声思考」本身，还是「让模型分解」？**  
A：数据把模型训成更泛化的 thinker（分解、回退、分析）。另一视角：base 模型本来能生成多种推理链，训练让它学会 **哪条对** → pass@1↑，而不只是 coverage↑。

**Q：能否用小模型做 reasoning、大模型出最终答案？**  
A：通常 reasoning 能力随规模上升；更常见是大模型出 traces，小模型做摘要。模型往往 **更喜欢自己的 traces**（即使别人的更好）。SWiRL 等会讨论「不同模型做评估 / 反馈」。

**Q：Reasoning 是硬编码序列还是 finetune 出来的？**  
A：两者都有；base 已有一定思考能力；课上会讲 ORM / PRM；大量是泛化，不只是模板。

**Q：样本数能否随难度自适应？**  
A：当时课上认为公开工作有限；可用 reward model 驱动「未解决就多采」——值得探索。

**Q：CoT 是偶然发现的涌现，还是训练进去的？**  
A：CoT **最初是涌现/发现**（GSM8K 等 signs of life；PaLM 放大）；模型见过系统性文本。但现代 **reasoning models 被明确训练去多想**；未来会学会「何时该多想」。Agentic 侧在找 planning / multi-step / self-correction——可称涌现，也可称数据+强化。

**Q：为何 RL 带来巨大跃迁？Pretrain 不是已经覆盖了吗？**  
A：仍有争议：有人认为 RL 只是抬 pass@1、coverage 本就在 pretrain 里；有人看到持续 RL 仍能改进。**「第一波 signs of life」已产品化，但机制未完全搞清。**

**Q：Coding agents 架构变了吗？**  
A：范式大体相同；更强模型 + 可验证奖励上的 RL + 自改进（生成更好测试）是关键。

### 5.2 易错点

1. **把 CoT 与 Reasoning Models 混为一谈**：CoT 是 prompting / 涌现起点；reasoning models 是训练目标与长思考行为。  
2. **把 Coverage 当成产品 Pass@1**：没有 verifier 时，coverage 只是上界。  
3. **以为温度越高越好**：过高直接崩。  
4. **以为 Agent = 多调几次 LLM**：缺 goal / feedback / stop / tools / memory 就不完整。  
5. **以为 verification 在代码域「自动完美」**：单测覆盖不足会假阳性（L02 会再强调人工抽查 ~97–98%）。  
6. **项目做成 demo app**：课程要研究问题与实验，不是纯工程展示。

---

## 6. 本讲论文速查（链接 papers/）

本讲以总览为主，论文精读集中在 **L02**。相关预告：

| 论文 | 卡 | 本讲角色 |
|------|----|----------|
| Large Language Monkeys (Brown et al., 2024) | [large-language-monkeys-2024.md](../papers/large-language-monkeys-2024.md) | Infinite monkey + coverage 预告 |
| Scaling test-time compute optimally (Snell et al., 2024) | [scaling-test-time-compute-optimally-2024.md](../papers/scaling-test-time-compute-optimally-2024.md) | L02 主讲；本讲仅铺垫「推理也是 frontier」 |
| Archon (Saad-Falcon et al., 2024) | [archon-2024.md](../papers/archon-2024.md) | L02 |
| How Do Large Language Monkeys Get Their Power (Laws)? (2025) | [how-do-ll-monkeys-get-power-laws-2025.md](../papers/how-do-ll-monkeys-get-power-laws-2025.md) | L02 解释 power law 来源 |

建议阅读：先完成本笔记 → 再读 L02 笔记 → 再读 papers/。

---

## 7. 工程可落地 Checklist

- [ ] 画清你的系统处在哪一环：**pretrain / finetune / inference**；别混谈「我们在 train」。  
- [ ] 若任务可验证（测、证明、仿真等价）：优先设计 **verifier**，再谈采样预算。  
- [ ] 评估时分开报：**pass@1**、**pass@k / coverage**、以及「用 RM/多数票选出的成功率」。  
- [ ] 产品默认温度不要盲目 >1.0；多样性用多 prompt / 多模型 / 工具，而不是高温。  
- [ ] Agent 设计清单：goal 澄清、工具集、停止条件、验证方式、失败回退。  
- [ ] 编排从简单到复杂：chain → parallel → orchestrator；别一上来「全自动开放环境」。  
- [ ] 创意 / 开放域：诚实对待 **human feedback 瓶颈** 与 judge 偏差。  
- [ ] 自改进环：test-time 产生可验证数据 → 过滤 → 再训练 → 再部署。  
- [ ] 成本：并行采样降延迟，但 **$/token 与总 FLOPs** 仍要记账（L02 有 DeepSeek vs GPT/Claude 例子）。  
- [ ] 做研究项目：写清 **hypothesis、指标、对照、失败模式**（单测假绿等）。

---

## 8. 若你只记 5 条

1. **三阶段世界观**：Pretrain 给能力底座；Finetune（instruction + RLHF）给对齐与听话；**Inference scaling** 在固定参数下再挖能力——这是 2024+ 新 frontier。  
2. **Emergent CoT**：大模型才能吃到思维链；reasoning models 进一步把「多想」训进 pass@1。  
3. **Infinite Monkeys**：弱模型 × 大量采样 × 好 verifier，coverage 可超过强模型单次——模型「会」的比「一次问出来」的多。  
4. **Agent ≠ Chat**：需要 goal、行动、环境反馈、停止条件、工具与记忆；现实多为 **agentic workflows**。  
5. **Verifier 是瓶颈**：生成容易、验证难（generation–verification gap）；可验证域才能闭环 self-improve——整门课反复围绕这一点展开。

---

## 附录 A：官网 Schedule（与本讲相关片段）

| # | 日期 | 主题 |
|---|------|------|
| 1 | Mon Sep 22 | Course Overview |
| 2 | Fri Sep 26 | Test-time Compute Scaling（Monkeys / Archon / Snell / Power Laws） |
| 3 | Mon Sep 29 | Robust Verification |

完整表见 https://cs329a.stanford.edu/

## 附录 B：推荐自行画的概念图

1. **Scaling 三轴** → loss ↓  
2. **Pretrain / SFT / RLHF / Inference** 流水线  
3. **Coverage 曲线** vs **Pass@1 曲线**  
4. **Chatbot vs Agent** 对照  
5. **Self-improve 环**：verify → synth data → train → better policy  

## 附录 C：术语英中对照

| English | 中文 |
|---------|------|
| Scaling laws | 缩放律 |
| Emergent abilities | 涌现能力 |
| Chain-of-thought | 思维链 |
| Instruction tuning | 指令微调 |
| RLHF | 基于人类反馈的强化学习 |
| Reward model | 奖励模型 |
| Test-time / Inference compute | 测试时 / 推理时算力 |
| Coverage | 覆盖率（至少一次做对） |
| Pass@k | k 次采样通过率 |
| Verifier | 验证器 |
| Generation–verification gap | 生成–验证差距 |
| Agentic workflow | 智能体式工作流 |
| Orchestrator | 编排器 / 中心规划 LLM |
