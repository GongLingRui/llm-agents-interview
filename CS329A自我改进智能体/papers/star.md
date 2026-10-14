# STaR — Bootstrapping Reasoning With Reasoning

| 字段 | 内容 |
|------|------|
| 全称 | STaR: Bootstrapping Reasoning With Reasoning |
| 作者 | Zelikman et al.（Stanford 相关） |
| 链接 | https://arxiv.org/pdf/2203.14465 · https://arxiv.org/abs/2203.14465 |
| 课程 | CS329A L06（主讲） |
| 笔记 | [L06](../notes/L06-train-time-rl.md) |

## 一句话

用小种子 rationale few-shot 生成推理；保留答对的；答错则 **给答案倒推 rationale**，迭代微调，自举推理数据。

## 问题

逐步推理标注稀缺且贵；纯 few-shot 弱于大规模微调；需要在无海量人工 CoT 时让模型学会「写出工作步骤」。

## 机制

```text
few-shot 生成 (rationale, answer)
  ├─ 正确 → 加入训练集
  └─ 错误 → hint 正确答案 → rationalization → 加入训练集（训练时藏 hint）
微调 → 外环迭代
```

假设：终局正确近似 rationale 质量；模型能凭答案反推步骤；基座对分布有非零可解率。  
非正式地像过滤式 off-policy RL，**不是**完整 PPO。

## 关键数字 / 观察

- GPT-J ~6B；GSM8K / CommonsenseQA / 算术。  
- 样本效率高：可用更少数据接近直接 SFT 精度（课上约 86% 数据 → ~72.5% 量级设定）。  
- CommonsenseQA：人类觉得 rationale 合理。  
- GSM8K 上 rationalization 增益有限（题过易 / CoT 已够）。  
- 多轮易 plateau；hint 链无过程过滤 → 可能学到错误推理。

## 变体

- **V-STaR**：生成器+验证器共训  
- **Quiet-STaR**：隐空间思考  

## 工程 takeaway

- 适合「有标准答案、几乎无 RL 集群」的冷启动。  
- 务必抽查 rationalization 质量或加 PRM。  
- 与蒸馏大模型对比：有教师时蒸馏往往更强；STaR 教的是自举环。  
- 监控可解种子比例——全失败则环转不动。

## 与课程关系

Train-time scaling 的最小完整例子；上承 L03 验证，下接 GRPO/DAPO「真正的 online RL」。
