# 论文卡：AlphaEvolve

| 字段 | 内容 |
|------|------|
| **标题** | AlphaEvolve: A Gemini-powered coding agent for designing advanced algorithms |
| **作者** | Novikov, Vũ, Eisenberger, … / Google DeepMind |
| **出处** | DeepMind white paper (2025) |
| **链接** | [PDF](https://storage.googleapis.com/deepmind-media/DeepMind.com/Blog/alphaevolve-a-gemini-powered-coding-agent-for-designing-advanced-algorithms/AlphaEvolve.pdf) |
| **课程** | CS329A L07 |

## 问题

在**可自动评测**的科学/工程问题上，如何用 SOTA LLM + 进化搜索，发现超越人类已有构造的**算法与启发式**（而不只是答竞赛题）？

## 机制（怎么做）

- **Program database** 存候选程序与分数。  
- **Prompt sampler** 拼 parent + inspirations + 评价反馈 + 可选文献 / meta-prompt。  
- **LLM ensemble** 产出 **diff**，应用到整文件（`EVOLVE-BLOCK`）。  
- **Evaluators pool** 跑用户 `evaluate()`（可多指标、可长时间、可并行）。  
- 相对 FunSearch：整文件、多语言、少采样、富上下文、多目标。

## 关键实验 / 数字

- 矩阵乘：14 个目标超 SOTA；**4×4 复数矩阵 48 次乘法**（改进该设定下长期沿用的 Strassen 系结果）。  
- 50+ 数学构造题：~75% 追平、~20% 刷新（如 11D kissing number 593）。  
- Google 内部：集群调度、训练 matmul kernel、TPU 电路、Attention 运行时等。

## 局限

- 必须有可靠自动评价；评价可被 hack。  
- 仍需算力与人工设定多目标权衡。  
- 发现需独立验证才算「定理级」贡献。

## 工程 takeaway

- 把「解」表示成**可演化代码**，用执行反馈接地，比纯自然语言假设生成更稳。  
- 档案 + 富反馈 prompt 是样本效率的关键。

## 一句话

**LLM 当变异算子、评测当自然选择：进化出新算法与生产级启发式。**
