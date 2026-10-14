# L03 · Robust Verification（鲁棒验证）

> 课程：Stanford CS329A Self-Improving AI Agents · Autumn 2025  
> 录音对照：`raw-transcripts/3.robustveriftication.md`  
> 官网：[cs329a.stanford.edu](https://cs329a.stanford.edu/)

---

## 1. 一句话总览 / 目标位置

**本讲解决的问题**：L02 证明了「重复采样能抬高 coverage（Pass@K）」，但多数投票等方法无法稳定选出正确答案——这就是 **generation–verification gap（生成–验证鸿沟）**。本讲回答：如何训练 / 组合 **verifier（验证器）**，在 best-of-N 中挑出正确解，并逐步把验证从「整题对错」推进到「逐步打分」，再推进到「无人工标注」与「弱验证器集成」。

**在课程地图中的位置**：

```text
L01 Agent 定义
  → L02 Test-time compute / Gen-Ver gap（发现问题）
    → L03 Robust Verification（如何选对答案）  ← 本讲
      → L04 Feedback + Tools/Code（反馈从哪来）
        → L05+ Planning / RL / Agents
```

**本讲四篇论文的时间线（课堂强调的「验证范式演进」）**：

| 阶段 | 论文 | 核心升级 |
|------|------|----------|
| 2021 | Training Verifiers / GSM8K | 引入 ORM 式 verifier + best-of-N |
| 2023 | Let’s Verify Step by Step | ORM → PRM；人工逐步标注 PRM800K |
| 2023 | Math-Shepherd | PRM **无需人工逐步标注**（MC 式自动标注） |
| 2025 | Weaver（Shrinking Gen-Ver Gap） | **不训新 verifier**，弱验证器 ensemble + 弱监督加权 |

---

## 2. 核心概念与定义

### 2.1 Generator vs Verifier

- **Generator（生成器）**：给定问题 \(q\)，采样候选解 \(r_1,\ldots,r_N\)。
- **Verifier（验证器）**：对 \((q,r)\) 输出分数 \(v(q,r)\)，表示「该解正确 / 质量高」的置信度。
- **Best-of-N / Rejection sampling**：取 \(\arg\max_j v(q,r_j)\) 作为系统输出。
- **Oracle verifier**：总能挑中正确解（只要 coverage 里有正确答案）→ success rate 追上 Pass@K。
- **Generation–Verification Gap**：\(\text{Pass@K} - \text{SuccessRate(verifier)}\)。Gap 大 = 模型「会生成」但「选不出来」。

### 2.2 Outcome vs Process Reward Model

这是本讲最重要的二分法：

| | **ORM（Outcome Reward Model）** | **PRM（Process Reward Model）** |
|--|----------------------------------|----------------------------------|
| 监督粒度 | 整条解 / 最终答案对错 | **每一步**推理是否正确 |
| 标签来源 | 终答与 ground truth 匹配（常可自动） | 人工逐步标注，或自动估计「该步通向正解的潜力」 |
| 测试时分数 | 通常取 **最后一个 token** 的正确概率 | 各步正确概率的 **乘积**（或其它聚合） |
| 假阳性 | 易放过「过程错、碰巧终答对」 | 更易压低此类 false positive |
| 可解释性 | 弱 | 强：能指出「第几步错了」 |
| 成本 | 低 | 人工 PRM 极贵；自动 PRM（Math-Shepherd）换算力 |

课堂定义（意译）：

- **Outcome-based**：reward 给 **整个 solution** 的正确性。
- **Process-based**：给优化 / 推理过程的 **每一步** 不同 reward。

### 2.3 其它课堂关键词

- **GSM8K**：Grade School Math 8K——约 **8500** 道小学数学应用题，强调多步推理 + 自然语言解题过程；至今仍常作小模型评测。
- **PRM800K**：约 **80 万**步级人工标签（Let’s Verify）。
- **Self-consistency (SC)**：多数投票的另一叫法——看哪个终答出现最多。
- **Convincing wrong answers**：终答错、但当前 PRM 给分仍高的样本——主动学习优先让人标这些，数据效率约 **2.6×**。
- **Hard / Soft estimate**（Math-Shepherd）：从某步 rollout \(N\) 条后续轨迹后，用「是否有任何一条对」或「有多少比例对」给该步打标。
- **Weak verifiers**：没有完美 verifier；ORM、PRM、LLM-as-judge、工具/rubric 评分都是弱信号。
- **Weaver**：score → weight → select；用弱监督估计各 verifier 准确率并加权。

### 2.4 为什么「验证」值得单独开一讲？

课上从 L02 直接甩出的问题是：模型似乎「知道」很多难题的答案（coverage 随采样上升），但 **自动选出正确那条** 失败。人类解题时若有 rubric / 逐步检查，正确率会高很多——verifier 就是给模型的「外挂 rubric」。

更深一层（alignment / 可靠性）：

- Outcome 监督允许「歪打正着」→ 表面上对、过程不可信。  
- Process 监督奖励 **人类认可的推理过程**，既提高选解质量，也降低错误信用分配带来的错对齐。  
- 当单个 verifier 不够好时，**弱信号集成** 仍可能逼近 oracle——这是 Weaver 的叙事。

### 2.5 三种「标签成本」阶梯（务必建立数感）

| 层级 | 你要标什么 | 成本 | 代表 |
|------|------------|------|------|
| L0 | 终答对错（可自动） | 极低 | GSM8K ORM |
| L1 | 每步对错（人工） | 极高 | PRM800K |
| L1′ | 每步对错（MC 自动） | 算力换人力 | Math-Shepherd |
| L2 | 几乎不标，只集成现成 RM/Judge | 推理期算力 | Weaver |
---

## 3. 方法 / 机制

### 3.1 Paper 1 — Training Verifiers to Solve Math Word Problems（GSM8K, OpenAI 2021）

**动机**：LLM 会自信地给出错误数学解；需要「像 rubric 一样」判断解是否正确。

**数据集 GSM8K**：

- ~8.5K 题（约 7.5K train / 1K test）。
- 设计原则：高质量、高多样性、中等难度、**自然语言逐步解**（非纯公式）。
- 解约需 2–8 步四则运算；适合诊断多步推理失败。

**训练流程（课堂口述）**：

1. （当时）先对 generator **SFT 约 2 epochs**（课堂：现代模型指令跟随已强，这一步是否必要可商榷）。
2. 每题采样约 **100** 条 completion。
3. 用人工终答做匹配 → 二值标签（correct / incorrect）。
4. 训 **1 epoch** verifier：架构也是 LM + **per-token 标量头**；question token 可 mask；最终解分数取 **最后一个 token** 的预测。
5. 联合目标：二值正确性 loss +（可选）language modeling loss——论文称双目标有帮助。

**测试时**：大量采样 → verifier 打分 → 取最高分。

**消融要点**：

- Sentence-level vs token-level 标签：token-level 更噪，但最终仍用 last-token 聚合。
- **大数据时 verification ≫ 纯 SFT**；小数据（如 175B 上 <~1000）verification 帮助有限。
- **大 generator + 小 verifier** 优于 **小 generator + 大 verifier**（生成平均更难；Pareto 尺寸比仍是开放问题）。
- Best-of-N：约到 **400** 仍有收益；再往 **800** 精度会掉——候选太近时 verifier 分不清对错，系统准确率下降（不是 coverage 下降）。实务上他们常停在 **~100**（增益与稳定性的折中）。

**与 L02 对照**：多数投票大约在 **~50** 或更早就跟不上 coverage；带 ORM 的 best-of-N 可撑到数百——这是「缩小 Gen-Ver gap」的早期证据。

### 3.2 Paper 2 — Let’s Verify Step by Step（OpenAI 2023）

**动机**：多步推理中，**早期一步错会拖垮整解**；仅看终答的 ORM 管不住过程幻觉。

**ORM vs PRM 训练对比**：

```text
问题 q
  → Generator 产出逐步解：s1, s2, …, sT, 终答
  → ORM：只看终答 vs ground truth → 一个标签
  → PRM：人工对每一步标 positive / negative / neutral
       测试分数 ≈ ∏_t P(step_t correct)
```

**为何 PRM 更抗 false positive**：模型可能「过程乱七八糟但碰巧答对」；逐步监督使这种解更难拿高分，也更贴近「人类认可的推理过程」（alignment 视角）。

**PRM800K 数据收集**：

- Generator 产生逐步解 → 人工逐步标注。
- **Active learning**：优先标 *convincing wrong-answer*（PRM 高分但终答错）→ 约 **2.6×** 数据效率。
- 迭代：用新数据重训 PRM → 再选更难骗的样本给人标。

**实验结论（课堂）**：

- Best-of-N 曲线：PRM > ORM > majority voting；多数投票约在 **~100** 样本后失效更明显。
- PRM 能捞出「正确解占比 <5%」的稀有正解。
- PRM **更 sample-efficient**：粗略对照——ORM 每题标 100 解 ≈ PRM 每题标 1 解的信息量量级（课堂类比，非严格等式）。
- **分布外泛化**：majority voting 有时比 ORM 更稳；但 PRM 整体优于多数投票，且更能扛 distribution shift。

**重要 caveat（Q&A）**：

- 现代系统常 **ORM + PRM 组合**。
- PRM 阈值是额外超参。
- 「跳步直接写答案骗 PRM」：若标签是人工、且要求逐步合理，人类会打低分；若用 PRM **训 generator（RL）** 则要警惕 reward hacking——本讲侧重 **冻结 generator、只训 verifier 做搜索**。

### 3.3 Paper 3 — Math-Shepherd（无人工逐步标注的 PRM）

**动机**：PRM 强但人工贵；能否 **自动** 造逐步标签？

**核心定义**：某中间步的质量 = **从该步继续采样后，到达正确终答的潜力**。

对当前步 \(s_t\)，再采样 \(N\) 条 continuation：

| 估计 | 定义 | 例子（N=3，2 条对） |
|------|------|---------------------|
| **Hard** | 任一条 continuation 终答对 → 标成功 | 1 |
| **Soft** | 正确 continuation 的频率 | 2/3 |

**课堂点名的弱点**：

1. **稀有正确路径**：N 太小会漏掉「难但正确」的分支 → 误标 0。
2. **难题无信号**：几乎所有 rollout 都错 → 标签全糊。
3. **错步碰巧终答对**：仍可能被标成「好步」（假阳性）；增大 N 有缓解但不保证。

**用法两条路**：

1. **Verification**：采样 N 候选 → PRM 打分 → 选最高。
2. **RL（PPO）**：用 PRM 作 reward，强化 generator 逐步偏好高分步骤。

**结果要点（课堂 + 论文）**：

- Soft 在 loss 上常更好，但 **N≈4** 时 hard/soft 差不大；最终常用 hard（更简单）。
- **无人工逐步标注** 的 Shepherd PRM：在 GSM8K / MATH 上优于 SC（多数投票）与 ORM；在更难的 MATH 上相对增益更大；甚至可比肩 / 超过 PRM800K 训练的 verifier（视设定）。
- Mistral-7B：PRM-PPO 相对 ORM-PPO 更好；再叠加 test-time verification 可继续涨；后期可能 plateau。
- 多层自改进：模型造标注 → 训 PRM → test-time 选解 + RL 改 generator。

### 3.4 Paper 4 — Shrinking Gen-Ver Gap with Weak Verifiers（Weaver, Stanford / NeurIPS 2025）

**动机**：仍有 Gen-Ver gap；**不训新大 verifier**，而是用 **inference compute + 多个弱验证器集成**。

**「Weak」含义**：不是故意选烂模型，而是承认 **没有 verifier 完美**——哪怕是目前最好的 RM / judge。

**Verifier 类型池**：

- 训练好的 ORM / PRM
- **LLM-as-judge**（问「这对吗？」或带 rubric / 工具）

**朴素 ensemble 不够**：各 verifier 质量差异大；等权平均会被差的拖累。有标签时，**加权**（Naive Bayes / logistic regression 等学权重）明显更好。

**Weaver 流程：Score → Weight → Select**

1. **Score & normalize**：统一量纲；用少量标签 **过滤** 质量过低的 verifier（课堂强调：这是关键步骤）。
2. **Weight（弱监督）**：在「各 verifier 捕捉正确性的独立侧面」假设下，用类似 Snorkel / 弱到强监督的思路估计准确率与组合权重；可用极少标签（课堂：约 **1%** train labels 的 unsupervised 变体）。
3. **Select**：对每个候选解得到聚合分，取最高。

**假设与直觉**：若所有 verifier 永远打一样的分，就学不到新信息；**一致性与不一致性** 才是可学信号。

**可扩展的验证算力轴**：

- 更多 generations（10 → 100 → 1000）
- 更大 generator / verifier
- 更多 verifier 进池
- 在固定 FLOPs 下如何分配：生成 vs 验证（开放题）

**结果印象（课堂）**：

- Weaver（含监督 / 弱监督）≫ Naive Ensemble ≫ Majority Voting / Multi-Agent Verification（MAV，多角度 prompt 打分）等。
- 难题上可有大幅跃迁（课堂举例：约 40%+ → 70%+ 量级），逼近更强推理模型档位。
- Llama 3.1 8B Instruct 作 generator + ≤8B verifier 池 → 多数据集平均约 **70%** 量级（缩小「模型档位」差距）。
- 论文进一步：把 Weaver 分数蒸馏到 **~400M cross-encoder**，保留大部分精度并大幅砍验证算力。

### 3.5 与前后讲的机制串联

```text
L02: 重复采样抬高 Pass@K，但选不出 → Gen-Ver gap
L03: 训 ORM/PRM，或弱验证器加权集成 → 抬 Success Rate
L04: 反馈还可来自工具执行、代码测试、自我批判（Constitution）
L06: 把 reward model 塞进 train-time RL（与 Math-Shepherd PPO、RLEF 呼应）
```

---

## 4. 关键数字与实验结论

| 来源 | 数字 / 现象 | 含义 |
|------|-------------|------|
| GSM8K | ~8500 题 | 多步小学数学基准 |
| Verifier 训练 | 每题 ~100 样本；SFT 2 epoch + verifier 1 epoch | 经典 ORM 流程 |
| Best-of-N | ~400 仍涨；~800 可能掉；实务 ~100 | Verifier 精度限制 N |
| Gen-Ver（L02 对照） | 多数投票常 ≤~50 失效 | 需更强 verifier |
| 尺寸搭配 | 大 gen + 小 ver 更好 | 生成更难 |
| PRM800K | ~800K 步标签 | 人工过程监督规模 |
| Active learning | ~2.6× 数据效率 | 优先标 convincing wrong |
| MATH 子集 | PRM best-of-N ~78%（论文） | 过程监督 SOTA 级 |
| Math-Shepherd | Mistral-7B GSM8K 77.9%→84.1%（PPO）；+verify →89.1% | 自动 PRM + RL + 验证 |
| Hard/Soft | N≈4 时接近；常用 hard | 自动标注实用折中 |
| Weaver | 相对 Pass@1 / majority 大幅收窄 gap；8B 档可达 ~70% 平均 | 弱验证器集成有效 |
| 蒸馏 | ~400M 保留 Weaver ~98%+ 精度，验证 FLOPs 可降数个数量级 | 工程可落地 |

**Best-of-N 与 verifier 错误率（务必记住）**：

- N 增大 → coverage 升，但候选解彼此更「像」→ **verifier 相对误差更致命**。
- 系统准确率 = f(coverage, verifier precision)；verifier 不准时，N 过大反而掉点。
- 因此：**先提高 verifier 质量 / 用 ensemble 降噪，再盲目加 N**。

---

## 5. 课堂 Q&A 与易错点

1. **标签从哪来？** 人只写一次标准终答；模型采样多解后与终答匹配得到 ORM 标签——不是每解都重新人工写完整解答。
2. **为何 N>400 准确率掉而不是平台？** 看的是「verifier 选出来的那条」对不对；N 大时易混淆近邻对错解，**precision 掉**。
3. **要对 verifier 做 test-time scaling 吗？** 可以也值得做（本讲末 Weaver；后续还可研究固定算力下 gen vs ver 分配）。
4. **大数据时 SFT 和 verification 会不会殊途同归？** 可能收敛类似；但 verification 的美在于 **不把 base generator 绑死在单一任务上**，verifier 可插拔。
5. **PRM 会不会错误 credit assignment？** 会；实务常 ORM+PRM；阈值是超参。
6. **ORM vs PRM 标签预算怎么公平比？** PRM 每解多步标签，难严格 apples-to-apples；论文尝试控制但仍有争议。
7. **多数投票比的是整段推理还是终答？** 本讲语境 **比终答**。
8. **跳步骗 PRM？** 人工标签可惩罚不合理跳步；用 PRM 训 generator 时更危险。
9. **Math-Shepherd 自动标签三大坑**：稀有路径、难题无信号、错步好运。
10. **弱验证器 ≠ 烂验证器**；关键是过滤差的 + 学权重，而不是无脑平均。
11. **独立假设**：Weaver 假设各 verifier 捕捉相对独立的正确性侧面——高度相关的池子增益有限。

---

## 6. 本讲论文速查

| 论文 | 笔记卡 |
|------|--------|
| Training Verifiers / GSM8K | [papers/training-verifiers-gsm8k.md](../papers/training-verifiers-gsm8k.md) |
| Let’s Verify Step by Step | [papers/lets-verify-step-by-step.md](../papers/lets-verify-step-by-step.md) |
| Math-Shepherd | [papers/math-shepherd.md](../papers/math-shepherd.md) |
| Shrinking Gen-Ver Gap (Weaver) | [papers/weaver-weak-verifiers.md](../papers/weaver-weak-verifiers.md) |

---

## 7. 工程可落地 Checklist

- [ ] 先测 **Pass@K / coverage**，确认 gap 主要在「选」还是「根本生成不出来」。
- [ ] 有自动终答检查（数学/代码）→ 先上 **ORM + best-of-N**，画 N–准确率曲线，找到拐点。
- [ ] 假阳性（过程错终答对）伤害大 → 引入 **PRM** 或逐步检查；人工贵则试 **Math-Shepherd 式 MC 标注**。
- [ ] Active learning：优先标 **高分错解**，别均匀标。
- [ ] Generator / Verifier 尺寸：优先保证 **generator 够强**；verifier 可略小，但要校准。
- [ ] 多 RM / LLM-judge 可用时：先 **质量过滤**，再 **加权**（有少量标签更好）；避免等权平均。
- [ ] 监控 verifier 校准与近邻混淆；N 过大时优先改 verifier 而非盲加采样。
- [ ] 固定 FLOPs 预算：显式权衡「多样本生成」vs「多样本/多模型验证」。
- [ ] 若要用 PRM 做 RL：防 reward hacking（跳步、讨好打分器）；可保留 ORM 终答门闩。
- [ ] 生产中考虑把 ensemble 分数 **蒸馏** 成小 cross-encoder，降延迟。

---

## 8. 若你只记 5 条

1. **Gen-Ver gap**：会生成 ≠ 会选中；verifier 决定 repeated sampling 能否变现。  
2. **ORM 看终局，PRM 看步骤**；PRM 更抗「歪打正着」，但人工逐步标注极贵。  
3. **Math-Shepherd** 用「从该步 rollout 能否到达正解」自动造 PRM 标签，打通无人工过程监督。  
4. **Best-of-N 受 verifier 错误率限制**：N 过大时近邻混淆，准确率可能下降。  
5. **Weaver**：过滤 + 弱监督加权集成多个弱验证器，用推理算力缩小 gap，甚至抬高小模型档位表现。

---

## 9. 决策表：什么时候用什么验证器

| 场景 | 优先方案 | 原因 |
|------|----------|------|
| 终答可自动检查、预算紧 | ORM + best-of-N | 标签便宜、实现快 |
| 假阳性伤害大（教学、安全关键推理） | PRM 或 ORM∧PRM | 管过程 |
| 无人工逐步预算 | Math-Shepherd 式自动 PRM | 用算力换标注 |
| 已有多个 RM / Judge API | Weaver 式过滤+加权 | 不训新大模型也能抬点 |
| 只要「最常出现的答案」 | Majority / SC | 无 verifier 时的基线；N 大易失效 |
| 要改 generator 本身 | PRM/ORM → RL（PPO 等） | 进 L06；防 hacking |
| 延迟敏感线上服务 | 蒸馏小 verifier | Weaver 论文路径 |

## 10. 公式与伪流程速查

### 10.1 Best-of-N

\[
\hat{r} = \arg\max_{j=1\ldots N} v(q, r_j),\quad r_j \sim \mathrm{Generator}(q)
\]

成功当且仅当 \(y(q,\hat{r})=1\)。Oracle 时成功率 → Pass@N。

### 10.2 Gen–Ver Gap

\[
\mathrm{Gap}(N) = \mathrm{Pass@N} - \mathrm{SuccessRate}_v(N)
\]

### 10.3 PRM 解分数（乘积聚合）

\[
v_{\mathrm{PRM}}(q,r) = \prod_{t=1}^{T} P(\text{step}_t \text{ correct}\mid q, s_{1:t})
\]

（也可用 min / 几何平均等变体；课上主讲乘积。）

### 10.4 Math-Shepherd soft/hard

从步骤 \(s_t\) rollout \(N\) 条完整解，设正确条数为 \(c\)：

- Soft: \(c/N\)
- Hard: \(\mathbf{1}[c\ge 1]\)

### 10.5 Weaver 聚合（直觉）

在 verifier 条件独立假设下，由各 \(v_k\) 的打标估计 \(P(y=1\mid v_{1:m})\)，再对候选 \(j\) 取最大后验。实践上先 normalize + filter，再学/估权重。

### 10.6 ORM 训练伪流程

```text
for question q in train:
  samples = generate(generator, q, n=100)
  for s in samples:
    label = (extract_answer(s) == gold(q))
    store (q, s, label)
train verifier on store  # binary (+ optional LM loss)
# test: best-of-N by verifier last-token score
```

## 11. 与 L02 / L04 的对照记忆

| 问题 | L02 | L03 | L04 |
|------|-----|-----|-----|
| 多采样有用吗？ | 有，coverage↑ | — | — |
| 如何选？ | 点出 gap | ORM/PRM/Weaver | 执行反馈、原则批判也可当信号 |
| 反馈从哪来？ | 隐含要 verifier | 训/集成 verifier | 工具 obs、测试、Constitution |
| 是否改权重？ | 通常不 | 可选 RL | RLEF / RLAIF 会改 |

## 12. 课上「可当研究项目」的提示（录音）

1. Generator vs Verifier 尺寸的 **Pareto 最优比**（今日强 base + 多开源 RM 条件下重做）。  
2. 固定 FLOPs 下：更多生成 vs 更多/更强验证 vs 更多 verifier 的分配。  
3. ORM vs PRM 在 **控制标注预算与算力** 后的公平对比。  
4. 对已 PPO 过的模型再生成标签、再训 verifier 的二次循环。  
5. 给 PRM 加 rubric / 工具（计算器）以激励自我纠错。

## 附：自学路径建议

1. 重画一张表：ORM / PRM / 自动 PRM / Weak ensemble 的标签成本、测试用法、典型失败模式。  
2. 读 GSM8K 与 Let’s Verify 的 best-of-N 图，对照 L02 多数投票失效点。  
3. 实现最小 ORM（终答匹配标签 + last-token 打分）再对比 majority vote。  
4. 选一题手算 Math-Shepherd hard/soft。  
5. 若有多个 RM API：做一次加权 vs 等权 ablation（哪怕只用 10 道带标签题学权重）。  
6. 用本讲第 9 节决策表，为自己的课程 project 选一条验证栈并写清失败模式。
