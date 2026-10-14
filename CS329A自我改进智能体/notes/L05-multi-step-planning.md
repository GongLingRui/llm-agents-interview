# L05 — Multi-step Reasoning / Planning

> 课程：Stanford CS329A · Self-Improving AI Agents（Autumn 2025）  
> 材料：`raw-transcripts/5.planmulti-stepreansoning.md` + 官网必读论文  
> 相关讲次：L02（test-time compute）· L03（验证器）· L04（ReAct / feedback）→ **本讲** → L06（train-time RL）

---

## 1. 一句话总览

多步 Agent 任务不是「一次采样出答案」，而是在 **推理 → 行动 → 观测 → 搜索/修订** 的闭环里探索多条轨迹；本讲用 **ADaPT（按需分解）**、**LATS（MCTS 树搜索）**、**SPRINT（交错规划+并行执行）**、**AB-MCTS（wider vs deeper）**、**SWiRL（离线合成数据 + 多步 RL）** 说明：何时搜、何时直接生成、何时并行、如何用过程奖励把能力写进模型。

---

## 2. 核心概念与定义

### 2.1 单步 vs 多步

| | 单步（single-step） | 多步（multi-step） |
|--|---------------------|---------------------|
| 典型设定 | 一次生成完整回答；奖励看最终结果 | 一连串 reason / act / observe；中途可改计划 |
| 优化对象 | 整段输出的 outcome | 轨迹上的每一步 + 终局 |
| 失败模式 | 答错即错 | **错误复合**；局部合理动作可能堵死后路 |
| 本课对照 | RLHF / RLAIF / 多数 RLEF | LATS、ADaPT、SPRINT、SWiRL |

课堂动机（旅游规划例子）：

1. **Reason**：预算、目的地约束、偏好  
2. **Act**：搜网页、读 Reddit、问朋友  
3. **Search / revise**：根据反馈换目的地、补行程  

目标不是「让 LLM 吐出一个 plan 然后执行」，而是鼓励 **多样化探索** 多条路径，并用环境反馈更新后续搜索。

### 2.2 四类「规划原语」（本讲地图）

```text
分解（ADaPT）     → 任务太大就拆；能执行就不拆
树搜索（LATS）    → 用 MCTS 在动作树上 explore / exploit
并行交错（SPRINT）→ 独立子计划并行执行，缩短串行 token
宽深自适应（AB-MCTS）→ 有外部反馈时，动态选 widen 还是 deepen
合成数据+多步 RL（SWiRL）→ 离线造轨迹 + process reward，训练时不跑 live tool
```

### 2.3 与前后讲的关系

- **相对 L02**：不只是 best-of-N / majority；要在 **结构化搜索** 上把 test-time compute 换成准确率。  
- **相对 L03**：LATS / SWiRL 的打分依赖 LLM-as-Judge 或 outcome；验证器质量仍决定搜索会不会走偏。  
- **相对 L04**：ReAct 是单轨迹「想-做」；LATS 在其上加 **树 + UCT + reflection + 回传**。  
- **通向 L06**：SPRINT / SWiRL 已把「结构化轨迹」做成训练数据；L06 讲如何用 STaR / GRPO / DAPO 把这种行为写进权重。

### 2.4 何时用 search，何时直接生成（决策表）

| 信号 | 倾向 **直接生成 / CoT** | 倾向 **搜索 / 树 / 并行规划** |
|------|-------------------------|--------------------------------|
| 任务步数 | 短、单跳 | 多跳、长 horizon、复合错误风险高 |
| 外部反馈 | 无 / 弱 | 有环境观测、工具、测试、可回滚模拟 |
| 动作可逆性 | 不可逆（付款、发邮件）→ **勿盲搜真环境** | 可逆 / 沙箱 / 模拟 → 可展开多分支 |
| 子任务依赖 | 强串行依赖 | 存在独立子图 → 可 SPRINT 式并行 |
| 成本预算 | 延迟与 API 成本敏感 | 愿意用更多 inference 换准确率 |
| 基座能力 | 问题已在能力内 | 需要探索多解、或需要 refine from feedback |

