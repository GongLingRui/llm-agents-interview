# KernelBench: Can LLMs Write Efficient GPU Kernels?

- **链接**：https://arxiv.org/abs/2502.10517  
- **关联讲次**：L13；L20（可执行但不平凡的验证）  
- **机构**：Stanford 等（Ouyang, Guo, Mirhoseini, Ré, …）

## 问题

LLM 能否从 PyTorch 参考实现生成**正确且快**的 GPU kernel？现有代码基准多测正确性，少测性能工程。

## 机制

- 基准：**250** 个 ML workload 任务  
- 指标：\(\mathrm{fast}_p\) = 正确且加速比 \(> p\) 的占比  
- 协议：支持多轮；可喂执行错误与 **profiler** 反馈

## 关键数字

| 观察 | 值 |
|------|-----|
| 顶尖模型零样本打平 PyTorch | **< ~20%** 任务 |
| 迭代 + 执行/profile 反馈 | 显著提升（课上/文中例：R1 某级 ~36%→~72%） |
| 提高 \(p\) | 难度陡增 |

## 工程 Takeaway

- Perf agent 需要 **profile 文本接口**，不是只看 pass/fail。  
- 平台相关优化仍是缺口。  
- 评测要防 baseline / 计时 reward hacking（社区后续 Verified 工作）。

## 一句话

正确且快的 kernel 生成是硬 agent 题；执行+profiler 迭代是当前最有效杠杆。
