# CodeMonkeys: Scaling Test-Time Compute for Software Engineering

- **链接**：https://arxiv.org/abs/2501.14723  
- **关联讲次**：L13；上游 L02 Large Language Monkeys  
- **机构**：Stanford Scaling Intelligence 等

## 问题

SWE-bench 上如何**有意地**缩放串行 / 并行 test-time compute，并解决多候选的 **selection**？现成单解框架重复采样不够。

## 机制

1. **Context**：便宜模型全库扫描相关性 + 摘要；前沿模型 ranking → ~128k；成本摊销到多样本。  
2. **Generation**：Testing / Editing 双状态机；强制自写测试；执行反馈迭代；每 issue 多轨迹并行。  
3. **Selection**：生成测试投票滤候选 + Selection 状态机写区分性测试并决策。

## 关键数字

| 指标 | 值 |
|------|-----|
| SWE-bench Verified score | **57.4%** |
| Coverage（生成） | **~69.8%** |
| 总成本 | **~$2300** |
| Context recall @128k | **~92.6%** |
| Barrel of Monkeys ensemble select | **66.2%**（coverage 80.8%） |

## 工程 Takeaway

- 为并行缩放而设计 context（摊销 + cache）。  
- 补丁与测试同源生长。  
- Selection 可跨异构提交复用（Barrel of Monkeys）。  
- 相近预算下串并行配置 coverage 常接近——要画清曲线再调。

## 一句话

把 Monkeys 式并行缩放做成可提交的 SWE 系统：摊销读库、测试驱动迭代、投票+精选。
