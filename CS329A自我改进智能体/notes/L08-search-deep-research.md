# L08 Self-Improvement with Search & Deep Research Agents

> **录音对照**：`raw-transcripts/7.self-improvmentdeepreasechagent.md`（**文件名易混，内容实际是 L08**，不是 L07）。  
> 主题：竞赛代码上的大规模搜索（AlphaCode / AlphaCode 2）+ 推理模型上的 Deep Research 式检索（Search-o1；课末点到 Search-R1）。

---

## 1. 一句话总览

正确答案往往已落在模型输出分布的搜索空间里——本讲讲如何**用海量采样 + 过滤/聚类/打分**把它捞出来（代码），以及如何在长推理中**按需搜索并把文档「读进」思维链**（Deep Research）；二者分别对接 **Homework 2（HumanEval 式）** 与 **Homework 3（agentic search）**。

---

## 2. 核心概念与定义

### 2.1 两种「Search」

| 类型 | 代表 | 在搜什么 |
|------|------|----------|
| **输出空间搜索** | AlphaCode / AlphaCode 2 | 同一题的百万级程序样本 → 筛到 ≤10 个提交 |
| **知识/工具搜索** | Search-o1 / Search-R1 | 推理中途查外部文档 / 学何时调用搜索引擎 |

课上金句：*solutions lie in the search space of the models*；难的是 **curate**（策展）出该提交 / 该采信的那一个。

### 2.2 `pass@k` vs `10@k`（必考）

- **`pass@k`**：生成 \(k\) 个样本，**全部**可拿去对隐藏测试评（coverage / 搜索能力上限）。  
- **`10@k`**：仍生成 \(k\) 个，但**最多提交 10 个**（竞赛规则）→ 还要测 **filtering + clustering + ranking** 好不好。  
- 关系：`pass@k = k@k`；`pass@k` 是 `10@k` 的上界。  
- 课上图：同样采样预算下，`pass@k` 可到 ~40%+ 时，`10@k` 可能卡在 ~30% → **selection bottleneck**。

### 2.3 Diversity（多样性）

采样更多只有在**语义/行为多样性**继续增加时才有用。AlphaCode 用：高温度、随机化 tag/rating、多语言（AC1）/ 多 fine-tune 变体（AC2）、**按行为聚类**避免交 10 个「长得不一样但测例行为一样」的解。

### 2.4 Uncertainty tokens（Search-o1）

长推理里知识缺口常表现为 **perhaps / wait / alternatively / likely** 等词（课上：GPQA 上 “perhaps” 可平均出现数十次）。这些信号可触发检索；若放任，错误会**级联**到最终答案。

### 2.5 与 Homework 的关系（课上明确）

| 作业 | 课上怎么说 |
|------|------------|
| **HW2** | HumanEval 等较简单代码；用 repeated sampling / pass@k 直觉；AlphaCode 是「更难、更长」的同类问题预览 |
| **HW3** | Deep Research / agentic search 更接近 Search-o1；鼓励自己试「只改 prompt 做摘要 vs Reason-in-Documents」 |

---

## 3. 方法 / 机制

### 3.1 AlphaCode（2022）：百万采样 → 过滤 → 聚类

**问题设定**：竞赛编程（长题面、需算法推理），远难于「补全一行」或 HumanEval 式 docstring→函数。

**流水线**：

```text
Pretrain（GitHub ~715GB，encoder-decoder 或后续 decoder-only）
  → Fine-tune CodeContests（GOLD + tempering + value conditioning…）
  → 每题采样 ~1e6 程序（半 Python / 半 C++；高温；随机化 tag & rating）
  → Filter：只保留通过题面公开样例的
  → Cluster：另训「测例生成模型」造新输入；按输出行为聚语义等价类
  → 从大到小簇各取代表 → 最多提交 10 个到 Codeforces
```

**GOLD**：one-of-many 设定下，标准 MLE 像在「召回所有写法」；竞赛指标像「精度（交对一个就行）」→ GOLD 提高对高似然、有意义 token 的权重。

**结果（课 + 论文）**：

