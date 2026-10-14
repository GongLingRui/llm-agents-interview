# L13 — Agentic Frameworks for Software Engineering

> **来源**：**无录音 / 论文整理**（仓库无本讲录音稿；依据 CS329A 官网必读论文 + arXiv）  
> **日期**：Mon Nov 3（Autumn 2025）  
> **主题**：把 test-time compute、执行反馈与 agent–系统接口用到真实软件工程与高性能计算  
> **必读**：CodeMonkeys · KernelBench · LLM Optimizers via Agent-System Interfaces

---

## 1. 一句话总览

软件工程 agent 的关键不只是「会写补丁」，而是把问题拆成 **Context → Generation → Selection**，并显式缩放 **串行迭代（执行反馈）** 与 **并行多样本**；在 GPU kernel 与并行程序 mapper 上，**可执行反馈 + 结构化接口（DSL / profiler）** 比纯标量 reward 的黑盒搜索更有效。

---

## 2. 核心概念与定义

### 2.1 为何 SWE 是 self-improving agents 的主战场

- **天然 verifier**：测试套件、编译器、benchmark、profiler  
- **长上下文**：百万 token 级仓库 vs 有限窗口  
- **经济价值高**：与 L17 GDPVal / METR 中软件任务强项一致  
- **可组合 test-time scaling**：与 L02 Large Language Monkeys 同一作者线（CodeMonkeys）

### 2.2 三类问题原型

| 原型 | 论文 | 成功信号 |
|------|------|----------|
| 修真实 GitHub issue | CodeMonkeys / SWE-bench Verified | 仓库官方测试通过 |
| 写正确且快的 GPU kernel | KernelBench | 功能正确 + 相对 PyTorch 加速比 |
| 优化并行程序 mapping | ASI + generative optimization | 端到端运行时间 / 吞吐 |

### 2.3 串行 vs 并行 test-time compute（贯穿本讲）

- **串行**：同一轨迹上多轮 edit ↔ run tests / profile  
- **并行**：多样本 / 多轨迹覆盖（coverage）  
- **选择**：有 coverage 不够，必须能从候选里挑对的（selection bottleneck）

---

## 3. 方法 / 机制

### 3.1 CodeMonkeys：为「可缩放 test-time」重设计的 SWE 系统

**论文**：*CodeMonkeys: Scaling Test-Time Compute for Software Engineering* (arXiv:2501.14723)

#### 三阶段分解

1. **Context**：找出需要改的文件并塞进窗口  
2. **Generation**：产生候选 codebase edit（最好带测试）  
3. **Selection**：从候选中选出最终提交  

指标对应：

- Context → **recall**（所需文件是否都在窗口里）  
- Generation → **coverage**（是否至少有一个正确 edit）  
- Selection → **score**（提交解决率）

#### Context：摊销后的「读遍仓库」

- SWE-bench 仓库常达百万 token → 不能每次用前沿模型硬吃全文  
- 因计划对每 issue 生成**多个**下游 edit，可把「扫描全库」成本 **amortize**  
- 做法：用便宜模型（如 Qwen2.5-Coder-32B）扫每个 Python 文件标相关性 + 摘要；再用前沿模型 **ranking** 压到 ~128k（平均约 50× 压缩）  
- 128k 限制下 recall 约 **92.6%**（论文）  
- 摊销后 relevance 扫描约占总费用 **~15%**；若不摊销、对每个样本重扫，会变最贵一步  
- 共享 context 也提高 **prompt cache** 命中

#### Generation：双状态机 + 自写测试

抽象为 state machine（继承 Moatless 等思想）：

1. **Testing SM**：先在**未修改**代码库上迭代写出测试脚本  
2. **Editing SM**：基于 context + 测试草稿写 edit；在「改前 / 改后跑测试」的反馈上同时改测试与补丁  
3. 每 issue 并行跑多条轨迹（如 **10** 个 (edit, test)）  

缩放旋钮：

- 每轨迹迭代次数（串行）  
- 每问题轨迹数（并行）  

