# Let’s Verify Step by Step

## Metadata

| 字段 | 内容 |
|------|------|
| 标题 | Let’s Verify Step by Step |
| 作者 | Lightman et al.（OpenAI） |
| 年份 | 2023 |
| arXiv | [2305.20050](https://arxiv.org/abs/2305.20050) |
| 课程讲次 | L03 Robust Verification |
| 一句话定位 | 在 MATH 上系统对比 ORM vs PRM，证明过程监督更可靠，并开源 PRM800K |

## TL;DR

用 **过程监督（PRM）** 给每一步推理打标签，对比只看终局的 **ORM**。在更具挑战的 MATH 上，PRM 显著优于 ORM 与多数投票；主动学习优先标注「高分错解」可提升约 **2.6×** 数据效率；发布 **PRM800K**（~80 万步级人工标签）。

## Problem

- 多步推理中幻觉/逻辑错误会级联。  
- Reward model 的可靠性决定 best-of-N / RL 上限。  
- 过程监督更对齐、可解释，但人工贵；需证明其相对 outcome 监督的收益，并降低标注浪费。

## Method

1. **Generator** 产出换行分隔的逐步解（格式 SFT）。  
2. **ORM**：按终答对错训；测试用 **最后 token** 分数。  
3. **PRM**：人对每步标 + / − / neutral；测试分数为各步正确概率的 **乘积**。  
4. 监督通常只标到 **第一个错误步**，使与 ORM 的信息对比更干净。  
5. **Active learning**：迭代重训 PRM，优先给人看 *convincing wrong-answer* 解。  
6. 大尺度用 GPT-4 级；小尺度用大 PRM 代理人类做消融。

## Key Results

- 代表 MATH 子集上 SOTA 级 PRM best-of-N 约 **78%**。  
- PRM > ORM > majority voting；PRM 更能捞出稀有正解。  
- PRM 更 sample-efficient，分布外更稳。  
- Active learning **~2.6×** 数据效率。

## Course Connection

- L03 核心二分：**ORM vs PRM**。  
- 解释为何过程监督降低「过程错、终答对」假阳性。  
- 为 Math-Shepherd「去掉人工逐步标注」与 ORM+PRM 组合用法做铺垫。  
- 课堂强调：本工作焦点是 **训可靠 RM 做搜索**，而非立刻 RL 改 generator。

## Engineering

- 需要逐步解析约定（换行/标记）。  
- 标注预算紧时：主动选高分错解，而非均匀采样。  
- 部署时定义 neutral 的合并规则与 PRM 乘积阈值。  
- 若用 PRM 做 RL，警惕跳步/讨好打分器；可保留 ORM 终答门闩。

## Limitations

- 人工逐步标注极贵，难复制到所有域。  
- ORM/PRM 标签预算难严格对齐（步数可变）。  
- 无可靠终答 checker 的域，「只标到首错」成本接近完整判题。  
- PRM 仍可能错误 credit assignment；常需与 ORM 联用。
