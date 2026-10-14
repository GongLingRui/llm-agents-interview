# DeepSeekMath — GRPO

| 字段 | 内容 |
|------|------|
| 全称 | DeepSeekMath: Pushing the Limits of Mathematical Reasoning in Open Language Models |
| 作者 | Shao, Wang, Zhu, Guo 等（DeepSeek） |
| 链接 | https://arxiv.org/abs/2402.03300 |
| 课程 | CS329A L06（主讲） |
| 笔记 | [L06](../notes/L06-train-time-rl.md) |

## 一句话

先 **code 基座 + 大规模精选数学网页** 打底，再用 **GRPO**（组相对优势、去掉 critic）做省内存的数学 RL，7B 冲上竞赛 MATH ~51.7%。

## 问题

开源模型数学远落后闭源；标准 PPO 要策略×2 + critic + RM，大规模难扛；仅堆 arXiv 不够。

## 机制

### 数据与预训练

- 从 DeepSeek-Coder 继续训 ~120B 数学相关 CC token。  
- 课上强调：细抠 Common Crawl / OpenWebMath 式管线 **优于** 盲目 arXiv；code→math 迁移关键。

### GRPO

对同题采样一组输出，奖励组内标准化：

```text
A_i ≈ (r_i - mean(r)) / std(r)
```

用组基线替代 value model；保留相对比较信号。  
需 **组内奖励方差**；全对/全错无学习。

## 关键数字

- MATH：Instruct **46.8% → GRPO 51.7%**；64 样本自洽约 **60.9%**。  
- GSM8K：约 **82.9% → 88.2%**（论文）。  
- 课上：提升更体现在 **majority@K**，未必抬 **pass@K**（更一致 ≠ 突然会新题）。  
- 动机对比：小模型 + train-time 可大幅超越「更大但无此回路」的模型在 AIME 叙事中的表现。

## 工程 takeaway

- RL 前先问：域数据与 code/math priming 够不够。  
- 实现 GRPO 时 log 组内 std；动态跳过零方差组（DAPO 进一步系统化）。  
- 难度配比与 KL：防止全难无信号或冲掉旧能力。  
- 同时跟踪 majority@K 与 pass@K。

## 与课程关系

给出「相对 PPO 简化」的标准答案；DAPO 在其不稳定处打补丁；与 STaR 比是真正的 online 组采样 RL。
