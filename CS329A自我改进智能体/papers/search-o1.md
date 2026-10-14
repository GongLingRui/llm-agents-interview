# 论文卡：Search-o1

| 字段 | 内容 |
|------|------|
| **标题** | Search-o1: Agentic Search-Enhanced Large Reasoning Models |
| **作者** | Li et al. |
| **出处** | arXiv:2501.05366 (2025) |
| **链接** | https://arxiv.org/abs/2501.05366 |
| **课程** | CS329A L08（对接 Homework 3） |

## 问题

o1 类 LRM 长推理强，但知识不足会导致 **uncertainty tokens** 与错误级联——如何把 **按需检索** 干净地嵌进思维链？

## 机制（怎么做）

1. **Agentic RAG**：推理中自主发出 search query（可多轮），而非只在开头检索一次。  
2. **Reason-in-Documents**：独立模块分析检索文，基于 query + 既往推理抽出精炼知识，再注入主链，避免长文噪声打断连贯性。  
3. 实现：QwQ-32B-Preview + Bing top-10 + 页面阅读 API。

## 关键实验 / 数字

- GPQA diamond overall pass@1：**Search-o1 63.6** > RAgent 61.6 > QwQ 直接 58.1。  
- 扩展集：整体 / 物理 / 生物可对标或超过人类专家跨域分数；化学仍有 gap。  
- 随检索文档数增加，Search-o1 继续涨；标准 RAG / 直接推理常平台期。  
- Multi-hop QA 上相对标准 RAG 提升显著；单跳提升有限。

## 局限

- 依赖检索质量与长上下文；化学等符号敏感域仍难。  
- 主要为推理时流程，不通过 RL 学搜索策略（对比 Search-R1）。

## 工程 takeaway

- Deep Research 最小闭环：**不确定就搜 → 先精炼再写入 CoT**。  
- 用 uncertainty 词频可做诊断与触发分析。

## 一句话

**给长推理模型装上「按需搜索 + 文内精读」，补知识缺口而不污染思维链。**
