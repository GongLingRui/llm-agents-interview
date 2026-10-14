# How Do Large Language Monkeys Get Their Power (Laws)?

## Metadata（作者/年份/arxiv）

| 项 | 内容 |
|----|------|
| 标题 | How Do Large Language Monkeys Get Their Power (Laws)? |
| 作者 | Rylan Schaeffer, Joshua Kazdan, John Hughes, Jordan Juravsky, Sara Price, Aengus Lynch, Erik Jones, Robert Kirk, **Azalia Mirhoseini**, Sanmi Koyejo 等 |
| 年份 | 2025 |
| arXiv | [2502.17578](https://arxiv.org/abs/2502.17578) |
| 课次 | CS329A **L02**（解释 Monkeys 的 scaling 曲线从何而来） |

## TL;DR

Monkeys / Best-of-N jailbreak 等现象里，**聚合**成功的 \(-\log(\mathrm{avg\ pass}@k)\) 对尝试次数 \(k\) 呈 **幂律**；但 **每一题** 上失败率应按 \((1-p_i)^k\) **指数**下降。两者不矛盾：只要各题单次成功概率 \(p_i=\mathrm{pass}_i@1\) 的分布有 **重左尾（极难题）**，指数族的混合就会在聚合层面长出幂律。该视角还能解释偏离幂律的案例，并用分布拟合 **更便宜地预报** 缩放指数。

## Problem

- Brown et al. (Monkeys) 与 Hughes et al.（多模态 jailbreak Best-of-N）都观察到：

\[
-\log\Big(\tfrac{1}{P}\sum_i \mathrm{pass}_i@k\Big) \approx a\,k^{-b}
\]

- 朴素概率却预测：**单题**

\[
\mathrm{pass}_i@k = 1-(1-p_i)^k \quad\Rightarrow\quad -\log(\mathrm{pass}_i@k)\approx (1-p_i)^k
\]

即单题应是 **指数**，不是多项式。  
- 谜题：幂律从哪来？如何预报？为何有的模型/设定不呈幂律？

## Method（机制优先）

### 1. 确认单题指数

在 MATH（Pythia 等）与 HarmBench jailbreak（Claude/GPT/Gemini/Llama…）上，逐题画 \(-\log(\mathrm{pass}_i@k)\)，实证为指数衰减；同时聚合曲线仍可呈幂律。

### 2. 分布桥接

\[
\mathrm{pass}_\mathcal{D}@k
= \mathbb{E}_{p\sim\mathcal{D}}[1-(1-p)^k]
= 1-\int_0^1 (1-p)^k\,p_\mathcal{D}(p)\,dp
\]

聚合行为完全由 **单次成功概率分布** \(\mathcal{D}\) 决定。

### 3. 何种 \(\mathcal{D}\) 产生幂律？

对 Uniform / Beta(\(\alpha,\beta\)) / Kumaraswamy / 等推导：例如 Beta 下

\[
-\log(\mathrm{pass}_{\mathrm{Beta}(\alpha,\beta)}@k) \propto k^{-\alpha}
\]

关键是 \(p\) 在 0 附近的 **左尾形状**（大量极小 \(p_i\)）。

### 4. 实证拟合

- 直方图显示 \(\mathrm{pass}_i@1\) 常具 power-law-like 左尾；可用 scaled Beta-Binomial 等拟合。  
- **反例**：某设定下 Llama-3-8B-IT jailbreak **缺乏** 该左尾 → 聚合也不呈幂律（解释先前「例外」）。

### 5. 预报

先估 \(p_i\) 分布，再解析/数值预报聚合缩放指数 → 相对直接狂采超大 \(k\) 拟合，可 **约低 1 数量级相对误差**，或等价少 **~2–4 数量级** 推理算力。

## Key Results

1. **单题指数 + 聚合幂律** 可同时成立。  
2. **充分机制**：重左尾的 \(p_i\) 分布（课上称 **long tail of hard problems**）。  
3. 左尾参数（如 Beta 的 \(\alpha\)）决定幂律指数。  
4. 解释 Monkeys 与 jailbreak 两篇的幂律，以及 **不出现幂律** 的案例。  
5. 提供 **低成本预报** scaling 的实用方法，利于评估设计。

## Course Connection（这门课怎么讲的）

- L02 在展示 Monkeys 拟合曲线后提问：若单题是 \(1-(1-p)^k\)，为何全集像 power law？  
- 课上回答：**需要难题长尾**；简单题堆在高 pass@1，难题拖出长尾——与论文定理/实证一致。  
- 把「科学上可预测的 inference scaling」从经验曲线提升到 **分布条件**。  
- 工程含义：若你的评测集没有长尾难题，不要指望同一条幂律外推。

## Engineering Takeaways

- 报告 scaling 时同时给：**(a) 聚合曲线 (b) \(p_i\) 直方图 (c) 若干单题曲线**。  
- 做预算规划：先中等 \(N\) 估 \(p_i\) 分布，再预报更大 \(K\)，避免一上来 10k×全量。  
- 构建评测集时有意保留难题长尾，才能反映真实 inference scaling。  
- 若分布无左尾，加大 \(K\) 的收益形态会不同——换策略或换数据。  
- 与 verifier 问题正交：即使 coverage 幂律漂亮，没有 precision 仍无法产品化。

## Limitations

- 理论依赖 i.i.d. 尝试；有温度退火、缓存、依赖采样时需修正。  
- 分布族选择影响指数解释；有限题量下左尾估计噪声大。  
- 主要解释 **coverage / any-success** 指标，不直接解决选样 / RM plateau。  
- Jailbreak 与数学以外的任务需再验证。  
- 预报方法仍需每题一定量初始采样。
