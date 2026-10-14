# L06 — Train-time Scaling / Scaling RL

> 课程：Stanford CS329A · Self-Improving AI Agents（Autumn 2025）  
> 材料：`raw-transcripts/6.traintimescalingRL.md` + 官网必读论文  
> 相关讲次：L02–L05（test-time / 验证 / 工具 / 多步规划）→ **本讲把成功轨迹写进权重** → L07/L08（开放进化、搜索与 deep research）

---

## 1. 一句话总览

**Train-time scaling** = 用可验证信号过滤模型自己的输出，再 SFT/RL 回去，让更小的模型在推理基准上逼近甚至超过更大模型；本讲路径是 **STaR（rationale bootstrapping）→ DeepSeekMath/GRPO（去掉 critic 的组相对 RL）→ DAPO（大规模长 CoT RL 的工程稳定化）**，并明确 **train vs test compute 取舍** 与 **reward hacking / 验证器依赖**。

---

## 2. 核心概念与定义

### 2.1 三条训练范式（课上对照）

| 范式 | 做什么 | 是否改参数 |
|------|--------|------------|
| Pre-training | 互联网规模 next-token | 是（从零/继续预训练） |
| Post-training（SFT / RLHF / RLAIF） | 偏好与指令 | 是 |
| **Test-time scaling** | 多样本、投票、搜索、验证器选解 | **否** |
| **Train-time scaling** | 把「采样+验证」得到的好轨迹再训回去 | **是** |

**本讲必须记住的闭环**（讲师原话级要点）：

```text
生成 → 用验证器/答案过滤 → 微调或 RL → 再生成 → …
```

若整堂课只带走一件事：这就是 train-time scaling。

### 2.2 课堂动机数字（AIME 等）

| 系统（课上口述） | 规模感 | AIME 相关表现（口播） |
|------------------|--------|----------------------|
| GPT-3.5（据称 ~175B） | 很大 | ~**5%** |
| DeepSeekMath | **7B** + train-time | **51.7%**；再加技巧可至 ~**60%** |
| DAPO on Qwen-32B | 中等开源 | ~**50%** |

解读：**参数不是唯一轴**；在可验证域上，把 compute 花在「学自己的正确推理」可以极大抬升小模型。

### 2.3 为何「推理模型」在可验证域更赚

Thinking / 长 CoT 把 token 预算分给：

1. **问题分析**（先弄清输入输出格式）  
2. **任务分解**  
3. **自我评估 / 回退（backtrack）**  
4. **多思路探索**

课上对比：编程、数据分析、数学计算等相对 GPT-4o（非 thinking）** win rate > 50%**；个人写作 / 文本编辑增益小——**有外部可验证闭环的域**更吃 train-time scaling。

### 2.4 Train-time vs Test-time：角色不同

| | Test-time | Train-time |
|--|-----------|------------|
| 作用 | 在已有能力上 **搜索** | 提高 **pass@1** 直接发射正确轨迹的概率 |
| 成本形态 | 每次请求多推理；易水平扩展 | 一次性（或阶段性）训练贵，推理可更省 |
| 依赖 | 好验证器才能「无限采样捡对的」 | 需要 **非零成功率** 才能 hill-climb |
| 比喻（课上） | 往墙上扔意大利面，知道该粘哪 | 教模型更常直接扔对 |

**预算怎么分？** 课上明确：没有普适 ROI 曲线；o1 的 AIME 图显示 **两边加 compute 都涨**，但是否「test 永远比 train 更划算」——讲师提醒 **图可能有问题 / 只是一次实例化**，不要当成定律。

### 2.5 与验证课（L03）的硬绑定

- Math / code：答案、执行、单测 → 强奖励。  
- 写作：弱验证 → train-time RL 易 **reward hacking** 或无信号。  
- 奖励模型太弱 → 爬不动；太容易被骗 → 策略钻空子。

---

## 3. 方法 / 机制

### 3.1 STaR — Bootstrapping Reasoning with Reasoning

**痛点**：

- 互联网很少大规模逐步推理文本  
- 人工标 rationale 极贵  
- 按模板自动生成只覆盖窄域  
- 仅 few-shot rationale **不如** 大规模微调

**算法骨架**：

```text
输入：小 seed 集 R = {(题, rationale, 答)}；大训练集 D = {(题, 答)}
重复 outer iteration:
  1) 用 R few-shot，对 D 中题目生成 (rationale, 答)
  2) 答对的 → 收入训练池（正向 bootstrapping）
  3) 答错的 → 给出正确答案作 hint，让模型「倒推」rationale（rationalization）
  4) 用 (题, rationale, 答) 微调；训练时 **不暴露 hint**
  5) 用新模型进入下一轮
```

