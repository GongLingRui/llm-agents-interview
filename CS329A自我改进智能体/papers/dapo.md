# DAPO — Decoupled Clip & Dynamic Sampling Policy Optimization

| 字段 | 内容 |
|------|------|
| 全称 | DAPO: An Open-Source LLM Reinforcement Learning System at Scale |
| 作者 | ByteDance Seed / 清华 AIR 等 |
| 链接 | https://arxiv.org/abs/2503.14476 |
| 代码 | https://github.com/volcengine/verl · https://dapo-sia.github.io/ |
| 课程 | CS329A L06（主讲） |
| 笔记 | [L06](../notes/L06-train-time-rl.md) |

## 一句话

公开可复现的大规模长 CoT RL 配方：非对称 clip、动态采样、token-level loss、超长塑形，使 Qwen2.5-32B 在 AIME 2024 达到约 **50** 分。

## 问题

朴素 GRPO 上 32B 仅 ~30 AIME；社区难复现 R1 级 RL。常见病：熵崩塌、奖励噪声、长度失控、训练不稳。关键技巧未在前序报告写清。

## 机制（四件套）

1. **Clip-Higher（非对称 clip）**  
   允许低概率 token 更大上调 → 保探索、抗熵崩塌。

2. **Dynamic Sampling**  
   Oversample → 丢弃全对/全错组 → 只训有混合结果的组 → 非零 advantage、稳有效 batch。

3. **Token-Level Policy Gradient Loss**  
   避免「超长垃圾答案」与短好答案同权；塑形长度与熵。

4. **Overlong Reward Shaping**  
   软惩罚/过滤截断噪声链；亦可配合加长 context 策略。

## 关键数字（课上消融链 · Qwen-32B · AIME）

| 步骤 | ~分数 |
|------|-------|
| 朴素 GRPO | 30 |
| + overlong filtering | 36 |
| + asymmetric clipping | 38 |
| + soft overlong punishment | 41 |
| + token-level loss | 42 |
| + dynamic sampling | **50** |

论文：基于 Qwen2.5-32B 达 50，并称以更少 step 超过 DeepSeek-R1-Zero-Qwen-32B（47）。

## 工程 takeaway

- 监控：**entropy、mean response length、全对组占比**。  
- 无提升时检查 RM 是否饱和或被 hack。  
- Dynamic sampling 的组大小随难度调，64 仅为示例。  
- 长推理 RL「算法名」往往不如 **这四类实现细节** 重要。  
- 开源 verl + 数据：优先复现仪表盘再调损失。

## 与课程关系

回答「为何只 GRPO 不够」；收束 L06 的 SFT vs RL、train vs test、验证器依赖与开放问题（pass@K、负例学习、奖励鲁棒）。
