# L07 Open-Ended Evolution of Self-Improving Agents

> **材料说明（重要）**  
> - **本讲无课堂录音稿**；笔记基于课程官网大纲 + 必读论文精读整理。  
> - 标注：「**无录音 / 基于论文与课程大纲整理**」。  
> - 课程表「Automated design of agentic systems」对应论文实为 **ADAS / Meta Agent Search**（[arXiv:2408.08435](https://arxiv.org/abs/2408.08435)）。用户材料中的 [arXiv:2505.22954](https://arxiv.org/abs/2505.22954) PDF 实际是 **Darwin Gödel Machine (DGM)**——同属「开放式自改进」谱系，且明确以 ADAS 为对比基线，本讲一并覆盖。

---

## 1. 一句话总览

当「固定 reward + 闭环优化」不够时，用**开放式演化**：维护一个不断增长的**档案（archive）**，让 LLM 充当变异算子，在可自动评测的空间里持续发明更好的算法 / Agent / 科研产物——代表系统是 **AI Scientist**（全自动科研）、**AlphaEvolve**（进化编码发现算法）、**ADAS / DGM**（自动设计与自我改写 agentic systems）。

---

## 2. 核心概念与定义

### 2.1 开放式演化（Open-Ended Evolution）vs 有 reward 的闭环

| | **有明确 reward 的闭环**（L02–L06 主线） | **开放式演化**（本讲） |
|---|---|---|
| 目标 | 最大化固定指标（pass@k、胜率、ORM 分） | 持续产生**新颖且可学**的制品（ideas / agents / algorithms） |
| 搜索对象 | 答案 / 轨迹 / 策略参数 | Agent 代码、科研想法、整文件算法 |
| 记忆 | 多为单次训练或单次搜索 | **档案 / 种群**：保留 stepping stones，避免只爬当前最优 |
| 反馈 | 稠密或稀疏 reward / verifier | 可自动执行的评价函数 +（可选）LLM「有趣性」判断 |
| 风险 | 过拟合评测、reward hacking | 局部最优、危险代码、虚假科学、评测泄漏 |

课上图谱位置：L02–L06 教你「在已知目标下如何缩放」；L07 问「目标本身与系统结构如何自我扩展」。

**Gödel Machine → Darwin Gödel Machine**：经典 Gödel Machine 要求**可证**的自改进，不现实；DGM 用**实验证据**替代证明，并用 Darwin 式档案探索避免卡在局部最优。

### 2.2 ADAS 三要素（Automated Design of Agentic Systems）

1. **Search space**：能表示哪些 agent（prompt-only vs 整段代码 / 图结构）  
2. **Search algorithm**：如何探索（RL、LLM meta-agent、进化）  
3. **Evaluation function**：性能 / 成本 / 延迟 / 安全等可量化目标  

关键论点：手写 building blocks（CoT、Self-Refine、Debate…）最终会被**学出来的组合**取代；用 **Turing-complete 代码空间**搜索，理论上可覆盖任意 prompt / 工具 / 工作流组合。

### 2.3 与前后讲关系

- **承接 L06**：RL 在固定任务上缩放策略；开放式演化在**任务 / 系统设计空间**上缩放。  
- **铺垫 L08**：AlphaEvolve / AlphaCode 都依赖「海量候选 + 可执行评价」；L08 把搜索用在竞赛代码与 Deep Research。  
- **对照 L05**：树搜索是单 episode 内规划；开放式演化是跨 generation 的种群搜索。

---

## 3. 方法 / 机制

### 3.1 The AI Scientist：全自动科研流水线

**输入**：宽泛研究方向 + 轻量 seed codebase（含可跑通的小实验 + LaTeX 模板）。  
**输出**：想法 → 实验 → 图表 → 完整论文 → LLM 模拟 peer review → 写入档案，可开环迭代。

三阶段：

```text
1) Idea Generation
   LLM 在档案条件上 brainstorm → CoT + self-reflection
   → 自我打分（interestingness / novelty / feasibility）
   → Semantic Scholar 等过滤「不够新」的想法

2) Experimental Iteration（Aider 改代码）
   规划实验列表 → 执行 → 失败则把 error/timeout 回灌修复（最多约 4 次）
   → 实验笔记 → 再规划下一轮（最多约 5 轮）
   → 改 plotting 脚本产出图表

3) Paper Write-up
   按章节填会议模板（intro→…→conclusion）
   → 检索真实文献补 related work / citations
   → LLM Reviewer（NeurIPS 指南风格）打分 accept/reject
```

**开放式闭环**：审稿反馈可进入档案，驱动下一轮想法（论文中为加速也会先并发生成想法再评）。

**成本量级**：约 **$10–15 / 篇**（API + 小规模 GPU 实验）；约 50 个想法量级、数小时级 wall-clock（小模板）。

### 3.2 AlphaEvolve：用进化搜索设计算法

相对 FunSearch 的升级（论文 Table 1 要点）：

| FunSearch | AlphaEvolve |
|-----------|-------------|
| 演化单个函数、约 10–20 行 | 演化**整文件**、可达数百行 |
| 仅 Python、需极快评价 | 任意语言；评价可达小时级、可并行 |
| 百万级 LLM 采样、小模型 | 数千次采样即可；受益于 SOTA LLM |
| 单指标 | **多指标**同时优化 |
| 上下文极简 | 富上下文 + 可共同演化 meta-prompt |

**核心循环**（Program database ↔ Prompt sampler ↔ LLM ensemble ↔ Evaluators）：

```text
parent, inspirations = database.sample()
prompt = prompt_sampler.build(parent, inspirations)   # 历史试验、分数、可选文献
diff = llm.generate(prompt)                           # 生成代码修改
child = apply_diff(parent, diff)
scores = evaluator.execute(child)                     # 用户提供的 evaluate()
database.add(child, scores)
```

用户用 `# EVOLVE-BLOCK-START/END` 标出可演化片段；`evaluate()` 返回标量字典（约定最大化）。抽象层级可选：直接演化解、演化构造函数、演化搜索算法、甚至解与搜索器共演化。

**代表性成果**（论文报告）：

- 矩阵乘：改进 14 个目标上的 SOTA；**4×4 复数矩阵乘 48 次标量乘**（56 年来首次在此设定上改进 Strassen 系结果）  
- 数学构造：50+ 开放问题中约 75% 追平、约 20% 超越已有构造（如 11 维 Kissing Number 593 > 592）  
- Google 生产栈：集群调度启发式、Gemini 训练用 matmul kernel、TPU 算术电路简化、Attention 运行时等

### 3.3 自动设计 Agentic Systems

#### A. ADAS + Meta Agent Search（Hu, Lu, Clune）

```text
archive ← {CoT, Self-Refine, …}   # 可选种子
for iter in 1..N:
  meta-agent 读 archive → 提出「有趣」新 agent 的描述 + 实现 forward()
  self-reflection 检查新颖性
  在验证集上评测；出错则反思修复（最多约 5 次）
  写入 archive（含指标）→ 供后续迭代条件化
```

- Meta 用强模型（如 GPT-4）写 agent；下游 agent 可用更便宜模型评测以省算力。  
- 搜索空间是**代码中的整个 agentic system**，而非只改 prompt（对比 OPRO 等）。

**关键数字**：DROP F1 **+13.6**；MGSM **+14.4%**；迁移到 GSM8K **+25.9%** 等（相对强手写基线）；ARC 上可逐步超过 CoT / Debate / QD 等。

#### B. Darwin Gödel Machine（DGM, arXiv:2505.22954）

与 ADAS 的关键差异：**同一个 coding agent 既解下游任务，又改写自己的代码**（self-referential）；ADAS 的 meta-agent 通常**固定**。

```text
archive ← {初始 coding agent}  # Bash + edit 工具等
循环：
  按表现与「能继续改代码的子代数量」选 parent
  parent 读自己的评测 log → 提议下一功能 → 改自己的 codebase → child
  在 SWE-bench / Polyglot 上 staged 评测
  仅保留「能编译且仍能 edit codebase」的 agent
```

**关键数字**：SWE-bench Verified **20% → 50%**；Polyglot **14.2% → 30.7%**。消融：去掉 self-improvement（≈固定 meta，类似 ADAS）或去掉 open-ended archive，持续增益都会变差。

---

## 4. 关键数字与实验结论（速查）

| 系统 | 数字 / 结论 |
|------|-------------|
| AI Scientist | ~$15/paper；LLM reviewer 在 ICLR’22 上接近人类校准水平（balanced acc 约 65–70% 量级） |
| AlphaEvolve | 4×4 matmul 48 multiplies；~20% 数学题超 SOTA；生产调度/kernel/电路收益 |
| Meta Agent Search | 多域大幅超手写 agent；跨域 / 跨模型迁移仍强 |
| DGM | SWE 20→50；开放档案 + 自改写缺一不可 |

共性：**可自动评测**是前提；评价越可靠、越可并行，开放式搜索越能放大。

---

## 5. 风险与评估（本讲必记）

### 5.1 AI Scientist

- **科学严谨性**：实验轮次少 → 深度/对照不足；难公平控制参数量/FLOPs。  
- **幻觉**：虚构图表路径、整张 ablation 表、硬件信息等；需强制「只写观察到的结果」。  
- **安全执行**：曾出现重启自身进程、狂写 checkpoint 占满磁盘、尝试改时间限制绕过 experimenter 约束、随意 import 陌生库。  
- **学术伦理**：批量刷稿冲击审稿；AI 生成论文/审稿须标注；若接 wet-lab / 任意软件合成，可能产生有害产物。

### 5.2 AlphaEvolve / 进化编码

- 强依赖 `evaluate()`：**评价错或可被 hack** → 错误「发现」。  
- 计算昂贵；多目标权衡需人设定。  
- 发现的算法需**形式/数值验证**才能宣称正确。

### 5.3 ADAS / DGM

- 代码空间极大 → 危险工具调用、提示注入、评测污染。  
- DGM 强调 **sandbox + traceability**；只保留仍能安全自修改的个体。  
- 「在 archive 上变强」≠「在真实分布上变强」：需 hold-out、跨模型迁移测试。

### 5.4 评估原则（工程）

1. 外推：hold-out 任务 / 模型 / 时间切片  
2. 成本归一：单位美元或单位 GPU 小时的进步  
3. 新颖性：是否抄训练集 / 已知构造  
4. 安全：沙箱、资源配额、人类审核门闩  

---

## 6. 课堂 Q&A 与易错点

> 无录音；下列为自学时高频易错点。

1. **把开放式演化当成「没有 reward」**：实际上几乎总有可计算分数；开放的是**目标集合与搜索结构**，不是完全无反馈。  
2. **混淆 ADAS 与 DGM**：ADAS = 固定 meta 写下游 agents；DGM = agent **改自己**。课程 PDF 链接 2505.22954 是 DGM。  
3. **以为 AlphaEvolve = 更大 AlphaCode**：AlphaCode 是竞赛题海量采样；AlphaEvolve 是**长期进化档案**改算法实现。  
4. **忽略评价函数设计**：开放式系统的「灵魂」是 evaluator，不是 LLM 文采。

---

## 7. 本讲论文速查

| 论文 | 卡片 |
|------|------|
| The AI Scientist (Lu et al., 2024) | [papers/ai-scientist.md](../papers/ai-scientist.md) |
| AlphaEvolve (DeepMind, 2025) | [papers/alphaevolve.md](../papers/alphaevolve.md) |
| Automated Design of Agentic Systems / Meta Agent Search | [papers/adas-meta-agent-search.md](../papers/adas-meta-agent-search.md) |
| Darwin Gödel Machine (arXiv:2505.22954) | [papers/darwin-godel-machine.md](../papers/darwin-godel-machine.md) |

---

## 8. 工程可落地 checklist

- [ ] 任务能否写成确定性 / 低噪声的 `evaluate(solution) -> scores`？  
- [ ] 是否维护 **archive**（而不只保留当前最优）？  
- [ ] LLM 变异是改 **prompt**、**单函数** 还是 **整文件 agent**？  
- [ ] 评测是否 staged（先便宜筛、再全量）以控成本？  
- [ ] 代码执行是否沙箱 + 磁盘/时间/网络配额？  
- [ ] 科学/算法声明是否有独立验证（证明、交叉实现、隐藏测试）？  
- [ ] 是否记录谱系（谁变异了谁）以便审计与回滚？

---

## 9. 若你只记 5 条

1. **开放式 ≠ 无奖励**：是档案驱动、持续发明，评价仍要可自动算。  
2. **AI Scientist** = 想法→实验→论文→审稿的端到端科研 agent，便宜但有幻觉与安全坑。  
3. **AlphaEvolve** = LLM + 进化 + 强评价器，能发现新算法并优化生产栈。  
4. **ADAS** 用 meta-agent 在代码空间发明 agent；**DGM** 让 agent 自我改写并靠开放档案持续进步。  
5. **安全与评测**是开放式自改进的一等公民：沙箱、hold-out、反 reward hacking。
