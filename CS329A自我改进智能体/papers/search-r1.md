# 论文卡：Search-R1

| 字段 | 内容 |
|------|------|
| **标题** | Search-R1: Training LLMs to Reason and Leverage Search Engines with Reinforcement Learning |
| **作者** | Jin, Zeng, Yue, Yoon, Arık, Wang, Zamani, Han |
| **出处** | arXiv:2503.09516 (2025) |
| **链接** | https://arxiv.org/abs/2503.09516 |
| **课程** | CS329A L08（课上提及、未细讲；对比 Search-o1） |

## 问题

仅靠 prompt 让模型与搜索引擎多轮交互往往次优——能否用 **RL** 直接优化「推理 ↔ 搜索」交错轨迹？

## 机制（怎么做）

1. 将搜索引擎视为环境；rollout 交错生成与检索。  
2. 兼容 **PPO / GRPO**；对 **retrieved tokens 做 loss mask**，只更新模型自己生成的 token。  
3. 结构化多轮：思考 / `<search>` 查询 / 检索块 / `<answer>`。  
4. **Outcome-based reward**（答案正确性），避免复杂过程奖。

## 关键实验 / 数字

- 七个 QA 数据集设定下，相对多种 RAG 基线：Qwen2.5-7B / 3B 约 **+24% / +20%**（论文摘要）。  
- 提供 RL 算法选择、模型选择、响应长度等实证观察。  
- 开源实现基于 veRL：https://github.com/PeterGriffinJin/Search-R1

## 局限

- 训练成本高于 Search-o1 式 prompting。  
- Reward 设计、搜索动作频率、稳定性仍敏感（后续有 Search-R1++ 等跟进研究）。

## 工程 takeaway

- 若 Deep Research 策略「教不会」，把搜索当 tool 用 **RL + token mask** 固化。  
- 与 L06（GRPO）直接相通：同一套优化器，环境换成检索。

## 一句话

**用 RL 把「何时搜、搜什么」学进权重；Search-o1 的流程版升级为可训练策略。**
