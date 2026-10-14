# L02 — Test-Time Compute Scaling：重复采样、最优分配与 Archon

> 一句话总览：在**不改模型参数**的前提下，用更多推理算力（并行采样、串行修订、PRM 搜索、多技术编排）可显著抬高能力；关键瓶颈是 **verifier**；coverage 对采样数常呈 **exponentiated power law**，其微观来源是「每题指数衰减 + 难题 pass@1 长尾」。

---

## 1. 本讲目标与在课程中的位置

### 1.1 与 L01 的衔接

L01 建立了三阶段：

| 阶段 | 算力特征（课上） |
|------|------------------|
| **Pretraining** | 历史上最贵：数月、大量 GPU、**数万亿 tokens** 量级 |
| **Fine-tuning** | 数据量通常 **低几个数量级** |
| **Inference** | 过去几乎「一次问答就结束」；现在可 **离线/在线大量烧 tokens** |

本讲专攻第三阶段：**固定参数，如何 scale test-time compute**。

### 1.2 必读论文（官网 L2）

| 论文 | arXiv | 课上角色 |
|------|-------|----------|
| Large Language Monkeys | [2407.21787](https://arxiv.org/abs/2407.21787) | repeated sampling + coverage laws |
| Archon | [2409.15254](https://arxiv.org/abs/2409.15254) | 推理架构搜索（iTAS） |
| Snell et al. Scaling test-time compute optimally | [2408.03314](https://arxiv.org/abs/2408.03314) | parallel vs sequential；compute-optimal |
| How Do Large Language Monkeys Get Their Power (Laws)? | [2502.17578](https://arxiv.org/abs/2502.17578) | 为何 aggregate 是 power law |

### 1.3 本讲要带走的能力

学完应能：

1. 画清 **generation → verification → selection** 流水线  
2. 区分 **coverage / pass@k** 与 **pass@1（经 imperfect verifier）**  
3. 解释 **generation–verification gap** 为何在难题上更大  
4. 在 parallel / sequential / PRM beam 之间做 **按难度分配**  
5. 理解 Archon：把 inference 当成 **可搜索的架构空间**

### 1.4 在课程中的位置

```text
L01: 提出 Monkeys / o1 直觉
L02: 机制 + 实验数字 + 最优缩放 + Archon   ← 本讲
L03: 专门缩小 gen–ver gap（Weaver / ORM / PRM 训练）
```

---

## 2. 核心概念与定义（含课上强调细节、数字、图意）

### 2.1 Repeated Sampling（Large Language Monkeys）

**设定：**

1. 同一问题输入，对 LLM **反复询问**（10 / 100 / … / 10,000 次）  
2. 用 **verifier** 选出正确响应作为系统输出  
3. 模型参数 **不动**

**课上核心观察：**

- Llama-3-8B / 70B 等单次弱于 GPT-4o，但多次采样 + 选对后可 **超过 GPT-4o 单次**  
- 横跨数学、竞赛编程、形式化证明、**SWE-bench** 等  
- 解读：小模型「已经知道」很多答案，只是第一次不一定说出来

**SWE-bench 课上数字（DeepSeek 等）：**

- 横轴样本数约 **1 → 1,000**  
- Coverage：DeepSeek-V3 类模型在 ~1000 samples 后可 **超过 Claude 3.5 / o1-preview 单次**  
- 意义：有 unit tests 时，开源模型 + 自动化选样 = 端到端更强系统

论文补充（与课一致）：DeepSeek-Coder-V2-Instruct 在 SWE-bench Lite 上 **15.9% (1 sample) → 56% (250 samples)**，超过当时单次 SOTA **43%**。

### 2.2 Coverage 与 Inference Scaling Law

**Coverage 定义：**

> 在预算 \(k\) 次采样下，**至少有一个样本做对**的题目占比。

课上拟合形式（exponentiated power law）：

\[
\text{coverage}(k) \;\sim\; \text{函数}(a,b,k)
\]

口语表述：coverage 与并行采样数之间呈现可用 **指数化幂律 / log-linear 类** 的可预测关系；可用曲线拟合 \(a,b\)，从而 **预测「要达到某 coverage 需要多少样本」**。

**实验覆盖范围（课上）：**

- 模型：Llama-3 8B/70B、Gemma、Pythia 等  
- 参数：**70M → 70B**  
- 即便 **70M** 也看到 scaling 行为  
- 跨域成立

### 2.3 单题指数 vs 全集幂律（Power Laws 论文 + 课上推导）

**单题：** 设题 \(i\) 的 pass@1 为 \(p_i\)，独立采样 \(k\) 次：

\[
\operatorname{pass}_i@k \;=\; 1 - (1-p_i)^k
\]

因此「全错概率」随 \(k\) **指数衰减**——单题上不应直接期望多项式。

**全集：** 对数据集平均后，课上观察到 **power law 型 coverage scaling**。

**充分必要条件（课上强调）：**

> 数据集上 pass@1 的分布必须有 **难题的长尾（long tail of hard problems）**。

图意：

- 大量简单题：pass@1 高，少量样本就覆盖  
- 长尾难题：pass@1 极低，需要巨大 \(k\) 才能碰到正确样本  
- 正是这组长尾把「每题指数」聚合成「全局幂律」

Power Laws 论文进一步：\(p(\text{pass@1})\) 左尾若似 Beta 等重尾，则 \(-\log(\text{平均 pass@}k)\) 对 \(k\) 呈幂律；并可用分布视角 **低一个数量级相对误差 / 少 2–4 个数量级推理算力** 去预报指数。

### 2.4 为何需要 Automated Verification

有了多样本，若不能挑出正确者，coverage 只是 **上界**。

| 域 | Verifier 例子 | 课上备注 |
|----|---------------|----------|
| 数学形式化 | Formal proof checker（Lean 等） | 逐步可检 |
| 代码 | Unit tests | 写测往往比写完整程序「更简单」（有争议但常用） |
| AI as compiler / CUDA | KernelBench：PyTorch 与 CUDA **同输入同输出** | 「默认近乎完美」的等价性检查 |
| 语言互译 / porting | 行为等价性 | Python↔Java 等 |

**无自动 verifier 时：**

- Majority vote、model-based ranker、reward model  
- 与 **true coverage** 之间存在巨大差距 → **generation–verification gap**

### 2.5 Generation–Verification Gap（本讲核心瓶颈）

课上图意（GSM8K vs MATH）：

| 曲线 | 含义 |
|------|------|
| 蓝：Coverage | 完美选择器（oracle） |
| 绿：Majority voting | 选出现次数最多的答案 |
| 其他：ORM best-of-n 等 | LLM 打分选最佳 |

观察：

- Majority 大约在 **10–50** samples 后 **plateau**  
- Coverage 继续随采样上升  
- **难题（Capital MATH / MATH）差距更大**  
- 原因：最难的题可能在 1k–10k 样本里只对 **1–3 次**——多数票永远找不到「稀有正确解」

论文呼应（Monkeys）：Llama-3-8B-Instruct 在 MATH 上 coverage **82.9%@100 → 98.44%@10k**，但 majority/RM 在同区间几乎不动（约 **40.5% → 41.4%**）。

### 2.6 Parallel vs Sequential Test-Time Scaling（Snell）

| 模式 | 机制 | 直觉 |
|------|------|------|
| **Parallel** | 独立采 N 个完整答案 | 探索不同高层策略 |
| **Sequential revision** | 同一条思路不断改写/补充 | 初解大致对时精修更划算 |

课上说明：

- Prompting 可诱导 revision；reasoning models **内部已训练出** 类似修订行为  
- 选择器：  
  - **ORM（Outcome Reward Model）**：看最终答案打分  
  - **PRM（Process Reward Model）**：对 **每一步** 打分（非 per-token）

### 2.7 PRM + Beam Search（课上伪流程）

```text
每层预算例如 4 个候选
→ PRM 打分
→ 保留 top-2
→ 从保留节点继续采样
→ 树式扩展（beam search）
```

要点：

- PRM 通常由 LM finetune；**in-domain** 更好，也有一定跨任务泛化  
- 步的定义：人类标注的「有意义块」或句子级等（细节 L03）  
- 可把 **parallel + sequential** 组合：多条并行轨迹，每条内部修订，再用 ORM/PRM 选

### 2.8 按难度的 Compute-Optimal Allocation

Snell 设定（课上）：

- MATH：**~12k train / 500 test**；PaLM 系模型  
- 用 pass@1（大量采样估计）把题分成 **5 个难度 bin**（1 易 → 5 难）  
- 观察 **sequential : parallel** 配比：

| 难度 | 课上倾向 |
|------|----------|
| 容易 | 更深色 = 更多 **sequential** tokens 更优 |
| 很难（bin 4–5） | 最优配比 **不稳定**；更需要探索 |

**Test-time vs Pretraining（FLOPs-matched 叙事）：**

- Easy / Medium：额外 test-time 往往 **优于** 单纯放大 pretrain  
- Hardest：更大 / 更多 pretrain 的 frontier 模型仍更好——即使给「合理大」的 test-time budget  
- 工程含义：不是人人都能 pretrain；但对多数题，**小模型 + 多推理** 很有用

课上回应「pretrain 只付一次、inference 每次都付」：仍要比较 **边际收益**；且多数组织根本付不起更大 pretrain。

### 2.9 Archon：Inference 当作 Architecture Search

**问题：** 如何混合 parallel / sequential / 多模型 / 多技术，在 **准确率 vs 成本（inference call budget）** 上推 Pareto。

**输入：**

- Target benchmarks  
- Inference call budget  
- Available LLMs  
- Available inference-time techniques  

**优化器：** **iTAS**（inference-time architecture search），课上用 **Bayesian optimization**

**输出：** 一层层堆叠的推理架构（像「推理版深度网络」，但层间无梯度权重）

### 2.10 Archon 组件（课上定义）

| 组件 | 做什么 |
|------|--------|
| **Generator** | 标准采样；可 n 次 / 多模型 ensemble |
| **Fusion** | 把 K 个回答合成 **一个**；「看到所有可能解法再写」——课上惊喜地强 |
| **Critic** | 评优缺点 |
| **Ranker** | Prompt 排序，取 top |
| **Verifier** | 带理由的打分 / 判定 |
| **Unit test gen + eval** | 生成测例；甚至让模型 **对照测例评判答案**（不必真跑） |

**Fusion 课上关键图：**

- Random < Ranker < Oracle selection < **Fusion**；Fusion 可 **超过 Oracle**（因为合成而非只选）  
- Top-k filter 再 Fusion（紫）往往更好  
- 多模型 ensemble 时趋势类似；通常用最好的模型做 fusion

**深度堆叠：** 像深度学习加层——更多 critic/fuser 层可继续涨点（任务相关）。

**搜索空间约束（降成本）：**

- 每层一种技术；第一层总是 generator  
- Critic 应在 Ranker/Fuser 前  
- Unit test generator 后必须跟 evaluator  
- Offline 先发现好的 motif（如 gen→critic→rank→fuse）再缩小搜索

**课上结果数字：**

- 仅用开源模型即可在多个任务上 **匹配/超过** 当时闭源 frontier 的 **pass@1**  
- 平均相对 GPT-4o / Claude 3.5 Sonnet：**+14.1%** pass@1（课上口述；论文摘要亦称约 **15.1%**）  
- 可做 **task-specific** 或 **general-purpose** Archon；通用版仍泛化良好

---

## 3. 方法/机制（分论文/分技术；伪流程；与前后讲关系）

### 3.1 Monkeys：端到端伪代码

```text
function RepeatedSample(x, π, K, V):
    S ← []
    parallel for k in 1..K:
        S.append( sample(π, x, T>0) )
    if V is automatic:          # unit tests / proof checker / I/O match
        return any s in S with V(s)=true   # or first passing
    else:
        return Aggregate(S)     # majority / ORM / ranker  ← 常 plateau
```

**Coverage 无偏估计（论文公式，课上等价叙述）：** 对每题先采 N 个，用 Chen et al. pass@k 组合公式估计，降低方差。

### 3.2 从单题到幂律：诊断检查清单

```text
1. 估每题 p_i = pass@1（可先用中等 N）
2. 画 p_i 直方图：是否左尾长？
3. 若有长尾 → 期望 aggregate coverage 近似 power-law 可拟合
4. 用 (a,b) 外推更大 K 的资源需求
5. 若无长尾（过简单集）→ scaling 形状会变（Power Laws 论文解释偏离）
```

### 3.3 Snell：Compute-Optimal 伪流程

```text
Offline:
  For each q in val:
      estimate difficulty bin b(q) via pass@1 or PRM-predicted
  For each bin b, budget N:
      pick θ*(b,N) = argmax accuracy among
         {best-of-N, beam+PRM, revision depth, mix ratios, ...}

Online:
  estimate b(q) ≈ model-predicted difficulty
  run θ*(b, N_budget)
```

课上强调：真实部署要平衡「估难度的算力」vs「解题算力」（探索-利用）；论文实验为简单起见常忽略估难度成本。

**组合示例：**

- 2 条并行，各自做 sequential revision → 2 个终答 → ORM/PRM 选优  
- 或 PRM beam 引导每条路径

### 3.4 Archon 搜索伪流程

```text
Space = sequences of layers from {Gen, Fuse, Critic, Rank, Verify, UT-gen, UT-eval}
         under ordering constraints + model choices + sample counts

BayesianOptimize:
  propose architecture A
  eval accuracy on train/held-out under call budget B
  update surrogate
return Pareto-good A*

Deploy A* → single final answer  # 优化的是 pass@1
```

### 3.5 与前后讲关系

| 本讲产出 | 下一讲用法 |
|----------|------------|
| Gen–Ver gap 图 | L03 目标：缩小 gap（弱验证器集成 Weaver 等） |
| ORM / PRM 定义 | L03 训练与 step-level 标注 |
| Unit-test-as-verifier | L04/L13 代码 agent、CodeMonkeys |
| Archon fusion/critic | 与 multi-agent / orchestrator 思想相通（L05+） |
| Synth data from verified samples | L06 train-time RL 的数据来源叙事 |

---

## 4. 关键实验结论与数字

### 4.1 Large Language Monkeys

| 结果 | 数字 / 现象 |
|------|-------------|
| 采样范围 | 常至 **10,000**/题；SWE 至 **250** |
| SWE-bench Lite | DeepSeek **15.9% → 56%**；SOTA 单次 **43%** |
| Gemma-2B CodeContests | pass@1 **0.02% → pass@10k 7.1%**（>300×） |
| 成本例 | DeepSeek ×5 attempts 解题率高于 GPT-4o/Claude×1，且约 **3×+ 更便宜** |
| FLOPs 固定 | MATH/GSM/MiniF2F 上 **8B 多样本** 常优于 70B；CodeContests 常相反 |
| Verifier 缺口 | MATH majority/RM ~**40%** 平台 vs coverage ~**98%** |

课上手动质检：数学答对抽查约 **97–98%**；但单测覆盖不足仍是失败模式。

### 4.2 Snell et al.

| 结果 | 数字 / 现象 |
|------|-------------|
| 数据 | MATH 12k/500；PaLM 2-S* |
| 难度 | 5 bins（pass@1 分位数） |
| Compute-optimal vs best-of-N | 可达约 **4× 更少** test-time compute 同水平 |
| FLOPs-matched | 小模型 + 最优 test-time 可胜过约 **14× 更大** 模型（在易/中等题） |
| Beam vs BoN | 低预算 beam 好；高预算易 **过优化 PRM**（短解、重复步） |
| Lookahead | 同预算下常不如更简单方法（模拟成本高） |

### 4.3 Archon

| 结果 | 数字 / 现象 |
|------|-------------|
| 平均提升 | 课上 **+14.1%** pass@1 vs GPT-4o / Claude 3.5 Sonnet；论文 ~**15.1%** |
| 相对其他推理架构 | 约 **20% 更少** calls、更少 in/out tokens（论文） |
| 仅开源 | 平均超 SOTA LLM 约 **11.2%**（论文） |
| Fusion | 可超过 oracle selection |
| 深度 | 多层 critic/fuse 持续有益（任务相关幅度：MixEval 小、MT-Bench 大） |

### 4.4 Power Laws 论文

| 结果 | 数字 / 现象 |
|------|-------------|
| 单题 | \(-\log(\mathrm{pass}_i@k)\) **指数**降 |
| 聚合 | \(-\log(\mathrm{avg\ pass}@k)\) **幂律**降 |
| 机制 | pass@1 分布 **重左尾** |
| 预报 | 用分布拟合约 **低 1 数量级相对误差** 或少 **2–4** 数量级算力 |

### 4.5 课上其他数字

- Majority plateau：约 **10–50** samples  
- Temperature：经验 **>~1.2** 变差（L01，本讲沿用）  
- HuggingFace 公开 **10k samples/题** 数据集，适合做缩小 gap 的课项目  

---

## 5. 课堂讨论 / Q&A / 易错点

### 5.1 Discussion 学生观点（课上）

1. **有好 verifier → 准确率大涨；verifier 质量决定上限**  
2. 迭代修订 / 下一轮改写（指向 sequential）  
3. 混合知识图谱 / RAG：先检索，不准再重采样（效率）  
4. 先「自学探索」再并行采样（self-study + search + tools——后续课）  
5. **难证其真、易证其假**：过滤明显错解；仿真/分子动力学等 noisy verifier  
6. **生成多个弱 verifier 再多数票** → 指向 L03 Weaver（10–20 个，不是 1000；很贵）  
7. Coverage 是否「真解决」？数学人工查 ~97–98%；SE 上单测覆盖不足会假绿  

### 5.2 Snell 段 Q&A

**Q：Revision 靠 prompt 还是模型内生？**  
A：本实验可视为 prompt/finetune；reasoning 模型内部已会修订。

**Q：PRM 按 token 吗？**  
A：**按 step**，不是 per-token。

**Q：为何难题仍偏爱大模型 pretrain？**  
A：小模型可专域 finetune，但在「通用数据配方」比较下，最难问题上更大 pretrain 仍赢。

**Q：Tree search？**  
A：本质是 parallel+sequential 混合；PRM beam 用来剪枝。

**学生直觉：** 易题任意路径易成功 → 少并行、多重修；难题需多并行找「一条活路」——与课上 bin 观察一致。

### 5.3 Archon 段 Q&A

**Q：谁做 fusion？**  
A：实验里常用 ensemble 中 **最好的那个模型**；按从强到弱加入模型。

### 5.4 易错点清单

1. **把 coverage 当上线指标**：没有 verifier 时不可部署。  
2. **难题上迷信 majority vote**：稀有正确解会被淹没。  
3. **无限升温求多样**：>1.2 常崩。  
4. **PRM beam 预算越大越好**：易过优化、掉点。  
5. **以为 test-time 可完全替代 pretrain**：最难问题不行。  
6. **Archon 无约束乱搜**：推理搜索极贵，必须先验剪枝。  
7. **混淆 pass@k 与 Archon 的 pass@1**：Archon 最终仍输出一个答案。  
8. **单测 = 真正确**：覆盖不足是系统性风险。

---

## 6. 本讲论文速查（链接 papers/）

| 论文 | 笔记卡 |
|------|--------|
| Large Language Monkeys (2024) | [large-language-monkeys-2024.md](../papers/large-language-monkeys-2024.md) |
| Archon (2024) | [archon-2024.md](../papers/archon-2024.md) |
| Scaling LLM Test-Time Compute Optimally (Snell et al., 2024) | [scaling-test-time-compute-optimally-2024.md](../papers/scaling-test-time-compute-optimally-2024.md) |
| How Do Large Language Monkeys Get Their Power (Laws)? (2025) | [how-do-ll-monkeys-get-power-laws-2025.md](../papers/how-do-ll-monkeys-get-power-laws-2025.md) |

---

## 7. 工程可落地 Checklist

### 7.1 上线前

- [ ] 任务是否 **自动可验证**？若否，先投资 verifier/RM，而不是盲目加 K。  
- [ ] 分开记录：coverage、majority、RM-selected、最终产品 pass@1。  
- [ ] 画 pass@1 直方图，确认是否有 **难题长尾**（决定采样是否还值得）。  
- [ ] 定温度与多样性策略（多 seed / 多 prompt / 多模型）。  

### 7.2 分配算力

- [ ] 易/中题：优先试 **revision / 更深串行**。  
- [ ] 难题：加 **并行探索**；必要时 PRM beam，但防过优化。  
- [ ] 做小规模难度分箱，得到 θ*(bin, budget) 再部署。  
- [ ] FLOPs/$ 固定时，比较「小模型×多样本」vs「大模型×少样本」（任务相关）。  

### 7.3 Archon 式系统

- [ ] 组件库：generate / fuse / critic / rank / verify / tests。  
- [ ] 固定安全排序规则，缩小搜索。  
- [ ] 优化目标含 **call budget**，不是只追分。  
- [ ] Fusion 前先 top-k 过滤。  
- [ ] 编码任务优先 unit-test 层。  

### 7.4 数据与研究

- [ ] 保存大规模采样（课上 HF monkey_business）供训更好 verifier。  
- [ ] 人工抽检 verifier 假阳性。  
- [ ] 项目想法：缩 gap、自适应 K、弱 verifier 集成、noisy simulation reward。  

---

## 8. 若你只记 5 条

1. **Repeated sampling**：弱模型 + 大 K + 好 verifier，coverage 可超强模型单次；能力常「藏在分布里」。  
2. **Inference scaling law**：coverage 随 K 可预测缩放；微观是每题指数，宏观幂律来自 **难题长尾**。  
3. **Bottleneck = verification**：majority/RM 早早 plateau，gen–ver gap 在难题上最大。  
4. **Compute-optimal**：parallel vs sequential / search 应按 **题目难度** 分配；可得 ~4× 效率；最难题仍要更大 pretrain。  
5. **Archon**：把 test-time 技巧当可搜索架构；Fusion 可超 oracle；开源组合可在 pass@1 上打过当时 frontier（课上 +14% 量级）。

---

## 附录 A：三阶段算力再对比（课上原话结构）

```text
Pretrain:  months, huge GPU fleets, trillions of tokens
Finetune:  orders of magnitude less data
Inference (old):  ~1 forward per user query
Inference (new):  10^2–10^4 samples, search, multi-layer Archon, offline agents
```

## 附录 B：何时用哪招（速查表）

| 场景 | 优先策略 |
|------|----------|
| 有强 unit test / 证明器 | Monkeys 式加大 K |
| 答案短、易重合 | Majority 可能够用（易题） |
| 答案稀有正确 | 必须更好 verifier，别靠投票 |
| 初解常接近正确 | Sequential revision |
| 策略空间大 | Parallel 或 PRM search |
| 多技巧不知如何拼 | Archon / Bayesian search |
| 极难题、预算有限 | 考虑换更强底座模型 |

## 附录 C：公式速记

**单题：**

\[
\operatorname{pass}_i@k = 1-(1-p_i)^k
\]

**聚合 coverage：**

\[
\operatorname{pass}_\mathcal{D}@k = \mathbb{E}_{i\sim\mathcal{D}}[\operatorname{pass}_i@k]
\]

**课上拟合叙事：** \(\log(\text{coverage})\) 与 \(k\) 常近幂律/可拟合外推。

**Compute-optimal（Snell）：**

\[
\theta^*_{q}(N)=\arg\max_\theta \Pr[y\sim\mathrm{Target}(\theta,N,q)\text{ correct}]
\]

## 附录 D：与 L01 Reasoning Models 的对照

| | Monkeys | o1-style reasoning |
|--|---------|-------------------|
| 主要抬高 | Coverage / pass@k | Pass@1 |
| 参数 | 固定 | 通常经 RL/SFT 改变「如何想」 |
| 外显 | 多样本 | 长思维链 / 自修正 |
| 共同 | 都在花 test-time compute | 都依赖某种正确性信号 |

## 附录 E：推荐项目方向（课上点名）

1. 用公开 10k-sample 数据 **缩小 gen–ver gap**  
2. 多弱 verifier 集成（接 L03）  
3. 自适应采样数随难度  
4. Noisy simulator 当 reward 的域  
5. 「易判错、难判对」的非对称过滤  

## 附录 F：术语英中对照

| English | 中文 |
|---------|------|
| Repeated sampling | 重复采样 |
| Coverage | 覆盖率 |
| Exponentiated power law | 指数化幂律 |
| Long tail of hard problems | 难题长尾 |
| Generation–verification gap | 生成–验证差距 |
| Outcome / Process reward model | 结果 / 过程奖励模型 |
| Compute-optimal scaling | 算力最优缩放 |
| Sequential revision | 串行修订 |
| Fusion | 融合生成 |
| Inference-time architecture search | 推理时架构搜索 |
