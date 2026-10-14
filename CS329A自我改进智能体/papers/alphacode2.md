# 论文卡：AlphaCode 2

| 字段 | 内容 |
|------|------|
| **标题** | AlphaCode 2 Technical Report |
| **作者** | AlphaCode Team, Google DeepMind（Leblond et al.） |
| **出处** | Tech report, Dec 2023 |
| **链接** | https://storage.googleapis.com/deepmind-media/AlphaCode2/AlphaCode2_Tech_Report.pdf |
| **课程** | CS329A L08 |

## 问题

在 AlphaCode 同一竞赛设定下，用 **Gemini** 与更好的搜索/重排，能否大幅提高 solve rate 与样本效率？

## 机制（怎么做）

1. **Gemini Pro** 两轮 GOLD fine-tune → **多个 policy**（超参多样性）。  
2. 仅 C++；随机温度与 metadata；采样可达 1M。  
3. 公开测例过滤（去掉 ~**95%**）→ 行为聚类 → 留最大 10 簇。  
4. 独立 **scoring model**（0–1 正确性）每簇挑最优 → ≤10 提交。

## 关键实验 / 数字

- 12 场近期 Codeforces、77 题：solve **43%** vs AlphaCode **25%**（~**1.7×**）。  
- 估计约 **85th percentile**；最好两场 >**99.5%** 参赛者。  
- ~**100 samples** ≈ 原版 1M 的表现（极大样本效率）。  
- 性能仍随采样预算近似 log-linear。

## 局限

- 系统昂贵；大量无效样本；强依赖可执行过滤。  
- 人机协作加过滤属性可到 ~90th percentile（报告叙述）。

## 工程 takeaway

- **更强基座 + 多政策多样性 + 学习打分**，往往比死堆采样更划算。  
- Scorer 训练数据应同分布但控制污染。

## 一句话

**Gemini 家族生成 + 过滤聚类 + 学习型 scorer：少采样达到更高竞赛百分位。**
