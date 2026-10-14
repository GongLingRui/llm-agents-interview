# 论文卡：The AI Scientist

| 字段 | 内容 |
|------|------|
| **标题** | The AI Scientist: Towards Fully Automated Open-Ended Scientific Discovery |
| **作者** | Lu et al. (Sakana / UBC / …) |
| **出处** | arXiv:2408.06292 (2024) |
| **链接** | https://arxiv.org/abs/2408.06292 |
| **课程** | CS329A L07 |

## 问题

能否让 frontier LLM **端到端**完成科研：出想法、写代码、跑实验、画图、写论文、甚至模拟审稿，并在开放式循环中积累知识档案？

## 机制（怎么做）

1. **Idea generation**：条件于 archive；CoT + self-reflection；自打 interestingness / novelty / feasibility；文献 API 滤重。  
2. **Experiment iteration**：Aider 改 seed 模板；失败回灌修复；多轮实验 + 笔记 + 绘图。  
3. **Write-up**：按会议模板逐节写；只引用真实实验产物；Semantic Scholar 补引用。  
4. **LLM Reviewer**：按 NeurIPS 指南打分；可作筛选 / 反馈给下一代想法。

## 关键实验 / 数字

- 约 **$10–15 / paper**；约 50 ideas 量级、小 GPU 模板约半天级。  
- LLM reviewer 在 ICLR 2022 OpenReview 子集上接近人类审稿一致性量级（文中 ~65–70% balanced accuracy 等设定）。  
- Claude Sonnet 3.5 论文质量相对最好；开源模型常漏章节 / 调不好工具。

## 局限与风险

- 实验深度不足、对照不严；幻觉图表/结果；LaTeX / 引用错误。  
- **代码安全**：自重启、占满磁盘、绕过时间限制、任意依赖。  
- 伦理：刷稿、有害科学探索、审稿自动化偏见。

## 工程 takeaway

- 开放式科研 agent 的瓶颈常在 **执行沙箱 + 结果诚实约束 + 评价校准**，不只是「会不会写英文」。  
- 小而可复现的 seed template 是实用起点。

## 一句话

**把科学共同体缩成：LLM 变异想法 + 可运行实验 + 模拟 peer review 的档案循环。**
