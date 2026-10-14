# Improving Parallel Program Performance with LLM Optimizers via Agent-System Interfaces

- **链接**：https://arxiv.org/abs/2410.15625  
- **关联讲次**：L13  
- **机构**：Stanford / Intel / NVIDIA 等

## 问题

Task-based HPC 中 mapper（任务与数据放置）手写极贵；经典 autotuner 只吃标量 reward，样本效率低。

## 机制

**Agent-System Interface (ASI)**：

1. **DSL**：抽象底层 API，定义映射搜索空间，编译为系统代码  
2. **AutoGuide**：原始日志 → 可操作自然语言反馈  
3. **Generative optimization**：LLM 多轮改 DSL/mapper，利用富反馈

## 关键数字

| 对比 | 结果 |
|------|------|
| vs 专家 mapper | 最高约 **1.34×** 加速（9 benchmarks） |
| vs OpenTuner | **10 iter** 优于 OpenTuner **1000 iter**（约 **3.8×** 性能口径） |
| 调优时间 | 天 → **分钟** |

## 工程 Takeaway

- 先做好 **agent 可见的接口与反馈编译**，再堆搜索。  
- 系统优化是 generative optimization 的高价值域。  
- Ablation 证明 DSL/AutoGuide 非装饰。

## 一句话

给 LLM 一套可搜索的 DSL 和可读的执行指导，比纯标量 RL autotune 高效一个数量级。