- 10 场 Codeforces、每场 5k+ 人：平均排名约 **top 54.3%**；估计 rating 1238，优于近 6 个月参赛用户约 **28%** 之上的那一档叙述（课上：competitive with ~28%）。  
- **41B + clustering** 稳定优于 9B / 无聚类；`10@k` 随 \(k\) 从 1k→1M **log-linear** 上升（与 Monkeys / pass@k 同型）。  
- 局限：loss 不是好的 solve-rate proxy；DP/构造题弱；百万采样不实用；难问题更需**多步**而非 one-shot。

**课上 Q&A**：不同 contest 方差 ← 题分布 + **selection 阶段瓶颈**；「采到 1T 会不会翻倍」← 要看多样性是否还在涨，且**做强模型往往比死撑采样更划算**（引出 AC2）。

### 3.2 AlphaCode 2：Gemini fine-tune + 多样性策略 + Scoring model

**假设**：不再自研预训练，改 **fine-tune Gemini Pro**。

**三大改动**：

1. **Policy 家族**：在 CodeContests v2（~15k 题、~30M 解）上用 GOLD 多轮 fine-tune，**扫超参得到多个生成模型** → 采样预算均分，抬 diversity。仅 **C++**（发现质量更高）。  
2. **Massive sampling**：随机温度 + 随机 metadata；可达百万级。  
3. **Scoring model（类 reward model）**：另一 Gemini Pro，在**更高质量人工校验集**上训，给代码打 0–1 正确性分；**勿与生成模型完全同数据**以防污染（课上讨论）。

**选交流程**：

```text
采样 → 执行公开测例 / 编译过滤（去掉约 95%）→ 剩 ~5e4
  → 行为聚类 → 取最大 10 簇
  → 每簇内用 scoring model 挑最好 1 个 → 提交 ≤10
```

**结果**：

- 同平台：solve rate **25% → 43%**（约 **1.7×**）；估计约 **85th percentile**（Expert–Candidate Master）；最好两场可超 **99.5%** 选手。  
- **样本效率**：约 **100** 个样本即可追平 AlphaCode 的 **1M** 表现（~10⁴×）。  
- 仍随采样预算 log-linear；系统仍贵、大量算力浪费在烂语法样本上。

**课上讨论：任务复杂度 / 把推理嵌进模型**

- 简单题：更少采样或用「简单题」专精模型。  
- 难题：需要多样性并行探索；或 **分解 + 多步闭环 / 树搜索**（连到 L05）；训练侧可加算法提示 / CoT / STaR 式理由。  
- 95% 浪费：可用 **self-refine + 执行反馈**（HW1 思路）或 **train-time RL**（L06）换 test-time 采样。

### 3.3 Search-o1：Agentic RAG + Reason-in-Documents

**动机**：LRM（o1 / QwQ 等）长链推理强，但有知识截止与缺口；缺口表现为 uncertainty tokens，会污染整条链。

**三种对比（课 + 论文 Fig.2）**：

| 方法 | 行为 | 问题 |
|------|------|------|
| Direct reasoning | 纯参数知识 | 猜结构 → 错答案级联 |
| Agentic RAG | 不确定时发查询、插入检索文档 | 文档又长又噪，打断连贯推理 |
| **Search-o1** | Agentic 检索 + **独立 Reason-in-Documents** 精炼后再注入 | 保持主链连贯 |

**机制要点**：

- 推理中解码特殊 search token，触发检索（可多轮）。  
- Reason-in-Documents：用**同一 LRM**、单独前向，基于「当前 query + 已有推理 + 原文」抽出可无缝衔接的精炼知识，再写回主链。  
- 实现上：Bing 检索 top-10 + Jina 读页；骨干常用 **QwQ-32B-Preview**。

**结果要点**：

- GPQA diamond：Search-o1 overall **63.6** pass@1 > RAgent **61.6** > 直接 QwQ **58.1**；化学仍相对弱、物理/生物更强。  
- 扩展集上相对人类专家：物理/生物可超过「该领域专家在跨域题上的表现」叙事；课上强调看**对角线**（物理学家做物理）。  
- **文档数缩放**：Search-o1 随 top-k↑ 仍涨；Direct / 标准 RAG 往往不涨甚至乱。  
- Multi-hop QA（HotpotQA / 2Wiki / MuSiQue / Bamboogle）：Search-o1 相对标准 RAG 增益大；单跳增益有限（本来一次检索就够）。  
- 分析：推理链里 uncertainty 词显著下降。

