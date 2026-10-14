# Absolute Zero: Reinforced Self-play Reasoning with Zero Data

- **链接**：https://arxiv.org/abs/2505.03335  
- **关联讲次**：L20；共鸣 L05 SWiRL  
- **别名**：Absolute Zero Reasoner (AZR)

## 问题

RLVR「zero」仍依赖人类策展题库；专家题会枯竭。能否让模型**自己出题又做题**？

## 机制

- 同一模型：**proposer** + **solver**  
- 代码执行器：验证任务合法 + 验证答案  
- 任务型：deduction / abduction / induction（程序–输入–输出推理）  
- Proposer 奖励：偏好中等难度（1 − success rate；全失败则 0）  
- Buffer + 参考样本促多样性与课程演化

## 关键结果

- **零外部题**仍达 coding 强结果，超过数万专家样本训练的对照  
- 复杂度与任务难度随训练上升  
- 意外：数学基准也提升；大模型收益更大

## 工程 Takeaway

- 可执行环境是 Absolute Zero 的前提。  
- 必须防 proposer 出垃圾题（语法 / 安全 / 确定性）。  
- 与开放进化、合成数据迁移叙事一致。

## 一句话

在代码环境里自玩「出题–做题」，零人类题库也能爬坡并迁移到数学。
