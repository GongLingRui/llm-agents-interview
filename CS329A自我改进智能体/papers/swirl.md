# SWiRL — Step-Wise Reinforcement Learning

| 字段 | 内容 |
|------|------|
| 全称 | Synthetic Data Generation & Multi-Step RL for Reasoning & Tool Use（SWiRL / Step-Wise RL） |
| 作者 | Goldie, Mirhoseini, Zhou, Cai, Manning 等 |
| 链接 | https://arxiv.org/abs/2504.04736 |
| 课程 | CS329A L05（主讲） |
| 笔记 | [L05](../notes/L05-multi-step-planning.md) |

## 一句话

离线生成多步工具轨迹，按步拆成前缀样本，用 **process reward** 做逐步 RL；训练时不 live 调工具，并能跨任务/跨工具泛化。

## 问题

经典 RLHF/RLAIF/RLEF 偏单步终局奖励；多步工具场景错误复合，且 live tool 拖垮/拖慢 RL。

## 机制

**Stage 1 合成**：逐步提示模型 CoT + 工具或最终答案；执行工具写入下一状态；轨迹拆成逐步子轨迹。  
**过滤**：process（Judge 逐步合理）/ outcome（终局对）/ both / random。  
**Stage 2 RL**：给定离线前缀，优化下一步动作的期望 process reward；**不重新执行工具**。  
**Inference**：真正跑 search/计算器等，用标签决定继续或 `<answer>`。

## 关键数字

- 相对提升约：GSM8K 21.5%、HotPotQA 12.3%、CofCA 14.8%、MuSiQue 11.1%、BeerQA 15.3%（论文摘要）。  
- 课上：HotPotQA 基线 ~65 → 本域 RL ~73；GSM8K 训 → HotPotQA ~71；反向迁移亦成立。  
- **Process-only 过滤对 RL 最佳**；SFT 更吃 process+outcome。  
- 合成量 100→10k：OOD 持续涨；逐步 process 分训后升高（含 OOD）。

## 工程 takeaway

- 工具执行放数据阶段并行离线；RL 环只吃文本前缀 + 标量分。  
- Judge 评「动作是否合理」而非必须看见工具返回值。  
- 勿把「只留全对轨迹」默认套到多步 RL。  
- 用跨工具迁移测试是否学到「多步习惯」而非单一 schema。

## 与课程关系

L03 过程奖励落地；L04 工具反馈；L06 的「逐步 RL」预演；强调泛化来自过程正确性提升。
