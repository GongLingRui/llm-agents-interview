# Shrinking the Generation–Verification Gap with Weak Verifiers（Weaver）

## Metadata

| 字段 | 内容 |
|------|------|
| 标题 | Shrinking the Generation-Verification Gap with Weak Verifiers |
| 作者 | Saad-Falcon, Buchanan, Chen et al.（Stanford 等） |
| 年份 | 2025（NeurIPS 2025） |
| arXiv | [2506.18203](https://arxiv.org/abs/2506.18203) |
| 课程讲次 | L03 Robust Verification |
| 一句话定位 | 不训新大 verifier：过滤 + 弱监督加权集成多个弱 RM/Judge，缩小 Gen-Ver gap |

## TL;DR

**Weaver** 把多个不完美的 verifier（奖励模型、LLM judge 等）集成为更强选择策略。关键不是等权平均，而是 **归一化、过滤低质 verifier、用弱监督估计权重**，在 repeated sampling 下抬高选中正确解的概率。可用极少标签；还可把集成分数蒸馏成小 cross-encoder 以省推理算力。

## Problem

- Pass@K 高但 Success Rate 低 → **generation–verification gap**。  
- 人类/形式化 oracle 难规模化；单 RM/Judge 噪声大、假阳性高。  
- 朴素 ensemble 假设等质量，差 verifier 会拖后腿；学权重又常要大量标签。  
- 多 verifier 推理昂贵。

## Method

1. 对每题采 K 个响应，跑 m 个 verifier 得分数。  
2. **Score**：归一化不同输出格式（logit / 0-1 / Likert）。  
3. **Filter**：用少量开发集去掉过差 verifier。  
4. **Weight**：弱监督潜变量模型，在「条件独立捕捉正确性侧面」假设下估准确率并加权（动机来自 Snorkel 等）。  
5. **Select**：\(\arg\max\) 聚合分。  
6. **蒸馏**：用 Weaver 伪标签训 ~400M cross-encoder，砍验证 FLOPs。

## Key Results

- 相对 majority voting / 朴素平均有大幅提升（课上/论文：双位数百分点级常见）。  
- 可把较弱 generator + 中小 verifier 池推到接近更强推理模型的档位（课上：8B 池约 70% 平均等）。  
- 监督加权 > 弱监督 > 朴素集成 > MAV/多数票（具体顺序依设定）。  
- 蒸馏保留 Weaver 约 **98%+** 精度，验证算力可降 **~99.97%** 量级。

## Course Connection

- L02 提出 gap，L03 收官作：**用推理算力集成弱验证器** 而非只训单一强 PRM。  
- 「Weak」= 承认不完美，不是故意用烂模型。  
- 固定算力下「多种生成 vs 多种验证」分配是课上开放研究题。

## Engineering

- 先建 verifier 池（多 RM + 多 prompt judge），再过滤。  
- 留 1% 级标签估任务难度/做过滤与校准。  
- 监控 verifier 相关性：全同意则集成增益有限。  
- 生产用蒸馏小模型扛 QPS；离线可用全量 ensemble。  
- 与 best-of-N 联用：N 增大时更依赖集成降噪。

## Limitations

- 独立假设过强时理论保证变弱。  
- 全员弱且同质偏差（如都偏爱长答案）时无法奇迹翻盘。  
- 仍需要一定开发标签做过滤；完全零标签更难。  
- 多 verifier 延迟高，必须蒸馏或级联（先粗后精）。
