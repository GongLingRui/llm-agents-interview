# 论文卡：AlphaCode

| 字段 | 内容 |
|------|------|
| **标题** | Competition-Level Code Generation with AlphaCode |
| **作者** | Li et al., DeepMind |
| **出处** | Science 2022 / arXiv:2203.07814 |
| **链接** | https://arxiv.org/abs/2203.07814 |
| **课程** | CS329A L08 |

## 问题

LLM 能补全代码，但能否在 **Codeforces 级竞赛**上端到端解题（长题面、算法推理、隐藏测例）？

## 机制（怎么做）

1. GitHub 大规模预训练（encoder-decoder 等）。  
2. CodeContests 微调；**GOLD** + tempering 等对准「多解一题」目标。  
3. 每题 **~100 万**样本（Py/C++、高温、随机 tag/rating）。  
4. 公开测例过滤 → **行为聚类**（另模生成测输入）→ 最多交 **10** 个。

## 关键实验 / 数字

- 度量：**`10@k`**（限交 10）vs **`pass@k`**（覆盖上限）。  
- 10 场模拟赛：平均约 **top 54.3%**。  
- 更大模型 + 聚类更好；solve rate 随采样预算 **log-linear**。  
- Selection 造成 `10@k` 明显低于 `pass@k`。

## 局限

- 百万采样不实用；难域（DP/构造）弱；训练 loss ≠ solve rate。  
- 依赖测例过滤，无执行反馈的多步精炼。

## 工程 takeaway

- 竞赛设定下，**多样性采样 + 行为去重**比盲目加温度更重要。  
- 产品若有提交限额，必须单独优化 **reranker / cluster**，不能只报 pass@k。

## 一句话

**百万次采样把解「埋进」分布里，再用过滤与聚类把它挖到 10 次提交里。**