**经验法则（课堂语气）**：  
能一次做对就别开树；开树就要有 **可执行的打分与终止条件**；真世界不可逆动作先放模拟 / approval gate。

---

## 3. 方法 / 机制

### 3.1 ADaPT — As-Needed Decomposition and Planning

> 课程必读；本讲口播未展开，以下据论文补全，并与 LATS/SPRINT 对照。

**问题**：

- **Iterative executor**（如 ReAct）：每步只选下一动作，长轨迹易淹没、组合能力弱。  
- **Plan-and-execute**：先定死高层计划；任一子任务太难 → 整任务失败，且无法按需再拆。

**机制（递归 as-needed）**：

```text
function ADaPT(task, depth):
  try executor(task)          # 先直接干
  if success: return ok
  if depth >= d_max: fail
  plan = planner.decompose(task)   # 失败才拆
  for sub in plan:
    ADaPT(sub, depth+1)            # 递归；失败的子任务再拆
```

要点：

1. **Planner / Executor 分离**：executor 用环境原子技能 few-shot；planner 用世界知识出子任务。  
2. **只在失败时分解**：避免过度分解的开销。  
3. **适配执行器能力**：弱模型拆得更深；强模型少拆。  
4. **适配任务复杂度**：复杂任务递归更深（论文分析）。

**与本讲其他方法**：ADaPT 管「何时拆目标」；LATS 管「在动作空间怎么搜」；二者可组合（executor 内嵌搜索 / planner 内嵌反思）。

### 3.2 LATS — Language Agent Tree Search

**定位**：把 **CoT（分解推理）+ ReAct（行动与观测）+ MCTS（规划搜索）** 统一到语言 Agent。

#### 六阶段循环

对状态（含 prompt、历史动作、环境观测）：

1. **Selection**：用 **UCT** 选待扩展节点（平衡 exploit / explore）  
2. **Expansion**：从节点采样多个候选动作（可并行执行）  
3. **Evaluation**：给新状态打分  
4. **Simulation**：从高分分支贪心或继续采样，走到终止 / 预算耗尽  
5. **Backpropagation**：用轨迹回报更新路径上节点价值  
6. **Reflection**：模型写「为何成/败」摘要，追加到上下文，提升后续质量

#### 价值与选择公式（课堂直觉）

状态价值近似：

```text
V(s) ≈ w1 * LLM_Judge(s) + w2 * SelfConsistency(action)
```

- **LLM-as-Judge**：问「该状态成功前景如何」，给 0–1。  
- **Self-consistency**：同动作被采样频率越高，越像模型「共识」。

UCT（来自 MCTS / bandit）：

```text
UCT(s) = V(s) + c * sqrt( ln(N_parent) / N_s )
```

- `N_s` 相对父节点访问少 → 探索项大 → 鼓励少访节点。  
- 纯贪心会锁死早期高分支，错过后期更好路径。

回传（课堂口述形式）：

```text
V_new(s) = ( V_old(s) * (N_s - 1) + Return ) / N_s
```

（随访问次数做增量平均；细节以实现为准。）

#### 与 Math-Shepherd / ReAct 的差异（课上明确对比）

| | Math-Shepherd 类 | ReAct | LATS |
|--|------------------|-------|------|
| 打分对象 | 推理轨迹逐步 verifier | 无树搜索 | **动作结果 + 观测** |
| 结构 | 搜索推理步 | 单轨迹 | **树 + UCT** |
| 反馈 | 过程分 | 环境观测 | 观测 + **reflection** |

#### 课堂迷宫例子（帮助记忆）

暗室两门 → 采样：开左门 / 开右门 / 找线索 → 执行得观测 → Judge+频率打分 → 扩展高分支直到出口 → 成败回报回传 → 写 reflection。

### 3.3 SPRINT — Interleaved Planning + Parallel Execution

**动机**：推理模型（o1 / Gemini Think / R1 等）越难越想越长，准确率上升，但 **大量子步骤彼此独立**，串行吐 token 浪费 wall-clock 与成本。

