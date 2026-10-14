# CS329A 全课程知识地图

> 用一张「闭环」把各讲串起来。细节见各 `LXX-*.md`。

## 1. 一条主线：Self-Improving Agent Loop

```text
目标 Goal
  → 生成候选（采样 / 规划 / 工具调用）
  → 验证 / 反馈（单元测试、PRM/ORM、执行、检索、人类/宪法）
  → 选择或修正（best-of-N、树搜索、critique-revise）
  → （可选）写入训练：SFT / RL / 进化
  → 能力提升 → 更难目标
```

| 环节 | 主要讲次 | 代表技术 |
|------|----------|----------|
| 生成更多候选 | L02, L08 | Repeated sampling, AlphaCode-scale search |
| 验证谁对 | L03 | ORM / PRM / Weak verifiers |
| 用环境反馈改 | L04 | ReAct, RLEF, Constitutional AI |
| 多步规划与搜索 | L05, L08 | LATS, ADaPT, Search-o1 |
| 训练时固化能力 | L06 | STaR, GRPO, DAPO |
| 开放探索 | L07 | AI Scientist, AlphaEvolve, ADAS / DGM |
| 真实软件任务 | L13 | CodeMonkeys, KernelBench |
| 记忆与上下文 | L14 | MemGPT, Cartridges, CacheBlend |
| 评测长程能力 | L17 | Long tasks, GDPVal, DeepScholar |
| 前沿开放问题 | L20 | 效率、开放端、可靠性 |

## 2. 两条缩放轴

```text
Train-time scaling          Test-time scaling
(数据/参数/RL 步数)          (采样数 N、搜索深度、工具轮次)
        \                      /
         \                    /
          →  同一能力 frontier：谁更便宜达到目标 accuracy？
```

- **课上核心权衡**：弱模型 + 大推理预算 vs 强模型 + 小预算（见 Snell / Monkeys）。
- **瓶颈常在 verifier**：coverage 上去了，选不出正确答案 → Gen–Ver Gap（L03）。

## 3. 概念速查（跨讲）

| 概念 | 一句话 | 首次重点出现 |
|------|--------|--------------|
| Coverage / pass@k | k 次采样中至少一次正确的比例 | L02 |
| 10@k | 只能提交 10 个候选时的成功率（需筛选） | L08 |
| ORM | 只看最终答案打分 | L03 |
| PRM | 逐步打分，可引导搜索 | L03 |
| Gen–Ver Gap | 会生成正确答案但选不出来 | L02–L03 |
| ReAct | Thought ↔ Action ↔ Observation | L04 |
| RLAIF / Constitutional | AI 反馈替代人类偏好标注 | L04 |
| Process supervision | 过程监督 > 纯结果监督（多数数学设定） | L03 |
| GRPO | 组相对策略优化，降 critic 成本 | L06 |
| Agentic RAG | 推理中途按需检索 | L08 |
| Open-endedness | 档案 + LLM 变异 + 可自动评测；目标/结构可扩展（非「无 reward」） | L07 |
| ADAS vs DGM | 课表「Automated design…」= ADAS (2408.08435)；材料链 2505.22954 实为 DGM | L07 |
| Long-horizon eval | 任务时长/经济价值，而非短题准确率 | L17 |

## 4. 决策表：你该加哪一招？

| 现象 | 优先尝试 | 参考讲次 |
|------|----------|----------|
| 单次生成偶尔对、经常错 | Repeated sampling + 可靠 verifier | L02–L03 |
| 采样很多仍选错 | 加强 verifier / PRM / 弱验证器集成 | L03 |
| 需要外部知识或环境 | Tools + ReAct / 执行反馈 | L04 |
| 任务要分解、会卡死 | ADaPT / 树搜索 / 回溯 | L05 |
| 推理太贵、想固化能力 | STaR / RL（GRPO/DAPO） | L06 |
| 要发现新算法/新工作流 | 进化搜索 / AI Scientist 式流水线 | L07 |
| 竞赛/代码：候选海选 | AlphaCode 式 filter→cluster→score | L08 |
| 知识缺口在长推理中传播 | Search-o1（检索+文内精炼） | L08 |
| SWE / kernel 等真实工程 | Test-time 闭环 + 专用 bench | L13 |
| 上下文装不下 / 多会话 | OS 式记忆分层、KV/RAG 缓存 | L14 |
| 「感觉变强了」说不清 | 长任务 / 经济价值评测 | L17 |

## 5. 论文阅读优先级（自学加速）

**必读（机制骨架）**

1. Large Language Monkeys  
2. Let's Verify Step by Step  
3. ReAct  
4. STaR  
5. LATS 或 ADaPT（二选一先读）  
6. Search-o1  
7. Measuring AI Ability to Complete Long Tasks  

**加深（系统/规模）**

- Snell compute-optimal、DeepSeekMath/GRPO、DAPO、AlphaCode 2、AlphaEvolve、MemGPT、CodeMonkeys  

**L20 自改进三瓶颈（课上收束）**

1. 多样轨迹（multi-agent finetuning 等）  
2. 元验证（验证验证器本身）  
3. 自提任务（Absolute Zero 式自博弈）  

另见效率轴：**Intelligence per Watt**（本地小模型覆盖大量查询）。

完整列表见 [`../papers/README.md`](../papers/README.md)。

## 6. 与 handbook 其他模块的映射

| 本课程主题 | handbook 模块 |
|------------|----------------|
| Agent loop / ReAct | `core/agent-loop`, `core/tool-use` |
| Planning / search | `core/reasoning-planning` |
| Memory | `core/memory`, `core/context-engineering` |
| RL / post-training | `runtime/reinforcement-learning` |
| Eval | `production/evaluation` |
| Coding agents | `agent-types/coding-agent` |
