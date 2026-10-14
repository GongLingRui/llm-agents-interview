# Intelligence per Watt（L20 课上研究）

- **关联讲次**：L20（Azalia Mirhoseini 等；与 Ray / Hennessy 等合作，课上介绍）  
- **地位**：把「智能 / 能耗」定义为推理时代的核心指标

## 问题

云端 LLM 需求与电力爆炸；多数用户查询其实不需要前沿大模型。本地加速器内存暴涨——如何量化并引导流量重分配？

## 定义

\[
\mathrm{IPW} = \frac{\text{average task accuracy}}{\text{average power draw}}
\]

- 课上「本地模型」工作定义：约 **≤20B active parameters**  
- 数据：~100 万真实查询 + 推理基准；多模型 × 多硬件

## 课堂数字

| 发现 | 值 |
|------|-----|
| 本地模型 chat 解决率进步 | 自 2023 **~3.1×**；覆盖约 **88.7%** 所述查询 |
| IPW 两年 | **~5.3×**（模型 3.1× × 硬件 ~1.7×） |
| M4 Max vs B200 | 本地 IPW 约低 **1.5×** |
| 简单查询占比 | 约 **77%** practical / info / writing |

## 开放方向（课上）

- 本地–云混合路由 serving  
- 面向边侧的能效架构 / kernel  
- Energy 作为一等优化目标

## 一句话

多数智能查询可由本地小模型完成；用 Intelligence per Watt 驱动下一代 serving 与硬件协同设计。