**训练数据管线（用 LLM 标注 LLM）**：

```text
R1 等生成长推理轨迹
    ↓ GPT-4o 等标注
切成 steps；标 plan vs execution
    ↓
建依赖 DAG（谁依赖谁）
    ↓ packing
把无依赖步骤打包成可并行 stage
    ↓ SFT
教模型输出带 tag 的 plan/exec 协议
```

**推理时协议（改输出协议，不改 Transformer 架构）**：

```text
Planner 输出可并行的 plan_1, plan_2, ...
    ↓ runtime
Executors 并行跑（可含工具：计算器 / Python）
    ↓ sync
结果写回同一 autoregressive context
    ↓
下一轮 plan / exec（可 replan）
```

要点：

- 仍是 next-token；并行发生在 **runtime 看到 tags 后的分支执行**。  
- 修订 / 回退仍可行：完整上下文保留。  
- 早期 stage 更宽（探索），后期更窄（收敛）——与 AB-MCTS「宽深」直觉相通。  
- **Straggler**：并行步时长不均；可用合并过碎步骤等启发式，无法完全消除。

### 3.4 Wider or Deeper? — Adaptive Branching Tree Search（AB-MCTS）

> 课程必读；口播未展开。核心问题：有外部反馈时，固定宽度的 MCTS / 纯 repeated sampling 都不理想。

| 策略 | 行为 | 局限 |
|------|------|------|
| Repeated sampling（wider） | 同 prompt 多样本 | 不 refine |
| 顺序 refine（deeper） | 沿一条改 | 多样性不足 |
| 固定宽 MCTS | 每节点固定分支数 | 限制 LLM 输出空间多样性 |

**AB-MCTS**：每个节点用反馈与后验，**动态决定** 再开新候选（wider）还是深挖已有候选（deeper）。  
适用：编码 / 工程任务有测试反馈；与 LATS「环境动作树」互补——AB-MCTS 更强调 **答案候选树上的宽深权衡**。

### 3.5 SWiRL — Synthetic Data + Multi-step（Step-Wise）RL

**设计目标**（课堂列举）：

1. 会选工具、会写对 query  
2. 多步间保持准确、能从错误恢复  
3. 知道何时停、出最终答案  
4. **训练时尽量不 live 跑工具**（慢、易挂、拖垮 RL）  
5. 希望跨工具 / 跨任务泛化

#### Stage 1：离线造多步轨迹

每步允许：CoT + 调工具 **或** 出最终答案。  
工具执行写进下一状态上下文。  
一条 \(k\) 步轨迹拆成 \(k\) 个 **前缀 → 下一动作** 子轨迹。

过滤策略（课上重点）：

| 过滤 | 含义 |
|------|------|
| Process filter | Judge 认为每步动作合理（可不看终局对错） |
| Outcome filter | 只留最终答案正确 |
| Both | 过程与终局都过 |
| Random | 对照 |

#### Stage 2：多步 RL（无 live tool）

训练时：给记录好的前缀上下文 → 模型提议下一动作 → 用 **存好的 process reward**（Judge 评「query/动作是否合理」）做优化；**不重新执行工具**。

直觉：Judge 往往不需要真实检索结果也能判断「先查 A 的年龄」是否是好动作。

目标（课堂形式）：

```text
最大化 E[ R(action | 到目前为止的 context) ]  （对轨迹上每一步）
```

#### 推理时

真正执行 calculator / search / Python；用 `<tool>` / `<answer>` 类标签控制终止。

#### 为何 process-only 过滤对 RL 更好？

- 只留「已经答对」的轨迹 → 模型重复会做的事，难学会新问题。  
- Process 好但 outcome 错的轨迹 → 仍提供 **可学的中间决策**；RL 在同一前缀上可试更好下一步。  
- **SFT 相反**：模仿学习更吃「过程+终局都正确」的示范；脏 outcome 会污染目标。

---

## 4. 关键数字与实验结论

> 以下以课堂口播与论文报告为准；不同设定不可直接横向比绝对分。

### LATS

