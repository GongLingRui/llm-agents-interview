# LATS — Language Agent Tree Search

| 字段 | 内容 |
|------|------|
| 全称 | Language Agent Tree Search Unifies Reasoning, Acting and Planning in Language Models |
| 作者 | Zhou et al. |
| 链接 | https://arxiv.org/abs/2310.04406 |
| 课程 | CS329A L05（主讲） |
| 笔记 | [L05](../notes/L05-multi-step-planning.md) |

## 一句话

把 **CoT + ReAct + MCTS** 接到同一语言 Agent：在动作树上用 UCT 探索，用环境结果与反思更新价值。

## 问题

单次 plan-and-execute 或缺搜索的 ReAct：难产生多样轨迹，也难把环境反馈系统性地用于多步优化。

## 机制

1. **Selection**：UCT 选节点  
2. **Expansion**：采样多动作（可并行执行）  
3. **Evaluation**：`LLM-Judge` + self-consistency 频率  
4. **Simulation**：沿高分分支走到终止/预算  
5. **Backpropagation**：轨迹回报更新路径价值  
6. **Reflection**：成/败文字反思写入上下文  

与 Math-Shepherd 等对比：打分偏 **动作结果/观测**，不只推理文本逐步分。

## 关键数字 / 实验

- HotPotQA：多样本/多轨迹与 reflection 明显抬分（多跳检索设定）。  
- WebShop：无微调、test-time 接近人类专家（课上表述）。  
- 代价：树扩展与 Judge 调用昂贵；不可逆动作场景需隔离。

## 工程 takeaway

- 先有可逆环境或模拟器，再上 MCTS。  
- 记录 visit / value / reflection，否则无法调 `c`。  
- 模块化：可换 bandit 选择规则；论文未穷尽对比。  
- 生产：对支付类动作加审批，勿直接树搜索真执行。

## 与课程关系

L04 ReAct 的「树化升级」；L02 test-time compute 的结构化用法；为 L05「何时 search」提供默认答案之一。
