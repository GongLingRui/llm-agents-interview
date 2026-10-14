# DeepScholar-Bench: Generative Research Synthesis

- **链接**：https://arxiv.org/abs/2508.20033  
- **关联讲次**：L17；L08 Deep Research；HW3  
- **机构**：Stanford

## 问题

Deep research 产品写 related work 时，检索、综合、引用是否可信？需要 live、抗污染基准。

## 机制

- 任务：为近期 arXiv 论文写 Related Work（~22 域，PhD 难度）  
- 每月新论文重跑；只用训练截止后论文  
- 三轴：knowledge synthesis / retrieval quality / verifiability（人类 agreement ~70–80%）

## 课堂强调数字

| 点 | 值 |
|----|-----|
| 现有系统综合上限 | **< ~19%** |
| Document importance | **< ~12.5%** |
| Oracle 源时 key fact coverage | 仍约 **~50%** |
| 文笔 vs 事实 | OpenAI DR 连贯但漏关键事实；可验证性未必最高 |

## 失败模式

漏奠基论文；只会判相关不会判重要；抽事实差；综合质量与可验证性难两全。

## 工程 Takeaway

- 分项指标，禁止只看「读起来顺」。  
- Live 基准防污染。  
- 适合作为课程项目的巨大 headroom 题。

## 一句话

研究综合仍远未解决：好英文掩盖漏事实与弱引用，整体 <19%。