**课上 Q&A**：如何知道搜什么？→ 抓实体 / 不可信参数知识的点；纠正先前推理依赖上下文缓冲；「只在 Agentic RAG 里加一句 summarize」≈ ACE 方向，但受**长上下文能力**限制——留给 HW3 实验。

### 3.4 Search-R1（课上点到、未细讲）

相对 Search-o1 的 **prompt / 流程工程**，Search-R1 用 **RL（PPO/GRPO）** 学「推理中穿插多次搜索」：

- 轨迹：`<think>` / `<search>` / `<info>` / `<answer>` 等结构化交错。  
- **Retrieved token masking**：不对检索全文算 policy gradient，训练更稳。  
- **Outcome reward**（答案对错）即可，不必上复杂过程奖。  
- 报告：相对同设定 RAG 基线，Qwen2.5-7B / 3B 约 **+24% / +20%** 量级。

记忆口诀：**Search-o1 = 会搜会读的 prompting 系统；Search-R1 = 用 RL 把「何时搜、搜什么」学进权重。**

---

## 4. 关键数字与实验结论

| 系统 | 数字 |
|------|------|
| AlphaCode | ~1M samples/题；Codeforces ~54.3% 排名；41B+cluster 最好 |
| 度量 | `10@k` < `pass@k`；两者均随 k log-linear |
| AlphaCode 2 | 43% vs 25% solve；~85th pct；~100 samples ≈ AC1 @ 1M |
| AC2 过滤 | ~95% 样本被扔掉 |
| Search-o1 | GPQA overall 63.6；随文档数可缩放；multi-hop 优势明显 |
| Search-R1 | RL + mask；相对 RAG +20–24%（论文设定） |

---

## 5. 课堂 Q&A 与易错点

1. **Contest 间方差大**：题分布差异 + 选择器失灵（近对但测例不过）。  
2. **采样预算幻想**：无额外多样性时，10× 采样≈白烧钱。  
3. **Scoring model 数据**：要同分布但避免与生成模型训练题完全重合。  
4. **校准**：token logprob 聚合后模型常**过度自信**；正确时相关、错误时仍可能很自信——主动研究课题。  
5. **文件名陷阱**：仓库 `7.self-improvment...` = **本讲 L08**，不是 Open-Ended Evolution。

---

## 6. 本讲论文速查

| 论文 | 卡片 |
|------|------|
| AlphaCode (Li et al., Science 2022) | [papers/alphacode.md](../papers/alphacode.md) |
| AlphaCode 2 Tech Report | [papers/alphacode2.md](../papers/alphacode2.md) |
| Search-o1 (arXiv:2501.05366) | [papers/search-o1.md](../papers/search-o1.md) |
| Search-R1 (arXiv:2503.09516) | [papers/search-r1.md](../papers/search-r1.md) |

---

## 7. 工程可落地 checklist

- [ ] 指标用 `pass@k` 还是必须模拟提交限额的 `n@k`？  
- [ ] 多样性：温度 / 多模型 / 多语言 / 行为聚类，缺哪一层？  
- [ ] 过滤信号：公开测例、编译、静态检查、学习到的 scorer？  
- [ ] Deep Research：一次 RAG 是否不够？是否要多轮 + 文内精炼？  
- [ ] 是否监测 uncertainty tokens / 幻觉率作为检索触发器？  
- [ ] 若要降采样成本：self-refine、更强 base、或 RL 化学搜索（Search-R1）？  
- [ ] HW2 先验证采样曲线；HW3 对比 RAG / Agentic RAG / Reason-in-Documents。

---

## 8. 若你只记 5 条

1. **策展 > 盲采**：百万样本必须 filter→cluster→（可选）score 才能交 10 个。  
2. **`10@k` 测的是选择器**；`pass@k` 测的是覆盖率。  
3. **AlphaCode 2**：更强基座 + 多 policy 多样性 + scoring model → 少采样大幅涨点。  
4. **Search-o1**：按需搜 + **Reason-in-Documents**，别把长文硬塞进思维链。  
5. **Search-R1** 用 RL 学搜索；本讲与 **HW2/HW3** 直接对应两套作业技能。
