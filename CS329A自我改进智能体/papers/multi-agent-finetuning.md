# Multi-Agent Finetuning（多样推理链 · L20）

- **关联讲次**：L20（课上关键思想；具体论文以幻灯为准）  
- **地位**：用多专长 agent 缓解单模型合成数据同质化

## 问题

Rejection sampling / STaR 式自改进几轮后平台：单模型轨迹缺乏多样性。

## 机制（课上）

- Generation agents + Critic agents  
- 多轮：各自作答 → 汇总 → 批评 → 再生成 → majority → SFT  
- Critic 在「对 / 被纠正」轨迹上学习对比  
- 穷人版：多模型 / 多 prompt 混采样

## 结果要点（课上）

- 多 agent FT：性能与 embedding 多样性可维持上升  
- 单 agent FT：准确率塌缩  
- 邻域（如 GSM8K）有迁移

## Takeaway

监控合成数据多样性；塌了就上多 agent / 多模型，而不是盲目加迭代轮数。

## 一句话

多样性是自改进飞轮的燃料；多专长 agent 是造燃料的一种方法。
