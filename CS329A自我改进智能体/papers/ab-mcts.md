# AB-MCTS — Wider or Deeper?

| 字段 | 内容 |
|------|------|
| 全称 | Wider or Deeper? Scaling LLM Inference-Time Compute with Adaptive Branching Tree Search |
| 作者 | Inoue, Misaki, Imajuku, Kuroki, Nakamura, Akiba（Sakana AI） |
| 链接 | https://arxiv.org/abs/2503.04412 |
| 课程 | CS329A L05（必读；口播未展开） |
| 笔记 | [L05](../notes/L05-multi-step-planning.md) |
| 代码 | https://github.com/SakanaAI/treequest |

## 一句话

在有外部反馈时，于每个节点 **动态选择 widen（新候选）还是 deepen（ refining 旧候选）**，统一 repeated sampling 与多轮改进。

## 问题

- Repeated sampling：只宽不深，不利用测试/执行反馈 refine。  
- 固定宽度 MCTS：分支数超参写死，限制 LLM 输出多样性。  
- 纯顺序 refine：多样性不足。

## 机制

Adaptive Branching MCTS：节点上用反馈与贝叶斯式后验，决定扩展新孩子或深入已有孩子。  
变体 AB-MCTS-M / AB-MCTS-A（不同决策原理与权衡）。  
评分可为测试通过率等；隐藏测试时用公开/代理评分引导搜索。

## 关键数字 / 实验

同 inference 调用预算下，在复杂编码、ML engineering、ARC-AGI 等上优于 repeated sampling 与标准固定宽 MCTS（论文宣称；用 GPT-4o、DeepSeek-V3 等）。

## 工程 takeaway

- 先确认存在 **可量化反馈**（单测、评测器）；无反馈则退回 best-of-N。  
- 不要把 branching factor 当成死超参；跟预算与后验走。  
- 与 LATS：LATS 偏 **环境动作序列**；AB-MCTS 偏 **答案候选树的宽深**。  
- 早期探索宜宽、后期宜深——与 SPRINT 观察到的并行宽度变化同直觉。

## 与课程关系

直接回答 L05 决策表中「wider vs deeper」；是 test-time scaling 在「多答案生成」族上的进阶。