可视为极简 **off-policy / 过滤式** 自改进，不是完整 online RL。

#### 三个关键假设

1. **终局正确 ≈ rationale 质量尚可**（数学上大致成立，但不完美）。  
2. **给答案后模型能写出有效推导**。  
3. **基座对目标分布已有非零可解子集**——否则无 hill-climb 信号。

#### Rationalization 的意义

只训「已经会的」→ 难学新题。  
对失败题给答案再生成 rationale → **扩展难题覆盖**，提高样本效率。

#### 变体（课上一带）

| 变体 | 想法 |
|------|------|
| **V-STaR** | 生成器 + 验证器一起转 |
| **Quiet-STaR** | 推理进隐空间 / MLP，不必全用自然语言 |

#### 实现细节（口播）

- 模型：开源 GPT-3 类 **GPT-J ~6B**  
- 小 warm-up、恒定 LR、多轮 outer loop；inner 步数逐渐加大（慢启动）  
- 数据：GSM8K（~9k）、CommonsenseQA、多位数加法等合成算术

### 3.2 DeepSeekMath + GRPO

#### 能力先于 RL（课上强调）

DeepSeekMath 成功不只靠 RL：

1. **从 code 基座出发**（DeepSeek-Coder）→ 数学与工具推理迁移更好；课上称 arXiv 堆料不如预期。  
2. **Common Crawl 上认真挖数学页**（OpenWebMath 等思路）→ 覆盖与 token 产量远好于「只吃 arXiv」。  
3. 数学指令 SFT（CoT / PoT / 工具整合）后再上 RL。

> 课堂金句级：基座域能力不够时，RL **造不出** 缺失知识；先 priming。

#### PPO 的内存痛点

典型要同时扛：

- 旧策略、新策略  
- Critic（value）  
- Reward model  

四套代价在大模型上很痛。

#### GRPO：Group Relative Policy Optimization

对同一问题采样一组答案 \(\{o_i\}\)，得分 \(\{r_i\}\)：

```text
A_i ≈ (r_i - mean(r_G)) / std(r_G)
```

- **用组内基线代替 learned critic** → 少一套大模型。  
- 与「奖励模型本就学比较」的直觉一致。  
- 课上强调：这是 **直觉式 advantage**；完整公式以论文为准。

#### 需要「奖励方差」

| 组内情况 | 后果 |
|----------|------|
| 全对或全错 | 标准化后无区分 / advantage≈0 → **浪费梯度** |
| 难度适中、有对有错 | 才有 hill-climb 信号 |

因此：**题太难全 0** 或 **太简单全 1** 都会让 GRPO 学不动。  
KL 惩罚：帮助学生不偏离参考策略、减轻「会做的简单题被冲掉」（课堂讨论点）。

#### 课上对 GRPO「提升了什么」的定性

- **majority@K（如 32）变好** → 输出更一致、更常对。  
- **pass@K 未必变好** → 不是突然会解一批「从前一次都解不出」的新题。  
- 解读：更像 **一致性 / 校准**，不是魔法般的能力跃迁。

### 3.3 DAPO — 大规模长 CoT RL 系统要点

**动机**：朴素 GRPO 上 Qwen-32B，AIME 可能只有 ~**30%**；DeepSeek 侧报告更高（课上 ~47%）。差距大量来自 **未写清的实现细节**：熵崩塌、过自信、长度失控、训练不稳。

DAPO 公开四招（课 + 论文对齐）：

#### （1）Clip-Higher / 非对称 clipping

- 标准 PPO clip 对「升概率 / 降概率」对称 → 低概率 token 难爬 → **探索崩塌**。  
- 非对称：允许低概率动作更大上调空间 → **熵更稳、准确率更高**（课上紫线 vs 崩塌曲线）。

#### （2）Dynamic Sampling

```text
对每题 oversample（例：目标组大小 64，可先采更多）
丢掉「全对」与「全错」组
只保留 0 < 正确数 < 组大小 的组再算梯度
```

目的：保证 **advantage 非零**、维持有效 batch，不把算力浪费在无信号组。  
64 是 exemplars，随难度与基座能力调整。

#### （3）Token-level policy loss

- Sample-level：一条又臭又长的垃圾答案与短好答案权重类似。  
- Token-level：按 token 塑形，抑制长度失控与熵乱飘。

#### （4）Overlong / 软超长惩罚

- 长思维易触发截断 → 截断链噪声大。  
- 对超长轨迹 **渐进惩罚**；也有工作选择 **RL 过程中加大 context**。  
- 过滤 overlong 本身就能抬分。

#### 课上累加消融（Qwen-32B · AIME，口播数字）

