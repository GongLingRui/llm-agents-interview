# ADaPT — As-Needed Decomposition and Planning

| 字段 | 内容 |
|------|------|
| 全称 | ADaPT: As-Needed Decomposition and Planning with Language Models |
| 作者 | Prasad, Koller, Hartmann, Clark, Sabharwal, Bansal, Khot 等 |
| 链接 | https://arxiv.org/abs/2311.05772 |
| 课程 | CS329A L05（必读；口播未展开） |
| 笔记 | [L05](../notes/L05-multi-step-planning.md) |

## 一句话

**先执行，失败再拆**：递归按需分解，使规划深度适配任务难度与 executor 能力。

## 问题

- ReAct 式 iterative executor：长轨迹组合难、干扰多。  
- 固定 plan-and-execute：子任务过难即整任务失败，无法自适应再分解。

## 机制

```text
尝试 executor(task)
失败且未超深度 → planner 分解为子任务
对每个子任务递归 ADaPT
```

Planner / Executor 模块分离；executor 带环境原子技能 few-shot；失败由 LLM 判定后触发更深分解。

## 关键数字（GPT-3.5）

相对强基线最高约：ALFWorld **+28.3%**、WebShop **+27%**、TextCraft **+33%** 成功率。  
相对 Reflexion：约 **+14.1% / +9% / +20%**。  
分析：递归深度随任务复杂度与模型强弱变化。

## 工程 takeaway

- 不要一上来把目标拆到原子——浪费调用。  
- 为 executor 设清失败判定与 `d_max`，防无限递归。  
- 可与 LATS/Reflexion **组合**（在模块内加强搜索或反思）。  
- 换更弱开源 executor 时，预期分解更深——提前预算深度。

## 与课程关系

补全 L05「分解」轴；与 SPRINT「静态 DAG 并行」不同：ADaPT 是 **运行时失败驱动的递归分解**。