- **HotPotQA**：多跳 QA，至少两篇 Wiki；增加轨迹/样本量显著提升；**reflection** 带来额外提升。  
- **WebShop**：无微调、纯 test-time；结果接近人类专家水平（课上表述）。  
- **代价**：树扩展 / 回传 / Judge 调用成本高；论文对 cost–benefit 分析不足。  
- **风险**：不可逆动作（支付等）场景不直接适用。

### SPRINT

- 数据：约 **6k** MATH 思维轨迹；保留高并行度；SFT **DeepSeek-R1-Distill-Qwen-7B**。  
- 相对 R1-Distill-7B：**约 +3.5%** 准确率，同时显著减少 **串行 token**。  
- 长轨迹（例如 >8k token 档）串行 token 可降约 **39%**（论文）；课上亦提 MATH 上约 **40%** 量级带宽/串行节省（任务相关）。  
- **OOD**：Countdown、GPQA Diamond 未训仍更好准确率 + 更多并行机会。  
- 简单短题：强行 plan–exec 可能 **不如 baseline**（并行机会少，开销主导）。

### SWiRL

- 合成：Gemma-2-27B；HotPotQA + GSM8K 等，约 **50k** 量级轨迹规模（论文：HotPotQA 10k 题 ×5；GSM8K 7.5k × 轨迹）。  
- 课堂关键迁移：  
  - 训 GSM8K（SymPy）→ 测 HotPotQA：**65 → 71**  
  - 训 HotPotQA（search）→ 测 HotPotQA：**65 → 73**；反向迁移同样成立。  
- 合成数据 **100 → 10k**：OOD（如 math）持续涨。  
- 相对提升（论文摘要）：GSM8K / HotPotQA / CofCA / MuSiQue / BeerQA 等相对准确率提升约 **11–21%** 量级。  
- RL **明显优于** 同数据上的多步 SFT；SFT 更吃 process+outcome 双过滤。

### ADaPT（论文）

- GPT-3.5：相对强基线，ALFWorld / WebShop / TextCraft 成功率最高约 **+28.3% / +27% / +33%**。  
- 相对 Reflexion：约 **+14.1% / +9% / +20%**。

### AB-MCTS（论文）

- 同 inference budget 下，优于 repeated sampling 与固定宽 MCTS（编码 / ML engineering / ARC-AGI 等设定）。

---

## 5. 课堂 Q&A 与易错点

### Q1：LATS 的 UCT 一定最优吗？试过别的 bandit 吗？

**A**：课上答：没系统比。贡献是平台；可换 Thompson / 其他 explore 策略。UCT 是经典折中，非证明最优。

### Q2：树里重复动作怎么处理？

**A**：同一父节点下重复动作会增加访问计数，进入 UCT；假设是 **树** 而非任意图，不自动做全局状态合并。

### Q3：SPRINT 需要改模型结构成多头并行吗？

**A**：不需要。仍是自回归；用 **标签协议** 标出可并行 plan，由 runtime 并行执行再写回 context。

### Q4：并行分支各自「对」但合并后矛盾怎么办？

**A**：最终要在统一 context 里 **综合出答案**；串行 CoT 也会自相矛盾。SPRINT 更常见是 **同一问题的不同子步骤**，而非完全对立的多解硬投票。

### Q5：如何负载均衡并行步？

**A**：难保证等时；可能有 straggler。可合并过碎 execution；优化空间仍大。

### Q6：SWiRL 的 Judge 看不到工具真实输出，分准吗？

**A**：评的是 **动作/查询质量**（例如该不该搜这个人的年龄），不是评检索页内容；工具结果已在离线数据的历史上下文里。

### Q7：要不要专门训一个 Judge？

**A**：课上实现是强模型 **prompt 评判**，未另训 reward model。

### 易错点清单

1. 把 LATS 当成「只多采样 CoT」——缺了 **环境动作价值 + UCT + 回传 + reflection**。  
2. 在不可逆环境直接 MCTS 展开。  
3. 假设 SPRINT 总会加速——**简单题可能更慢**。  
4. 用 SFT 心态过滤 SWiRL 数据（只留全对轨迹）——可能 **削弱** 多步 RL。  
5. 混淆「过程分变好」与「获得全新领域知识」——泛化来自 **多步工具使用习惯**，仍受基座知识限制。

