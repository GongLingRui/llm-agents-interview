# 论文卡：Automated Design of Agentic Systems (ADAS)

| 字段 | 内容 |
|------|------|
| **标题** | Automated Design of Agentic Systems |
| **作者** | Shengran Hu, Cong Lu, Jeff Clune |
| **出处** | arXiv:2408.08435 → ICLR 2025 |
| **链接** | https://arxiv.org/abs/2408.08435 |
| **课程** | CS329A L07（课表条目名；勿与 2505.22954 DGM 混淆） |

## 问题

Agent 的 building blocks（CoT、Self-Refine、Debate、工具…）多为手写组合——能否在**代码空间**自动发明更好的 agentic systems？

## 机制（怎么做）

定义 ADAS 三要素：**search space / search algorithm / evaluation**。  
提出 **Meta Agent Search**：

1. Archive 存已发现 agents（可种子初始化）。  
2. Meta-LLM 读 archive → 写「有趣」新 agent 的 `forward()` 代码。  
3. Self-reflection 保新颖；验证集评测，失败则修复。  
4. 写入 archive，迭代。

代码空间 Turing-complete → 理论上可表示任意 prompt/工具/工作流。

## 关键实验 / 数字

- ARC：逐步超过 CoT / COT-SC / Self-Refine / Debate / QD 等手写基线。  
- DROP F1 **+13.6**；MGSM **+14.4%**；优于仅 prompt 优化（OPRO）。  
- 从 MGSM 迁到 GSM8K **+25.9%**；跨模型迁移仍保持优势。

## 局限

- 科学/多任务域增益较小（可能受基座知识上限）。  
- Meta 固定、不算「自我改写」（对比 DGM）。  
- 安全：自动生成任意代码 agent 需沙箱。

## 工程 takeaway

- 先把 agent 框架缩成可评测的 `forward`，再让 meta-LLM 做开放式架构搜索。  
- **Archive stepping stones** 比只保留当前最优更重要。

## 一句话

**用 meta-agent 在代码里「编程出更好的 agent」，并用不断增长的档案做开放式搜索。**
