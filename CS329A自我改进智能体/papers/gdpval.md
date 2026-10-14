# GDPVal: Real-World Economically Valuable Tasks

- **链接**：https://arxiv.org/abs/2510.04374  
- **关联讲次**：L17  
- **机构**：OpenAI（课上介绍）

## 问题

模型在**真实有经济价值、来自十年经验专家**的工作上，相对专家胜率如何？与「任务时长翻倍」叙事是否一致？

## 机制

- 覆盖 GDP 高价值相关 ~9 sector、44 occupations、~1320 tasks（公开 gold ~220）  
- 以数字任务为主；多模态交付物；多数需 reference files  
- 专家 pairwise 偏好 → win rate（含 ties）  
- 任务偏 well-specified、偏 one-shot

## 课堂强调数字

| 点 | 值 |
|----|-----|
| 趋势形状 | 近两年偏 **线性**（非 METR 式指数） |
| Claude Opus 4.1 | 胜率约 **47.6%** |
| 可接受偏差 / 更好 / 灾难 | ~**50% / 20% / 29%** |
| 多试自修 | GPT-5 ~1.6× cost / 1.4× speed 量级改进口径 |
| 失败主因 | Instruction following；假读 reference；格式 |

## 工程 Takeaway

- 按时长、模态、职业拆开看，别只看 aggregate。  
- 欠指定时模型不会「自己长出专家上下文」。  
- 近端：人类 architect + 模型 execute 已有 ROI。

## 一句话

经济任务胜率线性爬升且分职业分化；测的是「给足上下文后输出够不够好」，不是自主找活。