---

## 6. 本讲论文速查

| 论文 | 卡片 | 课堂角色 |
|------|------|----------|
| LATS | [lats.md](../papers/lats.md) | 主讲：MCTS Agent |
| SPRINT | [sprint.md](../papers/sprint.md) | 主讲：并行交错规划 |
| SWiRL | [swirl.md](../papers/swirl.md) | 主讲：合成数据 + 多步 RL |
| ADaPT | [adapt.md](../papers/adapt.md) | 必读补充：按需分解 |
| Wider or Deeper? (AB-MCTS) | [ab-mcts.md](../papers/ab-mcts.md) | 必读补充：宽 vs 深 |

---

## 7. 工程可落地 Checklist

- [ ] 画清任务图：哪些边是 **环境动作**，哪些是纯思维；动作是否可逆。  
- [ ] 先定 **终止条件** 与 **打分器**（规则 / 测试 / Judge），再开搜索。  
- [ ] ReAct 基线跑通后，再加树：记录节点数、LLM 调用次数、延迟。  
- [ ] 不可逆动作：模拟器 / dry-run / 人工审批闸门。  
- [ ] 长 CoT：检查是否存在可并行子图；有则考虑 SPRINT 式 tag + runtime。  
- [ ] 监控并行 **straggler** 与 sync 开销；简单题关闭并行。  
- [ ] 造多步数据时：**逐步** 交互式标注，便于拆成前缀样本。  
- [ ] RL 数据优先试 **process filter**；SFT 优先 **process+outcome**。  
- [ ] 训练阶段避免 live tool；推理阶段再接真工具。  
- [ ] 测 **跨工具 / 跨数据集** 迁移，避免只过拟合单一 API schema。  
- [ ] 对比「加 search」vs「直接生成」的 **准确率–成本 Pareto**。  
- [ ] 日志：每步动作、观测、Judge 分、是否回传/反思——否则无法调 UCT/`c`。

---

## 8. 若你只记 5 条

1. **多步 Agent = reason + act + observe + search**；单次 CoT 不够应对错误复合。  
2. **ADaPT**：能执行就不拆；失败再递归分解——适配模型能力与任务难度。  
3. **LATS**：MCTS + 动作结果打分 + UCT + reflection，把 test-time compute 变成多跳表现；贵，且怕不可逆动作。  
4. **SPRINT**：用标注把长思维收成 DAG，SFT 出并行协议；减串行 token，常附带准确率与 OOD 收益。  
5. **SWiRL**：离线多步轨迹 + process reward 做逐步 RL；**process 过滤** 利于 RL 泛化，RL 优于模仿；训练时不跑 live tool。

---

## 附录 A：本讲口播结构（便于对照 raw）

| 时间线（约） | 内容 |
|--------------|------|
| 0:00–22:00 | LATS：动机、六阶段、UCT、实验、Q&A |
| 23:00–49:00 | SPRINT：并行动机、DAG 数据、SFT、效率与准确率、Q&A |
| 50:00–结束 | SWiRL：离线合成、process RL、泛化、vs SFT |

ADaPT / AB-MCTS：官网 Lecture 5 阅读列表，口播未讲，笔记已按论文补全。

## 附录 B：一页伪代码总览

```text
# 测试时搜索型（LATS / AB-MCTS）
while budget:
  node = select(UCT or adaptive wide/deep)
  children = expand(LLM_actions or LLM_answers)
  scores = evaluate(env_or_tests + judge)
  backup(scores); maybe reflect

# 按需分解（ADaPT）
try execute(task)
else: for sub in plan(task): recurse(sub)

# 并行交错（SPRINT inference）
plans = model.parallel_plans(context)
obs = parallel_exec(plans); context += obs; repeat

# 训练写入权重（SWiRL）
data = offline_rollouts_with_tools()
data = process_filter(data)
RL_stepwise(policy, R_process)  # no live tools
```
