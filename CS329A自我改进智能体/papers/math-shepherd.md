# Math-Shepherd

## Metadata

| 字段 | 内容 |
|------|------|
| 标题 | Math-Shepherd: Verify and Reinforce LLMs Step-by-step without Human Annotations |
| 作者 | Wang et al.（DeepSeek / 北大等） |
| 年份 | 2023 |
| arXiv | [2312.08935](https://arxiv.org/abs/2312.08935) |
| 课程讲次 | L03 Robust Verification |
| 一句话定位 | 用蒙特卡洛式自动标注训练 PRM，既做 best-of-N 验证，又做逐步 PPO 强化 |

## TL;DR

不必人工逐步打分：定义某步质量为「从该步继续采样后到达正确终答的潜力」，用 **hard/soft estimate** 自动造过程标签，训练 **Math-Shepherd PRM**。同一 PRM 可用于 **重排多候选** 与 **逐步 PPO**，在 GSM8K/MATH 上显著超过多数投票与 ORM，并提升开源 7B–70B 模型。

## Problem

- PRM 强于 ORM，但 PRM800K 式人工标注昂贵，阻碍落地。  
- 需要可扩展的过程监督信号，同时服务 **test-time verification** 与 **train-time RL**。

## Method

1. 对逐步解的中间步 \(s_t\)，用 LLM 解码 \(N\) 条后续轨迹。  
2. **Hard estimate**：任一条终答正确 → 该步成功。  
3. **Soft estimate**：正确轨迹比例作为分数。  
4. 用自动标签训 PRM；验证时对 N 候选打分取最高。  
5. **Step-by-step PPO**：以 PRM 为 reward 强化 generator。  
6. 实务：N 较小时 soft/hard 接近；可用 hard 简化。

## Key Results

- 验证设定下 Shepherd 优于 SC（多数投票）与 ORM；MATH 上相对增益往往更大。  
- 例：Mistral-7B 经 PRM-PPO 后 GSM8K **77.9%→84.1%**，MATH **28.6%→33.0%**；再加验证可到 **89.1% / 43.5%**。  
- DeepSeek-67B + 验证可达更高开源成绩（论文宣称工具无关设定下很强）。  
- PRM-PPO 优于仅用 ORM 的 RL；验证 + RL 可叠加但会平台期。

## Course Connection

- L03 演进第三步：**去掉人工逐步标注**。  
- 连接「验证」与「自改进 RL」——同一自动 PRM 服务两条路。  
- 课堂点名自动标注三大坑：稀有正确路径、难题无信号、错步好运。  
- 与 L04/L06 的执行反馈 RL、train-time RL 叙事一致。

## Engineering

- 需要终答可自动检查（数学 final answer）。  
- 控制 rollout N 与深度以平衡标签噪声和算力。  
- 先 verification 再决定是否上 PPO。  
- 可加工具（计算器/SymPy）增强逐步检查（课堂 Q&A）。  
- 监控假阳性步骤与 reward hacking。

## Limitations

- 依赖可验证终答；开放生成难直接用。  
- Hard/Soft 都是 outcome 启发式，不是真正的人类过程判断。  
- N 小会误杀难但正确的中间步；难题上标签稀疏。  
- 「错步但终答对」仍可能被标正。
