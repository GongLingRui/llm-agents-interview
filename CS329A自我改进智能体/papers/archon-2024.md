# Archon: An Architecture Search Framework for Inference-Time Techniques

## Metadata（作者/年份/arxiv）

| 项 | 内容 |
|----|------|
| 标题 | Archon: An Architecture Search Framework for Inference-Time Techniques |
| 作者 | Jon Saad-Falcon, Adrian Gamarra Lafuente, Shlok Natarajan, Nahum Maru, Hristo Todorov, Etash Guha, E. Kelly Buchanan, Mayee Chen, Neel Guha, Christopher Ré, **Azalia Mirhoseini** |
| 单位 | Stanford（及 UW 等） |
| 年份 | 2024 |
| arXiv | [2409.15254](https://arxiv.org/abs/2409.15254) |
| 课次 | CS329A **L02**（课上强调 TA 为共同作者之一） |

## TL;DR

把 repeated sampling、fusion、critic、rank、verify、unit-test 等 **推理时技巧** 当成可堆叠「层」，用 **Bayesian 架构搜索（iTAS）** 在给定 LLM 集合与 call budget 下自动拼出准确率–成本 Pareto 更优的系统；开源组合可在多项任务上平均超过当时 GPT-4o / Claude 3.5 / o1 等约 **15%**。

## Problem

- 单一技巧（只加采样、或只 fusion）无法通吃数学 / 代码 / 指令遵循。  
- 技巧之间的 **交互** 不清（多模型 vs 多样本？critic 放哪？）。  
- 设计空间巨大：选哪些模型、采样几次、如何过滤与融合——需要 **自动化搜索**，而非手搓。  
- 现有 MoA / LLM-Blender / ADAS / AFlow 等在算力利用与任务泛化上仍有局限。

## Method（机制优先）

### 框架输入 / 输出

```text
输入: 目标 benchmark、inference call budget、可用 LLMs、可用技巧集合
过程: Bayesian optimization 搜索层状架构
输出: 优化后的 Archon 架构（最终通常产出 1 个答案 → 优化 pass@1）
```

### 组件（LLM Components）

| 组件 | 机制 |
|------|------|
| **Generator** | 采样；可并行 ensemble / 多样本 |
| **Fuser** | 给定题 + K 个回答 → 合成 1（或多）个更高质回答 |
| **Ranker** | 对候选排序，保留 top-K |
| **Critic** | 为每答写优缺点，供后续 rank/fuse 使用 |
| **Verifier** | 两阶段：写理由 → 输出 Correct/Incorrect；过滤 |
| **Unit Test Generator / Evaluator** | 生成自然语言/代码测例；评估候选是否通过（可扩展到非代码推理） |

### 组合规则（缩小空间）

1. 每层仅一种组件类型  
2. Generator **只能**在第 1 层（可多个）  
3. Critic 必须在 Ranker/Fuser 之前  
4. Rank/Critic/Verify/UT 层通常单模块  
5. Fuser 可多层、可并行多个  
6. UT Generator 后必须接 Evaluator  

类比神经网络：层变换「字符串列表」状态；但 **无层间可学习权重**，可开箱部署。

### 搜索（iTAS）

- Bayesian optimization，比 greedy/random **更省配置评估次数**  
- Offline 先验：发现 gen→critic→rank→fuse 等 motif，再限缩空间  
- 目标：最大化准确率，约束 inference calls / tokens  
- 可优化 **task-specific** 或 **general-purpose**（多任务一起）

## Key Results

1. **SOTA 平均 +15.1%**（摘要）相对 frontier 与既有推理架构；课上口述相对 GPT-4o/Claude 3.5 Sonnet **+14.1%** pass@1。  
2. 同时约 **−20% inference calls**、更少 in/out tokens。  
3. **仅开源模型** 平均仍可超 SOTA LLM ~**11.2%**。  
4. **Fusion 可超过 Oracle selection**（课上左图红线 > 蓝 oracle）：合成 > 只挑最好候选。  
5. Top-k 过滤后再 fuse 通常更好；**加深** critic/fuse 层类似加深网络，多任务继续涨点。  
6. 代码任务：加重采样 + unit test 层，CodeContests Pass@1 例：**17.9% → 29.3%**（论文消融量级）。  
7. 通用 Archon 相对 task-specific 仍有强泛化。

## Course Connection（这门课怎么讲的）

- L02 后半主线：在 Monkeys / Snell 之后回答「如何 **混合** 多种 test-time 方法并控预算」。  
- 课上逐步演示 generator / fusion / critic / ranker / unit-test，再展示搜索出的复杂架构图。  
- 强调：最终仍是 **一个答案（pass@1）**，与 Monkeys 的 coverage 叙事形成对照。  
- 指向后续：更好 verifier（L03）、工具与反馈（L04）、规划（L05）。

## Engineering Takeaways

- 先实现组件库 + 硬编码合法拓扑，再上 Bayes 搜索。  
- **Fusion 性价比极高**；务必试「多候选 → 合成」而不只 best-of-n。  
- 编码 / 推理：优先加 **unit-test gen+eval**。  
- 指令遵循：critic + rank 收益大。  
- 按 call budget 做 Pareto，而不是无限堆层。  
- Ensemble 时从最强模型开始往下加；融合者常用最强模型。

## Limitations

- 搜索本身极贵（大量推理调用）；依赖先验剪枝。  
- Prompt-based 组件，未针对每技巧单独 RL 训练。  
- 组件顺序规则来自消融，未必全局最优。  
- 延迟 / 美元成本随层数上升；生产需额外 SLA 约束。  
- Benchmark 集中在指令、推理、竞赛代码；长程 agent 轨迹未充分覆盖。
