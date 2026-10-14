# MemGPT: Towards LLMs as Operating Systems

- **链接**：https://arxiv.org/abs/2310.08560  
- **关联讲次**：L14  
- **机构**：UC Berkeley

## 问题

固定上下文窗口无法支撑长文档分析与多会话一致人格；硬扩窗口贵且未必会用。

## 机制

- **Main context**：system + working context + FIFO（含递归摘要）  
- **External**：recall / archival 存储  
- **Queue manager**：pressure 警告、flush、驱逐  
- **Function calling**：自我换页、检索、改写工作记忆；heartbeat 链式推理

## 实验域

- 超窗口文档 QA / 分析  
- 多 session 对话记忆与人设

## 工程 Takeaway

- 把上下文当稀缺物理资源，用 OS 策略管理。  
- 必须 token-aware（警告阈值、分页）。  
- 依赖可靠 tool calling；摘要错误 = 静默遗忘风险。

## 一句话

用虚拟内存思路 + 函数调用，让固定窗口 LLM 自我管理分层记忆。
