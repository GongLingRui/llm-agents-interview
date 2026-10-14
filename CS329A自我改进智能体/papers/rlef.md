# RLEF: Grounding Code LLMs in Execution Feedback with Reinforcement Learning

## Metadata

| 字段 | 内容 |
|------|------|
| 标题 | RLEF: Grounding Code LLMs in Execution Feedback with Reinforcement Learning |
| 作者 | Gehring et al.（Meta FAIR） |
| 年份 | 2024 |
| arXiv | [2410.02089](https://arxiv.org/abs/2410.02089) |
| 课程讲次 | L04 Feedback + Tools/Code |
| 一句话定位 | 把多轮「写代码→跑测试→读反馈→再写」做成 MDP，用 PPO 让模型真正学会利用执行反馈 |

## TL;DR

**RLEF** 将代码生成做成多轮对话：每轮提交代码，获得 **public test** 执行反馈并继续改；结束时用 **private test** 给二元奖励，PPO 更新政策。相对独立采样，模型在 CodeContests 上以更少样本达到更强 solve rate，并泛化到其它代码基准——关键是训出「会用反馈迭代」的能力。

## Problem

- 编码 agent 必须在推理时消化执行反馈；但许多 LLM 多轮自修并不比同预算独立采样更好。  
- 需要端到端目标：既对齐题意，又把反馈用进策略。  
- 若训练时可见全部测试输出，模型可能抄答案而非学会修复。

## Method

1. **迭代合成**：题面 → 代码 → public 反馈 → 再生成，直至通过或达 turn 上限。  
2. **两层测试**：public 驱动内环；private 决定最终 reward（防泄漏/背答案）。  
3. **PPO + KL**：朝初始政策正则；可对非法代码给小惩罚。  
4. **Token 级政策 + Turn 级 value**：整段回复共享 advantage（课上类比序列级奖励）。  
5. 在 Llama 3.1 8B/70B 等上从 CodeContests 训练。

## Key Results

- CodeContests 上相对先前 SOTA 提升，且采样需求约降 **一个数量级**。  
- 训练后模型在多 turn 中错误输出减少、修复更有针对性。  
- 泛化到 HumanEval+、MBPP+ 等。  
- 课上强调：基座「看见反馈」≠「会用反馈」；RLEF 后才真正接地。

## Course Connection

- L04 第二条反馈来源：**执行 / 测试**。  
- 与 L03 ORM 类比：private pass/fail ≈ 可编程 outcome reward；细粒度 error 则偏向 process。  
- 与 ReAct 同构：Act=交代码，Obs=执行反馈，Thought 可含在回复中。  
- 指向 SWE-bench / CodeMonkeys：先检索再补丁再测。

## Engineering

- 严格隔离 public/private 或训练/验收测试。  
- 反馈模板包含失败用例、stderr、超时；限制 max turns。  
- 监控 reward hacking（刷 public、空洞实现）。  
- 难题考虑更富信号（堆栈、覆盖率、过程奖励）。  
- 仓库级任务先做上下文检索，再套同一执行环。

## Limitations

- 二元最终奖励对超难/长程调试可能稀疏。  
- 依赖可运行测试；缺测试的任务要自动生成测试（另题）。  
- 课上讨论：wrong↓ 但 timeout↑ 等现象域相关。  
- 竞赛题规模较小，迁到大型软件工程需额外检索与工具。
