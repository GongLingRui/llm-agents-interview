# Constitutional AI: Harmlessness from AI Feedback

## Metadata

| 字段 | 内容 |
|------|------|
| 标题 | Constitutional AI: Harmlessness from AI Feedback |
| 作者 | Bai et al.（Anthropic） |
| 年份 | 2022 |
| arXiv | [2212.08073](https://arxiv.org/abs/2212.08073) |
| 课程讲次 | L04 Feedback + Tools/Code |
| 一句话定位 | 用少量人类原则驱动自我批判与修订，再以 RLAIF 训练更无害且少逃避的助手 |

## TL;DR

**Constitutional AI（CAI）** 几乎不用人类「有害偏好」标签：人只写一份 **Constitution（原则列表）**。先让模型按原则 **critique → revise** 并 SFT；再用 AI 反馈训 preference model 并做 RL（**RLAIF**）。结果在 harmlessness–helpfulness 折中上优于传统 HH-RLHF，且鼓励解释性拒绝而非含糊逃避。

## Problem

- RLHF 需要大量人类偏好标注，迭代慢、贵。  
- Helpfulness 与 Harmlessness 常冲突：只训无害易导致逃避式拒答。  
- 希望原则透明、可改，并探索 **AI 监督 AI（scalable oversight）**。

## Method

**Stage 1 — Supervised CAI**

1. Red-team 提示采样可能有问题的回复。  
2. 采样一条原则，生成 **critique**。  
3. 按批判 **revise**。  
4. 对修订回复做 SFT（可多轮修订）。

**Stage 2 — RL CAI / RLAIF**

1. 从模型采样成对回复。  
2. 用 Constitution 让 AI 标注偏好 → 训 preference model。  
3. 以 PM 为 reward 做 RL；可选 CoT 评价提高透明与表现。

人类监督压缩为 **O(10)** 条自然语言原则（课上约 16 条）。

## Key Results

- RL-CAI 在 harmlessness Elo 上显著更强，同时保持有竞争力的 helpfulness。  
- 相对 Helpful-only / HH 人类反馈基线，Pareto 前沿更好；CAI+CoT 进一步改善。  
- SL 阶段已能抬升无害性；RL 阶段提升可靠性。  
- 推动后续 RLAIF、Self-Refine、RL for self-correction 等工作。

## Course Connection

- L04 第三条反馈：**原则驱动的自我反馈**。  
- 与 L03 verifier/judge 同源：都是用模型评模型，但目标是对齐而非数学正确性。  
- 课上提醒：自评可能过自信 → 多模型共识批判更稳。  
- 自改进环：原则 → 合成数据 → 更新政策 → 更符合原则的行为。

## Engineering

- 把政策写成可版本化的 Constitution；按产品场景裁剪原则。  
- 流水线：红队采样 → critique/revise → SFT → AI 偏好 → RL。  
- 用小规模人类一致性集校准 preference model。  
- 监控过度拒答；明确「解释为何拒绝」的产品目标。  
- 改宪法后通常重跑后训练（占算力比例相对预训练更小）。

## Limitations

- 原则本身的价值选择与谁有权起草，是治理问题。  
- AI 反馈偏差会进入 PM；仍需人类抽检。  
- Helpfulness–harmlessness 张力不会消失，只是更好折中。  
- 持续学习 / 删除旧规则仍困难（课上开放问题）。  
- 预训练脏数据无法单靠 CAI 清零。
