# Large Language Monkeys: Scaling Inference Compute with Repeated Sampling

## Metadata（作者/年份/arxiv）

| 项 | 内容 |
|----|------|
| 标题 | Large Language Monkeys: Scaling Inference Compute with Repeated Sampling |
| 作者 | Bradley Brown*, Jordan Juravsky*, Ryan Ehrlich*, Ronald Clark, Quoc V. Le, Christopher Ré, **Azalia Mirhoseini**（*共同一作） |
| 单位 | Stanford / Oxford / Google DeepMind |
| 年份 | 2024 |
| arXiv | [2407.21787](https://arxiv.org/abs/2407.21787) |
| 资源 | Code: ScalingIntelligence/large_language_monkeys；Data: HuggingFace `ScalingIntelligence/monkey_business` |
| 课次 | CS329A L01 预告 + **L02 主讲** |

## TL;DR

固定模型参数，仅靠 **正温度重复采样** 拉高 **coverage**；在可自动验证域，coverage 直接变成成绩，弱模型可打过强模型单次；coverage–样本数常呈 **exponentiated power law**；无自动 verifier 时 majority/RM **早早封顶**，暴露 generation–verification gap。

## Problem

- 训练算力缩放已被充分研究；**推理算力**常被限制为「每题一次尝试」。  
- 已有 AlphaCode 等暗示百万级采样有用，但缺少跨任务、跨模型、跨预算的系统刻画。  
- 需要同时回答：**(1) Coverage 能否随采样预算持续涨？(2) 能否从样本堆里精确挑出正确答案（Precision）？**

## Method（机制优先）

### 核心流程

```text
1. 对问题 x，从 LLM 以 T>0 独立采样 K 个候选
2. 用域相关 verifier 筛选 / 排序
3. 输出通过验证（或聚合后）的最终答案
```

### 任务与 Verifier

| 任务 | Verifier |
|------|----------|
| GSM8K / MATH | 最终数字答案匹配（coverage = oracle） |
| MiniF2F-MATH | Lean4 proof checker |
| CodeContests | 隐藏 I/O 测试 |
| SWE-bench Lite | 仓库单测；一次 sample = 一整段 agent trajectory（Moatless Tools） |

### 指标

- **Coverage / pass@k**：至少一解正确的题目比例；用 Chen et al. 无偏估计。  
- 有自动 verifier → coverage ≈ 可部署成功率。  
- 无自动 verifier → 另测 majority voting、reward model selection，对比 coverage 上界。

### Scaling law 拟合

对 coverage 与样本数关系拟合 **exponentiated power law**，用于预测「达到目标 coverage 需要多少 K」。

## Key Results

1. **跨五任务 coverage 平滑上升**（到 10k；SWE 到 250）。弱模型多次采样超过 GPT-4o 单次。  
2. **SWE-bench Lite**：DeepSeek-Coder-V2-Instruct **15.9% → 56%**（250 samples），超当时单次 SOTA **43%**。  
3. **跨规模**：70M–70B、Llama/Gemma/Pythia、base/instruct 大多有效；Gemma-2B CodeContests **0.02% → 7.1%@10k**。Pythia 在 CodeContests 上甚至 10k 仍接近 0（数据域不匹配）。  
4. **成本**：同 FLOPs 下，MATH 等常偏 **小模型多样本**；CodeContests 常偏 **大模型**。API 上 DeepSeek×5 可比 GPT/Claude×1 更强且约 **3×+ 便宜**。  
5. **Verifier 瓶颈**：MATH 上 coverage 可到 ~98%@10k，majority/RM 停在 ~**41%**；样本越多，**gap 越大**。

## Course Connection（这门课怎么讲的）

- L01 用 **infinite monkey theorem** 引入；展示 1→10k、弱模型超 GPT-4o。  
- L02 展开：SWE coverage、power law、长尾难题条件、majority plateau、KernelBench/CUDA 可验证域、HF 数据可做课项目。  
- 直接铺垫 L03：**如何缩小 generation–verification gap**。  
- 与 o1 对照：Monkeys 抬 **coverage**；reasoning RL 抬 **pass@1**。

## Engineering Takeaways

- 可验证域：把 K 当成一等公民超参；并行采样控延迟、用 $ / FLOPs 记账。  
- 先测 coverage 上界，再决定是否投资更好模型或更好 verifier。  
- 难题不要依赖多数票；稀有正确解需要精确 verifier。  
- 同预算下做 **模型大小 × 样本数** 网格搜索，最优随任务变。  
- Agent 设定里「一次 attempt」= 完整多轮轨迹，成本按轨迹计。

## Limitations

- 强依赖 verifier 质量（单测假绿、标注错误）。  
- 开放域 precision 未解决；RM/投票不随 K 扩展。  
- 极大 K 的系统优化（批处理、专用推理栈）讨论有限。  
- Coverage 乐观；产品必须交 **一个** 答案。  
- 部分小模型在某些域（如 Pythia-CodeContests）重复采样也无效。
