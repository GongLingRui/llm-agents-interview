# L04 · Learn from Feedback with Tools & Code（工具 / 代码反馈）

> 课程：Stanford CS329A Self-Improving AI Agents · Autumn 2025  
> 录音对照：`raw-transcripts/4.learnfromfeedbacwithtoolscode.md`  
> 官网：[cs329a.stanford.edu](https://cs329a.stanford.edu/)

---

## 1. 一句话总览 / 目标位置

**本讲解决的问题**：L02–L03 解决了「多采样 + 如何验证」；要把 LLM 变成能在真实世界干活的 **agent**，还必须让它 **与环境交互并学习反馈**。本讲三条主线：

1. **ReAct**：Thought → Act → Observe 交错，把推理与工具调用接到一起；  
2. **RLEF**：用 **代码执行 / 测试反馈** 做多轮迭代，并用 RL（PPO）把「会用反馈改代码」训进模型；  
3. **Constitutional AI**：用人类写的原则（Constitution）驱动 **critique + revise**，再以 **RLAIF** 替代大规模人类偏好标注。

**统一主题**：三种技术都是 **self-improvement 反馈环**，差别在于 **反馈从哪来**：

| 方法 | 反馈来源 | 作用层面 |
|------|----------|----------|
| ReAct | 环境观察（搜索、网页、工具返回） | 多为 **prompt / 推理时** 闭环 |
| RLEF | 执行结果、单测 pass/fail | **训练时 RL** + **推理时多轮修复** |
| Constitutional AI | 模型按原则自我批判 / AI 偏好模型 | **SFT + RLAIF** 后训练对齐 |

**在课程地图中的位置**：

```text
L03 Robust Verification（如何判断对错）
  → L04 Feedback 来源多样化（工具 / 执行 / 宪法式自评）  ← 本讲
    → L05 Multi-step Planning（更长程分解与搜索）
    → L06 Train-time RL（系统化 RL 配方）
```

---

## 2. 核心概念与定义

### 2.1 ReAct 三元组

- **Thought（思考）**：自然语言推理轨迹；**不直接改环境**，只更新上下文，方便规划、纠错、决定下一步查什么。  
- **Act（行动）**：对外部环境的合法动作（如 `Search[Apple Remote]`、点击、提交答案）。  
- **Observe（观察）**：环境返回（检索结果、页面状态、错误信息）。  

循环形态（交错，而非「先想完再一口气做」）：

```text
Thought₁ → Act₁ → Obs₁ → Thought₂ → Act₂ → Obs₂ → … → Finish
```

对比：

| 范式 | 有推理？ | 有行动？ | 典型问题 |
|------|----------|----------|----------|
| Standard prompting | 弱 | 无 | 幻觉、无 grounding |
| Chain-of-Thought | 有 | 无 | 过程不接地气，幻觉可传播 |
| Act-only（如早期 WebGPT 式交互） | 弱 | 有 | 缺高层规划与工作记忆 |
| **ReAct** | 有 | 有 | 上下文长、动作空间大时示范不够 |

### 2.2 工具调用与 grounding

- **Grounding**：把生成锚定在外部信息 / 执行结果上，而不是仅靠参数记忆。  
- **有效动作空间**：常把下一步动作做成「在合法动作集合上分类 / 选择」，降低非法动作。  
- 今日模型：thinking 模式里常已内化 tool-call 痕迹（蒸馏自类似轨迹）；ReAct 仍是理解 agent workflow 的积木。

### 2.3 RLEF 关键术语

- **Execution feedback**：跑代码 / 跑测试得到的输出、报错、超时、失败用例信息。  
- **Public vs Private tests（两层测试）**：  
  - **Public**：迭代时可见，驱动多轮修复；  
  - **Private**：最终计分 / RL reward，训练时对模型「隐藏细节」以防背答案。  
- **Turn-level value / token-level policy**：策略仍按 token 生成；价值 / advantage 可在 **整轮回复** 上计算（课堂类比：接近对整段给 reward，而非逐步 token reward）。  
- **Binary reward**：最终私有测试全过 → 正奖励；否则负 / 零（可加非法代码小惩罚、KL 等）。

### 2.4 Constitutional AI / RLAIF

- **Constitution**：少量自然语言原则（课堂：约 **16** 条量级），定义期望行为。  
- **Critique**：按某条原则批评当前回复（有害？偏见？不宜儿童？…）。  
- **Revise**：按批评改写回复。  
- **SL 阶段**：在「修订后回复」上做 SFT。  
- **RL 阶段**：用 AI 反馈训 preference model，再 PPO/RL 对齐 → **RLAIF**（Reinforcement Learning from AI Feedback），对比经典 **RLHF**。

### 2.5 与 self-improvement 闭环的关系

抽象闭环（三篇论文共用）：

```text
产出候选行为 / 回复 / 代码
    → 获得反馈（观察 / 测试 / 原则批判）
        → 修正或强化策略
            → 更好的下一轮产出
```

- ReAct：**在语言上下文里即时闭环**（通常不更新权重）。  
- RLEF：**推理时多轮修 + 训练时用执行信号更新权重**。  
- CAI：**用原则生成合成偏好数据，再更新权重**；人主要写原则而非逐条偏好标注。

### 2.6 「反馈质量」比「有没有反馈」更重要

课上反复出现的隐含命题：

| 反馈类型 | 信噪比（典型） | 可规模化 | 备注 |
|----------|----------------|----------|------|
| 单元测试执行 | 高（若测试好） | 高 | RLEF 核心 |
| 搜索引擎 snippet | 中（噪声/矛盾） | 高 | 需 ReAct 推理过滤 |
| 单模型自我批判 | 中低（过自信） | 极高 | CAI；可用多模型共识加强 |
| 人类逐步偏好 | 高 | 低 | RLHF / 人工 PRM |
| 无 grounding 的 CoT | — | — | 不是外部反馈 |

设计 agent 时先问：**我的环境能否给出可自动消费的信号？** 若能，优先把闭环建在该信号上，再考虑 RL。

### 2.7 Agent vs「会聊天的 LLM」

课上收束：ReAct 已开始像 agent（观察 + 动作），但还缺会话记忆、长期状态等（后续讲）。本讲三块都是 **让 LLM 按某种 workflow 对外界做出反应**——这正是 L01 agent 定义里的 workflow 积木。
---

## 3. 方法 / 机制

### 3.1 ReAct（Yao et al., 2022）

**问题设定**：人类是「边想边做」；LLM 长期把 **reasoning** 与 **acting** 拆开——CoT 不接地，工具 agent 缺显式推理。天气/新闻类问题若无搜索必幻觉。

**方法**：Few-shot 提示冻结大模型（原论文 PaLM 等）交错生成 Thought 与 Act；环境写入 Observation。知识任务上动作空间可极简（Wikipedia search / lookup / finish）。

**示例（课堂 HotpotQA）**：  
问「除了 Apple Remote，还有什么设备能控制它最初设计交互的程序？」  
- 标准 / 纯 CoT：易错。  
- ReAct：Thought「先查 Apple Remote」→ Search → Obs 提到 Front Row → Thought「再查 Front Row」→ … →  grounded 终答。

**结果要点（课堂）**：

- **知识密集**（HotpotQA、FEVER）：ReAct > Act-only；相对纯 CoT 互有胜负（FEVER 上 ReAct 更好；HotpotQA 未必总赢）。  
- **ReAct + CoT-SC 回退 / 组合** 往往最好：**内生知识 + 外部检索** 互补。  
- 失败模式：CoT 主因是 **幻觉**；ReAct 因检索 grounding，幻觉更少、轨迹更可审计。  
- **决策任务**（WebShop）：ReAct 成功率可超过 imitation learning / IL+RL 基线；但仍低于人类专家（课堂：约 **66.6 vs 82.1**）；多步错误会级联，success rate 压力大。  
- Finetune / outer-loop（后续课）可进一步抬升。

**局限**：

- 动作空间很大时，few-shot 示范塞不进上下文。  
- 多步推理 → **推理成本 / 延迟**升高。  
- 环境噪声、矛盾检索结果需要额外置信度、回退、多数票、反思层（课堂讨论）。

**讨论扩展（课堂）**：任务分解、记忆、推理–行动比例、多模型分工、防 overthinking——都是后续 planning / agent 课的伏笔。

### 3.2 RLEF — Grounding Code LLMs in Execution Feedback（Meta, 2024）

**动机**：编码 agent（课堂：多数同学已在用 Claude Code 一类工具）必须理解用户意图，并基于「代码跑出来怎样」迭代。仅独立采样往往不如「带着反馈改」——但要 **真正会用反馈**，需要专门训练。

**框架（端到端 RL）**：

```text
自然语言题面
  → 生成代码（Turn 1）
  → 跑 Public tests → 反馈拼进对话
  → 再生成（Turn 2…）直到过 public 或达 turn 上限
  → 用 Private tests 给二元 reward
  → PPO 更新政策
```

两阶段直觉：

- **左：利用（inference-time）**：当前政策在多轮执行反馈下试错、修 bug。  
- **右：改进（training-time）**：用最终私有测试奖励更新政策 → 模型更会「读反馈并改代码」。

**两项课堂强调的设计**：

1. **Two-tier tests**：public 促迭代；private 定 reward → 降低「记住测试输出」捷径，并可用小 public 加速迭代。  
2. **Hybrid token–turn policy**：token 级生成 + **turn 级** value/advantage（整段一个优势信号）。

**现象与数字（课堂）**：

- 在 CodeContests 上，相对基线 **solve rate** 随采样预算（log 横轴）明显上移；可达 SOTA 同时 **样本需求降约一个数量级**（论文宣称）。  
- 基座模型「只看错误解+反馈」未必会用；**RLEF 训练后**才真正学会利用反馈，并泛化到 HumanEval+ / MBPP+ 等。  
- Turn 1→2→3：错误输出变少，修复更有针对性；对照无迭代环时编辑更瞎。  
- 反馈内容主要是失败信息，未必含「如何修」的建议；赛题代码规模通常不大（课堂：常 <~100 行）。  
- Ablation 讨论：binary reward 对很难题可能不够 → 需要更细的 error trace / 过程奖励（连回 L03 ORM vs PRM）。  
- Wrong output 变少、timeout 可能变多——域相关现象。

**与作业 / 后续**：作业会在数学等设定里练「找错→改」；SWE-bench / CodeMonkeys 等处理「仓库装不进上下文」——先检索/摘要相关代码，再补丁+测试（L13 方向）。

### 3.3 Constitutional AI（Anthropic, 2022）

**动机**：RLHF 要大量人类偏好标注，贵且慢；希望模型 **helpful 且 harmless**。Constitutional AI 用 **原则** 替代海量有害偏好标签。

**两阶段**：

**A. Supervised Constitutional AI**

1. 用 red-team 提示引出可能有问题的回复。  
2. Critique request（例：是否有害/不道德？是否性别偏见？是否不适于儿童？）。  
3. Revision request（删除/改写违规内容）。  
4. 在修订结果上 **SFT**。  
5. 多轮修订：harmlessness↑，单纯 helpfulness 可能↓，但「综合」可单调改善（课堂表述）。

**B. RL Constitutional AI (RLAIF)**

1. 采样成对回复，用 Constitution 驱动 AI 比较 → 训 **preference model**。  
2. 用该 PM 作 reward，RL 微调 LLM（更 thoughtful / respectful 等）。  
3. 可选 CoT 式评价，提高可解释性与人类评判表现。

**结果要点**：

- 相对 Helpful-only / HH 的 RLHF，**RL-CAI** 在 harmlessness–helpfulness Pareto 上更好；Claude 系强 harmlessness 的重要技术来源之一。  
- CAI+CoT 可进一步推 Pareto；harmlessness↑ 常与 helpfulness 有张力，需折中。  
- 后续工作：RLAIF vs RLHF 比较、Self-Refine、RL 教 self-correct 等。  
- 课堂提醒：自评可能过度自信 → **多模型共识批判** 有时更稳。

**开放问题**：改宪法后如何高效更新、如何「忘掉」旧规则（持续学习）；偏好模型仍需少量人类一致性校验。

### 3.4 三者如何拼进「Self-Improving Agent」

```text
          ┌─ 外部知识 / 工具 ──► ReAct 轨迹
目标任务 ─┼─ 可执行检验 ──────► RLEF / 单测反馈
          └─ 行为原则 ────────► Critique–Revise / RLAIF

再叠加 L03：对多条轨迹用 verifier / ensemble 做选择
再叠加 L02：重复采样与 test-time compute
```

工程上常见栈：

1. ReAct（或原生 tool-calling）收集 grounded 轨迹；  
2. 代码域用执行反馈做内环修复；  
3. 用原则 / rubric / PRM 做外环打分或偏好优化；  
4. 有奖励信号处上 RL（RLEF、PRM-PPO、RLAIF…）。

---

## 4. 关键数字与实验结论

| 主题 | 数字 / 结论 | 含义 |
|------|-------------|------|
| ReAct WebShop | ~66.6% success vs 人类 ~82.1% | 有头空间；多步级联误差 |
| ReAct vs IL/RL | 绝对成功率可 +10%～+34%（论文域） | 少样本提示可胜大数据模仿 |
| HotpotQA/FEVER | ReAct+CoT-SC 组合最佳 | 内外知识要结合 |
| RLEF | 样本预算可降 ~10× 量级仍更强 | 会用反馈比盲采样高效 |
| RLEF 泛化 | CodeContests 训练 → HumanEval+/MBPP+ | 执行反馈技能可迁移 |
| CAI 原则数 | ~O(10)（课上 16） | 极少「比特」的人类监督 |
| CAI Pareto | RL-CAI 优于标准 RLHF 折中 | 少人类有害标签仍可更安全 |
| SFT 修订轮数 | harmlessness↑ helpfulness 可能↓ | 需看联合目标 |

---

## 5. 课堂 Q&A 与易错点

### ReAct

1. **模型知道自己知不知道吗？** 校准差、常过度自信；应用重点是 **强制用对工具**，而非玄学「自知」。  
2. **交错还是「先列完所有 thought」？** **交错**：Thought₁–Act₁–Thought₂…，每步观察影响下一步。  
3. **检索矛盾怎么办？** 置信度、多数票、护栏、人类裁决；比纯参数幻觉仍可控。  
4. **Thought 算不算 action space？** 形式化可把语言思想当作不改环境的扩展动作；实现上常与环境动作分开两条环。  
5. **噪声环境？** 反思层、回退、重复尝试建置信、多次检索。

### RLEF

6. **Public/Private 从哪来？** 赛题测试集人工拆分；public 给推理环，private 给 reward。  
7. **谁判 pass/fail？** **真跑测试**，结果文本喂回模型。  
8. **Binary reward 够不够？** 简单题够；难题可能要堆栈/过程奖励（连 L03）。  
9. **是否鼓励「第一次就对」？** 推理环允许多 turn；奖励在最终私有测试——可调 turn 上限与 shaping。  
10. **为何泛化？** 学会的是「读失败信号并修补」的策略，不只背题。  
11. **仓库太大？** 先搜索/摘要/图检索相关片段（Claude Code、SWE-bench 思路），再进测试环。

### Constitutional AI

12. **改宪法要不要从头预训练？** 通常是后训练小比例算力；如何持续遵守新规仍是开放题。  
13. **SL 阶段有没有显式 reward？** 主要是对修订轨迹做似然；靠原则引导的数据分布偏移。  
14. **AI 反馈准不准？** Preference model 仍应用少量人类一致性集校验。  
15. **只做过滤数据能否代替 CAI？** 预训练过滤有帮助，但网上脏数据无法靠过滤清零，后训练原则仍关键。

### 总易错

16. **别把 ReAct 当成过时玩具**：搜索空间难定义的任务上，手写 workflow 仍关键；RL 也需要动作/奖励设计。  
17. **反馈信号质量决定闭环能否 hill-climb**——噪声反馈会放大错误轨迹。

---

## 6. 本讲论文速查

| 论文 | 笔记卡 |
|------|--------|
| ReAct | [papers/react.md](../papers/react.md) |
| RLEF | [papers/rlef.md](../papers/rlef.md) |
| Constitutional AI | [papers/constitutional-ai.md](../papers/constitutional-ai.md) |

（验证侧 ORM/PRM 见 [L03](L03-robust-verification.md)。）

---

## 7. 工程可落地 Checklist

- [ ] Tool-calling：交错 Thought/Act；动作集显式枚举；Observation 原样回灌。  
- [ ] 为幻觉高发任务加检索/执行 grounding；关键路径加 verifier（L03）。  
- [ ] 编码 agent：区分 **可见测试** 与 **隐藏验收**；防测试泄漏与过拟合。  
- [ ] 多轮修复：把 stderr、失败用例、超时写进上下文；限制 max turns。  
- [ ] 若有环境可自动打分：考虑 RLEF 式 PPO（注意 KL、非法输出惩罚）。  
- [ ] 对齐 / 安全：先写清 Constitution（原则可版本化）；跑 critique→revise→SFT；再 RLAIF。  
- [ ] 用少量人类标签校准 AI judge / preference model。  
- [ ] 监控 helpfulness–harmlessness 张力；防过度拒答。  
- [ ] 长仓库：检索 → 局部编辑 → 测试；不要假设全库进上下文。  
- [ ] 噪声工具输出：加反思、回退、重复采样与投票。  
- [ ] 记录完整轨迹（thought/act/obs 或修码对话）便于调试与日后 SFT/RL。

---

## 8. 若你只记 5 条

1. **ReAct = Thought–Act–Observe 交错**：推理规划行动，行动用观察接地，降低幻觉并提高可解释性。  
2. **反馈来源决定改进方式**：工具观察（ReAct）、执行测试（RLEF）、原则自评（CAI）都是自改进环。  
3. **RLEF** 把执行反馈同时用于 **多轮推理修复** 与 **PPO 训练**；public/private 测试分离是关键防捷径设计。  
4. **Constitutional AI**：人写原则 → critique+revise 做 SFT → AI 偏好做 RLAIF，用极少人类有害标签推进 harmlessness。  
5. **Self-improvement 成立的前提是反馈有足够信噪比**；再与 L02/L03 的采样与验证组合，才构成完整 agent 栈。

---

## 9. 反馈来源决策表

| 你能获得的信号 | 优先技术 | 典型坑 |
|----------------|----------|--------|
| 可调用 API / 浏览器 / 检索 | ReAct / 原生 tool-call | 非法动作、噪声 obs、死循环 |
| 单元测试 / 解释器 | RLEF 式多轮 +（可选）PPO | 测试泄漏、奖励稀疏、超时 |
| 清晰行为原则、要对齐风格/安全 | CAI：critique–revise → RLAIF | 原则冲突、过度拒答、AI 偏见 |
| 数学终答 checker | 回 L03 ORM/PRM | 过程错终答对 |
| 以上都没有 | 先设计可自动的 proxy 奖励/测试 | 人工逐步偏好极贵 |

## 10. 伪流程速查

### 10.1 ReAct 一步

```text
context = [task, few_shot_demos]
loop until Finish or max_steps:
  thought = LLM(context + "Thought:")   # 不调用环境
  action  = LLM(context + thought + "Act:")
  if action is Finish: return answer
  obs = env.step(action)
  context += thought + action + obs
```

### 10.2 RLEF 一轮训练 episode

```text
dialog = [problem]
for turn in 1..T:
  code = policy.sample(dialog)
  public_fb = run(public_tests, code)
  dialog += code + public_fb
  if public_all_pass: break
reward = 1 if run(private_tests, code).all_pass else 0
# (+ KL to ref, penalty if invalid code)
PPO_update(policy, trajectory, reward)
```

### 10.3 Constitutional AI

```text
# SL
for prompt in red_team:
  y0 = model(prompt)
  for principle in sample(constitution):
    critique = model(prompt, y, principle, "critique")
    y = model(prompt, y, critique, "revise")
  SFT_buffer.add(prompt, y)

# RL
pairs = sample_pairs(model_SFT)
pm = train_preference_model(AI_labels(pairs, constitution))
RL_finetune(model_SFT, reward=pm)
```

## 11. Self-improvement 闭环总图（本讲收束）

```text
                    ┌─────────────────────────────┐
                    │     Self-Improving Loop     │
                    └──────────────┬──────────────┘
           产出候选（文本/代码/动作）
                    │
     ┌──────────────┼──────────────┐
     ▼              ▼              ▼
  工具观察       执行测试        原则批判
  (ReAct)        (RLEF)         (CAI)
     │              │              │
     └──────────────┼──────────────┘
                    ▼
         修正上下文 或 更新权重
                    │
                    ▼
         （可选）L03 Verifier 选优
                    │
                    ▼
              更好的下一轮产出
```

**成立条件（课上金句意译）**：反馈环必须有足够信号；有信号就能在训练数据之外继续 hill-climb。

## 12. 与前后讲的接口

| 接口 | 内容 |
|------|------|
| ← L02 | 多采样预算可套在 ReAct 轨迹或代码候选上 |
| ← L03 | 执行 pass/fail ≈ ORM；逐步单测/lint ≈ 弱 PRM；PM/Judge ≈ verifier |
| → L05 | 任务分解、树搜索、多步规划扩展 ReAct 的线性交织 |
| → L06 | PPO/GRPO 等系统化；RLEF 是代码域实例 |
| → L13 | SWE-bench、CodeMonkeys：检索 + 补丁 + 测试 |

## 13. 课上开放问题 / 项目点子

1. 噪声或误导性环境反馈下的反思、回退、置信度。  
2. 推理–行动最优比例（防 overthinking）。  
3. 复合系统：子任务路由到不同模型。  
4. Binary reward 不够时：error trace / 过程奖励对比（接 ORM vs PRM）。  
5. Public/Private 拆分的消融与泄漏分析。  
6. 多模型共识批判 vs 单模型自评。  
7. ReAct 手写 workflow 与纯 RL 探索在「搜索空间难定义」域上的边界。  
8. KV-cache 上 Thought vs Observation 的表征差异（课上当研究题抛出）。

## 附：三篇论文对照表

| | ReAct | RLEF | Constitutional AI |
|--|-------|------|-------------------|
| 主领域 | QA / 决策 / 工具 | 竞赛编程与代码 | 对话安全与对齐 |
| 是否更新权重 | 原版主要为 prompting | 是（PPO） | 是（SFT+RL） |
| 反馈 | 环境 obs | 测试执行 | 原则驱动的 AI 反馈 |
| 与 L03 关系 | 可对多轨迹再验证 | 二元执行 ≈ 可编程 ORM | Judge/PM ≈ 另一种 verifier |
| 今日形态 | 原生 function call / thinking | 编码 agent 内环 | 宪法/政策 + RLAIF 变体 |
| 人类成本 | 写示范轨迹 | 设计测试拆分 | 写 Constitution + 少量校准 |

## 附：自学路径建议

1. 手写一条 HotpotQA 的 ReAct 轨迹（含一次检索失败再改 query）。  
2. 本地对一道编程题模拟 public/private：只把 public 失败信息给模型再生成。  
3. 写 5 条「迷你宪法」，对同一有害请求做 critique→revise 两轮。  
4. 画一张图：把 L02 采样、L03 验证、L04 三种反馈画进同一 agent 环。  
5. 阅读后续 L05，看 ReAct 如何扩展成树搜索 / 任务分解。  
6. 对照第 9 节决策表，为自己的应用选「主反馈通道」并列出失败模式。