| 配置 | 约分 |
|------|------|
| 朴素 GRPO | **30** |
| + overlong filtering | **36** |
| + asymmetric clipping | **38** |
| + soft overlong punishment | **41** |
| + token-level loss | **42** |
| + dynamic sampling | **50** |

→ 可超过课上引用的 DeepSeek-R1-Zero-Qwen-32B（~47）一档；论文亦称用更少 step 达到 SoTA 量级。

### 3.4 三法如何选型（课末对照）

| 方法 | 何时用 | 你需要什么 |
|------|--------|------------|
| **STaR** | 只有小 rationale seed + 已知答案；几乎无 RL 基建 | 可验证答案；基座有非零可解率 |
| **GRPO / DeepSeekMath** | 基座已 math/code priming；要 memory-friendly online RL | 组采样；奖励方差；通常要 RM 或规则分 |
| **DAPO** | 长推理链、竞赛级难度、训练不稳 | 上述全部 + clip/采样/长度/熵监控 |

**SFT vs RL（课上回答）**：

- 有大量高质量示范 → **SFT 更快**。  
- 有强奖励、样本少 → **RL 更会 hill-climb**，但实现细节极敏感。  
- RL 不自动教「全新知识」；多在已有知识设计空间里更好探索。

---

## 4. 关键数字与实验结论

### 动机与缩放

- GPT-3.5 级 ~5% vs DeepSeekMath 7B ~51.7%（AIME 口播对比）。  
- o1：train-time 与 test-time compute **双侧** 提升 AIME pass@1（log-linear 叙事）。  

### STaR

- 相对直接 SFT：可用更少数据达到接近精度；口播一例约 **86% 数据 → ~72.5%** 量级设定（对照基线以论文为准）。  
- CommonsenseQA：人类偏好 STaR rationale 合理。  
- **GSM8K 上 rationalization 增益有限**：题对 GPT-J 过易 / CoT 步数已够，强制 rationale 帮助不大。  
- 多轮后易 **plateau**（不是完整 RL）。  

### DeepSeekMath / GRPO

- MATH：**46.8% → 51.7%**（Instruct → GRPO）；自洽 64 样本可至 **60.9%**（论文）。  
- GSM8K：**82.9% → 88.2%** 等（论文）。  
- 课上定性：提升 **majority@K**，未必提升 **pass@K**。  

### DAPO

- 朴素 GRPO ~30 → 全套 ~**50**（AIME，Qwen2.5-32B，口播消融链）。  
- 监控三角：**response length、entropy、全对样本占比**。  

### 负结果 / 边界（同等重要）

- 学 **失败样本** 仍不成熟；多数管线直接丢掉。  
- Reward 饱和 / 被 hack → 曲线假涨。  
- 厂商 RL 占比口述（非正式）：过去 ~1% → 或许 ~5%；有声称 50% RL 但未匹配涨幅 → **瓶颈常在奖励而非比例**。

---

## 5. 课堂 Q&A 与易错点

### Q1：Train / Test compute 怎么分配？有没有平坦 ROI？

**A**：留到后半；结论是 **互补、无单一最优比例**。Test 在强验证基准上可「几乎无限捡」；Train 要求闭环里已有足够成功。图不能当物理定律。

### Q2：专攻难题会不会毁掉简单题？

**A**：一般不应；除非过拟合到破碎/重复推理（overthinking）。GRPO 若难度分布全是极难 → 全 0 奖励无学习；KL / 保留参考策略有助。DAPO 的 dynamic sampling 也在造「有信号」的 batch。

### Q3：STaR 里 hint 出的 rationale 错了怎么办？

**A**：原版 **不在 step3 过滤**；后续可用 PRM / 人工。这是已知漏洞：可能学到「看似通向正确答案的错误推理」。

### Q4：为何不叫完整 RL？

**A**：主要用正确输出做过滤式 SFT；无完整 online advantage / 探索控制。加验证器闭环才更像 V-STaR 等。

### Q5：小模型该 STaR 还是蒸馏大模型？

**A**：实践常蒸馏更强；STaR 价值在 **自学回路教学** 与无教师时的自举。课上承认蒸馏路径合理。

### Q6：什么限制 STaR 上限？

课堂讨论汇总：

1. 基座推不出来的逻辑跳跃  
2. 给 hint 仍 rationalize 失败  
3. 某些答案易反推、某些不易  
4. 可解种子太少 → 泛化差  

### Q7：AIME 题少，信号不够怎么办？

**A**：RL 相对数据高效；关键是 **有足够可爬样本**，不是题库无限大。验证器用 ensemble（回扣 L03）。Dynamic sampling 还会滤掉无信号题组。

### Q8：RL 能让模型「学会从没见过的知识」吗？

