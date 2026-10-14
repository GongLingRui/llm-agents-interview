# SPRINT — Interleaved Planning & Parallel Execution

| 字段 | 内容 |
|------|------|
| 全称 | SPRINT: Enabling Interleaved Planning and Parallelized Execution in Reasoning Models |
| 作者 | Biju, Talaei, Huang, Pourreza, Mirhoseini, Saberi 等 |
| 链接 | https://arxiv.org/abs/2506.05745 |
| 课程 | CS329A L05（主讲） |
| 笔记 | [L05](../notes/L05-multi-step-planning.md) |

## 一句话

把长串行 CoT **重写成 plan–DAG–并行 exec** 监督数据，SFT 后让模型在推理时输出可并行协议，减串行 token 且常涨点。

## 问题

LRM 越难越长 → 准但慢；轨迹中大量子步骤独立，却被强制串行生成。

## 机制

**数据**：R1 等轨迹 → LLM 切步、标 plan/exec、建依赖 DAG、packing 可并行 stage → 过滤高并行样本。  
**训练**：少量结构化轨迹 SFT（课上约 6k MATH；论文亦称约 1.7k 级演示即可解锁行为，以论文设定为准）。  
**推理**：planner 一次吐多个独立 plan → executors 并行（可含工具）→ sync 回 context → 交错下一轮；**不改 Transformer 结构**。

## 关键数字

- R1-Distill-7B：准确率约 **+3.5%**（课），并显著减串行 token。  
- 长轨迹（>8k）：串行 token 可降约 **39%**；MATH 场景课上亦提 ~40% 量级。  
- OOD：GPQA / Countdown 准确率匹配或更好，串行 token 大降（论文：GPQA ~45%、Countdown ~65% 等，视长度档）。  
- 短简单题：可能不如 baseline（并行开销 > 收益）。

## 工程 takeaway

- 先分析任务是否真有并行宽度；无宽度别强上。  
- Runtime 必须实现 tag 解析、并行、同步、写回。  
- 处理 straggler；过碎步骤可合并进 plan。  
- 后续可用 GRPO 等在并行协议上继续 RL（课上展望）。

## 与课程关系

连结 L02「加 compute」与「加 wall-clock 效率」；与 AB-MCTS「先宽后窄」行为同构；数据策展套路与 SWiRL 同属「LLM 标 LLM」。
