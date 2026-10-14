# Training Verifiers to Solve Math Word Problems（GSM8K）

## Metadata

| 字段 | 内容 |
|------|------|
| 标题 | Training Verifiers to Solve Math Word Problems |
| 作者 | Cobbe et al.（OpenAI） |
| 年份 | 2021 |
| arXiv | [2110.14168](https://arxiv.org/abs/2110.14168) |
| 课程讲次 | L03 Robust Verification |
| 一句话定位 | 发布 GSM8K，并用「采样 + 验证器打分」证明 verification 比纯 SFT 更会吃数据 |

## TL;DR

在多步数学应用题上，大模型仍常错。作者发布 **GSM8K**（~8.5K 高质量小学数学题），并训练 **verifier** 判断候选解是否正确；测试时 **best-of-N** 取 verifier 最高分。验证相对纯微调有更优的数据缩放，增益约等于把模型放大一个数量级档位。

## Problem

- 自回归生成无法中途纠错，一步错满盘皆输。  
- 仅靠继续放大 generator，在数学推理上缩放不友好。  
- 需要可诊断、难度适中、含自然语言逐步解的基准。

## Method

1. **GSM8K**：人工出题；高多样性、中等难度、自然语言解题过程。  
2. **Finetune baseline**：SFT 后低温采 1 条判对错。  
3. **Verification**：高温采多条 → verifier 打分 → 取最高。  
4. Verifier：LM + 正确性头；标签来自终答是否匹配；可加 LM loss；常用 last-token 分数。  
5. 辅助：训练时注入计算器注解，减轻纯算术错误。

## Key Results

- Verification 相对 SFT 数据缩放更好；论文称提升约可比 **~30×** 模型规模带来的增益。  
- 课上补充：大 generator + 小 verifier 通常优于反过来；best-of-N 约到数百仍可能受益，过大则 verifier 混淆近邻解。  
- Dropout 等正则对 finetune 与 verification 都有帮助。

## Course Connection

- L02 Gen-Ver gap 的「第一代补丁」：用 ORM 式 verifier 替代/增强多数投票。  
- 直接引出 L03 的 ORM 概念；后续 PRM / Math-Shepherd / Weaver 都在此基础上加细粒度或集成。

## Engineering

- 有自动 checker 的任务：优先实现「多采样 + 二分类 verifier」。  
- 记录 N–准确率曲线，找到 verifier 精度崩溃点。  
- Generator 与 verifier 分网，减轻 generator 过拟合。  
- 算术域考虑工具（计算器）与验证解耦。

## Limitations

- 标签依赖可检查终答；开放式无标准答任务难直接套。  
- Token/句级噪声标签；过程错但终答对会被标正（ORM 假阳性）。  
- 当时依赖先 SFT generator；今日指令模型上流程可简化但仍需校准。
