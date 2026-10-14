# DeepSeekMath-V2（元验证 · L20）

- **关联讲次**：L20；对照客座 Thang Luong / Gemini IMO  
- **地位**：在定理证明域用 generator–verifier–**meta-verifier** 自动化找推理缺陷

## 问题

只奖励最终答案会放过错误证明；朴素 LLM-as-judge 会把错证判对。

## 机制（课上）

1. 人类无参考解标 issue → 训 verifier  
2. Verifier 打分（0 / 0.5 / 1）  
3. Meta-verifier 检查「issue 是否真实、分数是否合理」  
4. 闭环改进 generator 与 verifier；逐步自动化标注

## 结果要点（课上）

- 迭代后 proof score 爬升  
- Best@32 在 2024 IMO shortlist 上约 **42%** proof score  

## Takeaway

LLM verifier 需要 **meta-verification** 防幻觉 issue；配方可迁到其他「过程易错」域，但仍偏易验证领域。

## 一句话

给判卷模型再加一层「查判卷」，才能把证明自改进环跑稳。
