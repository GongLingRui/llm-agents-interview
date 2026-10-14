# ReAct: Synergizing Reasoning and Acting in Language Models

## Metadata

| 字段 | 内容 |
|------|------|
| 标题 | ReAct: Synergizing Reasoning and Acting in Language Models |
| 作者 | Yao et al.（Princeton / Google） |
| 年份 | 2022 |
| arXiv | [2210.03629](https://arxiv.org/abs/2210.03629) |
| 课程讲次 | L04 Feedback + Tools/Code |
| 一句话定位 | 用提示让 LLM 交错生成 Thought 与 Act，观察环境反馈，协同推理与行动 |

## TL;DR

**ReAct** 把 CoT 式推理轨迹与面向任务的动作交错生成：推理用于规划、跟踪与改计划；动作用于查询知识库或操作环境。在 HotpotQA、FEVER、ALFWorld、WebShop 上优于「只推理」或「只行动」基线，轨迹更可解释；与 CoT-SC 组合可进一步融合内外知识。

## Problem

- CoT 封闭在模型内部，易幻觉、难更新事实。  
- 纯行动 / 工具 agent 缺显式高层推理与工作记忆。  
- 需要一种简单、通用、可 few-shot 的方式把两者协同起来。

## Method

1. 扩展动作空间：环境动作 ∪ **语言 Thought**（Thought 不改环境，只改上下文）。  
2. Few-shot 轨迹示范：Thought / Act / Obs 交错。  
3. 知识任务：密集交错 + 简单 Wikipedia API（search/lookup/finish）。  
4. 决策任务：Thought 可稀疏出现，由模型决定何时想、何时做。  
5. 可选：与 CoT-SC 回退/组合；可进一步 finetune。

## Key Results

- QA/事实核查：ReAct > Act-only；与 CoT 互有胜负；**ReAct↔CoT-SC 组合**整体最佳。  
- ALFWorld / WebShop：1–2 shot ReAct 可大幅超过用大量数据训练的 IL/RL（论文：成功率绝对提升约 **34%** / **10%**）。  
- 课上 WebShop：ReAct ~**66.6%** vs 人类专家 ~**82.1%**，仍有差距。  
- 错误分析：CoT 失败多为幻觉；ReAct 因 grounding 更可信。

## Course Connection

- L04 开篇：反馈来自 **环境观察**。  
- 是现代 tool-calling / agent workflow 的概念祖型；今日 thinking 模型常已内化类似交织。  
- 为 L05 规划、搜索式 agent 打下「语言空间策略」基础。  
- 与 L03 可组合：对多条 ReAct 轨迹再验证。

## Engineering

- 显式合法动作表，降低非法调用。  
- Observation 原样回灌；控制步数与上下文长度。  
- 检索矛盾时加投票/反思/护栏。  
- 大动作空间：示范不够时考虑检索示范、分层策略或 RL。  
- 记录完整轨迹便于人工审计与后续 SFT。

## Limitations

- 依赖提示质量与示范覆盖；大动作空间难 few-shot。  
- 多步导致延迟与费用上升。  
- 环境噪声可导致循环；需回退与终止条件。  
- 原版冻结模型 + 提示，复杂域仍需训练/外环优化。