**A**：课上倾向：**RL 是在已会的设计空间里更好搜索**；缺知识仍靠预训练/数据。

### 易错点

1. 把 majority@K 上涨当成 pass@K / 新能力。  
2. GRPO 时忽略难度混合 → 全 0/全 1。  
3. 长 CoT RL 不盯 entropy / length → 崩塌或爆炸。  
4. 假设加 RL 比例就能涨点 —— 忽略 reward 质量。  
5. 在不可验证域硬套同一套 outcome RL → hacking。

---

## 6. 本讲论文速查

| 论文 | 卡片 | 课堂角色 |
|------|------|----------|
| STaR | [star.md](../papers/star.md) | Rationale bootstrapping |
| DeepSeekMath (GRPO) | [deepseekmath-grpo.md](../papers/deepseekmath-grpo.md) | 组相对 RL、去 critic |
| DAPO | [dapo.md](../papers/dapo.md) | 大规模 RL 四件套 |

（本讲未精讲但相关：V-STaR、Quiet-STaR、RFT、PPO/DPO 统一视角——见 DeepSeekMath 论文讨论。）

---

## 7. 工程可落地 Checklist

- [ ] 先问：域内是否有 **可靠 verifier**（规则、执行、单测、形式化）？无则先投资验证而非 RL。  
- [ ] 画闭环：采样温度、样本数 K、过滤规则、更新算法、held-out。  
- [ ] STaR：准备 seed rationale；失败支路是否 rationalize；是否加 PRM 滤 hint 链。  
- [ ] 上 RL 前：code/math **数据 priming** 是否足够（DeepSeekMath 教训）。  
- [ ] GRPO：实现组采样与标准化；记录组内 reward std；全同组跳过。  
- [ ] 难度配比：混合易/中/难，保证非零且非饱和奖励。  
- [ ] 加 KL 或参考策略约束，监控简单题回归。  
- [ ] 长 CoT：启用非对称 clip、token-level loss、overlong 处理。  
- [ ] Dynamic sampling：oversample → 滤全对/全错 → 稳态有效 batch。  
- [ ] 仪表盘三件套：**entropy、均值回复长度、acc=1 组占比**。  
- [ ] 同时报 **majority@K 与 pass@K**，避免自欺。  
- [ ] Reward 曲线与 **独立 benchmark** 一起看，防 hacking。  
- [ ] 失败样本：当前默认丢弃；若要学负例，单独立项，勿假设「滤掉即最优」。  
- [ ] 训练结束后再扫一遍：格式、连贯性、overthinking 重复话术。

---

## 8. 若你只记 5 条

1. **Train-time scaling** = 验证过滤后的自生成数据再训练；可在小模型上换出大推理表现。  
2. **STaR**：保留做对的 rationale；做错的用答案 hint 倒推再训——自举推理数据，但受基座与 rationale 质量上限约束。  
3. **GRPO**：组内标准化奖励代替 critic，省内存；要有奖励方差；常抬 **一致性（majority）** 而非 pass@K。  
4. **DAPO**：非对称 clip + 动态采样 + token loss + 超长塑形，才能把长 CoT RL 拉到竞赛级分数。  
5. **取舍与风险**：test 搜、train 提 pass@1；二者都依赖验证器；弱/可攻击奖励 → 无学习或 reward hacking；RL 多在「已有知识」里搜索。

---

## 附录 A：口播结构对照

| 时间线（约） | 内容 |
|--------------|------|
| 0:00–16:00 | 动机、AIME、train vs test、推理行为模式 |
| 16:00–40:00 | STaR 算法、假设、实验、Q&A、变体 |
| 41:00–53:00 | DeepSeekMath 数据与 GRPO |
| 53:00–结束 | DAPO 四技巧、消融、开放问题、Q&A |

## 附录 B：监控与失败模式速查

```text
症状                     可能原因                 先试
---------------------------------------------------------
奖励涨、榜不涨           hacking / 过拟合         换独立评测、硬规则
entropy → 0            clip 过对称、过优化       Clip-Higher、调温
长度爆炸 / 截断噪声      sample-level loss        token loss、soft overlong
梯度像没学              组内全对或全错            dynamic sampling、调难度
简单题崩               无 KL / 分布过难          KL、加简单题
多数@K↑ pass@K 不动    「更稳」非「更会」         换数据或基座能力，别只加 RL
```

## 附录 C：与 L05 的衔接一句

L05 的 SPRINT/SWiRL 解决「多步轨迹怎么结构化、怎么逐步打分」；L06 解决「如何把这些信号 **稳定地** 写进参数」——同一自改进环的 **inference 结构** 与 **train 优化** 两半。