发现：前几轮串行迭代收益大；之后**相近预算下不同串并行配置 coverage 相近**（与 Monkeys 的 log-linear coverage 叙事一致）。组合缩放可达 SWE-bench Verified **~69.8% coverage**。

#### Selection：测例投票 + 多轮甄选

- 用模型生成的测试做投票，滤出通过最多的候选  
- 再用 **Selection SM**：写区分性测试，在 top 候选上执行，决定继续写测试或提交  
- 单独 majority vote 不够；「测试过滤 + 模型精选」最好  

#### 结果与成本（工程必记）

- CodeMonkeys：**57.4%** SWE-bench Verified，推理花费约 **$2300**  
- **Barrel of Monkeys**：把 leaderboard top 提交的 edit 与自身候选混合（coverage **80.8%**），同一套 selection → **66.2%**，超过 ensemble 内最佳单项（62.8%），接近当时 o3 报道分  
- 含义：selection 方法可跨**异构来源**复用——系统工程价值不低于单次生成

#### 工程要点

- 强制「补丁与测试一起长」→ 执行反馈密度远高于只看最终 fail/pass  
- 为并行缩放而设计 context（摊销、缓存）  
- 发布代码与样本便于复现 scaling 曲线  

---

### 3.2 KernelBench：LLM 能否写出高效 GPU Kernel？

**论文**：*KernelBench: Can LLMs Write Efficient GPU Kernels?* (arXiv:2502.10517；Stanford / Mirhoseini 等)

#### 设定

- **250** 个精选 PyTorch ML workload  
- 输入：PyTorch 参考实现；输出：自定义 GPU kernel  
- 真实工程环境：正确性 + 性能同时重要  

#### 指标 \(\mathrm{fast}_p\)

\[
\mathrm{fast}_p = \text{既正确且相对 PyTorch baseline 加速比} > p \text{ 的任务占比}
\]

- \(p=1\)：至少打平 Eager  
- 提高 \(p\) 难度陡增  

#### 主要结论

- 前沿推理模型零样本最好，但整体仍弱：打平 PyTorch 的任务 **< ~20%**  
- 失败模式：执行错误、功能不正确、不会做平台相关优化  
- **迭代 + 执行 / profiler 反馈** 可大幅提升（例：DeepSeek-R1 某 Level 从 ~36% → ~72%）  
- 仍是难基准；与「只生成能跑的代码」不同，要 **fast and correct**

#### 与课程连接

- L04 / L03：execution feedback 即强 verifier；但性能反馈是**更稠密 / 更难解释**的 reward  
- L17：CUDA kernel 类任务出现在长时程套件（人类可达数小时）  
- L20：课上点名 KernelBench——编译可验证，但读 profile、改非热点更难，常需问题分解  
- 后续社区出现 KernelBench-Verified 等，提醒注意 baseline（如 TF32）与 reward hacking——评测设计本身要 robust（呼应 L03 / L17）

---

### 3.3 Agent-System Interfaces：用 DSL + AutoGuide 做生成式系统优化

**论文**：*Improving Parallel Program Performance with LLM Optimizers via Agent-System Interfaces* (arXiv:2410.15625)

#### 问题

科学计算 / HPC 中，task-based runtime（Legion 等）把性能关键决策因子化为 **mapper**（任务→处理器、数据→内存）。手写 mapper 要数天、强系统功底，领域科学家做不了。

#### Agent-System Interface (ASI)

两块：

1. **DSL**：挡住底层系统 API；编码所有性能关键映射决策；定义**结构化搜索空间**；编译生成真实 mapper 代码  
2. **AutoGuide**：把原始执行日志译成**可执行的自然语言指导**（错误解释、建议），而非只给标量 runtime  

#### Generative optimization vs 经典 autotune

| | OpenTuner 类 RL | LLM generative optimization |
|--|-----------------|------------------------------|
| 反馈 | 标量性能 | 标量 + 自然语言解释 / 建议 |
| 搜索 | 黑盒 | 在 DSL 空间内迭代改写 |
| 样本效率 | 需大量 trial | **10 iter** 可超过 OpenTuner **1000 iter**（论文：约 3.8× 性能优势口径） |

#### 结果

- 跨 9 个 benchmark，可超过专家手写 mapper，最高约 **1.34×**  
- 调优时间：天 → 分钟  
- Ablation：去掉 DSL / AutoGuide 会显著变差 → **接口设计是一等公民**

#### 思想提升（对本课）

> Agent 强不强，不只取决于基座模型，还取决于**系统暴露给 agent 的接口是否可搜索、反馈是否可解释**。

这与 ReAct 工具设计、SWE 里「给模型 bash + 测试」同一谱系，只是把接口抬到 DSL + 领域反馈编译器。

---

## 4. 三篇论文对照

| 维度 | CodeMonkeys | KernelBench | ASI Optimizers |
|------|-------------|-------------|----------------|
| 任务 | 修 issue | 写 kernel | 写/优 mapper |
| 主反馈 | 测试执行 | 正确性 + 速度 | 运行时 + AutoGuide 文本 |
| 缩放 | 串行迭代 × 并行轨迹 | 迭代 refinement | 生成式多轮优化 |
| 接口创新 | 测试–编辑双 SM；摊销 context | 统一 fast_p | DSL + AutoGuide |
| 选择问题 | 显式 selection SM | 常取最快正确 | 迭代保留更优 mapper |

---

## 5. 与课程其他讲的连接

| 讲次 | 连接 |
|------|------|
| L02 Monkeys | CodeMonkeys 是 SWE 上的 scaling 实例化；coverage log-linear |
| L03–L04 | 测试 / 编译 / profile = verifier；RLEF 精神 |
| L05 Planning | 状态机 = 结构化多步计划 |
| L08 Search | 并行样本 + 选择 ≈ 搜索 |
| L14 Memory / Cache | 仓库级 context、prompt cache 摊销 |
| L17 Eval | SWE-bench 偏见；长时程 CUDA；专家 vs contractor |
| L20 | 可验证域飞轮；IPW / kernel 能效 |

---

## 6. 工程可落地 checklist

- [ ] 把 SWE agent 拆成 context / generate / select，分指标监控  
- [ ] 生成阶段强制「补丁 + 复现测试」同源迭代  
- [ ] 扫描仓库用便宜模型，前沿模型预算留给 edit / select  
- [ ] 画清 coverage–budget 与 score–budget 两条曲线，避免只报 coverage  
- [ ] Kernel / perf 任务：接入 profiler 文本，不只 pass/fail  
- [ ] 对系统优化：先投 DSL / 工具层，再堆采样次数  
- [ ] Ensemble 异构来源 + 统一 selection（Barrel of Monkeys 模式）  
- [ ] 成本表按阶段拆开（relevance / gen tests / gen edits / selection）  

---

## 7. 本讲论文速查

| 论文 | 卡片 |
|------|------|
| CodeMonkeys | [papers/codemonkeys.md](../papers/codemonkeys.md) |
| KernelBench | [papers/kernelbench.md](../papers/kernelbench.md) |
| ASI LLM Optimizers | [papers/asi-llm-optimizers.md](../papers/asi-llm-optimizers.md) |

---

## 8. 若你只记 5 条

1. SWE agent = **Context + Generate + Select**；三者指标不同。  
2. 为并行缩放而**摊销**全库扫描；共享 context 便宜且利于缓存。  
3. **自写测试 + 执行反馈** 是串行 scaling 的燃料。  
4. KernelBench：正确且快远难于「能跑」；profiler 迭代关键。  
5. 系统场景要先做好 **Agent–System Interface**（DSL + 可解释反馈），再比拼优化器。

---

## 9. 无录音说明

本笔记依据官网阅读列表与论文正文整理，**未包含课堂板书 / 口播数字的额外口径**。若日后补录音，优先核对：课上复现的 SWE-bench 数字、KernelBench 现场 demo、以及 ASI 与课上项目的对应关系。
