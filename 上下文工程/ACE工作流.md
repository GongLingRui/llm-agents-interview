### ACE 核心工作流：从“二元”到“三角”

#### 之前主流 (Dynamic Cheatsheet)
- **角色**：
  - 1. Generator (生成器)
  - 2. Curator (策展人)
- **问题**：
  `Curator` 角色“过载”：它既要“反思”轨迹，又要“策展”记忆。
  → 导致“整体重写”和“上下文崩溃”。


#### ACE 框架 (Figure 4)
- **角色**：
  - 1. Generator (生成器)
  - 2. Reflector (反思器) (新！)
  - 3. Curator (策展人) (简化！)
- **问题**：
  `Reflector` 专门“反思”。
  `Curator` 只管“添加”。
  → 职责分离，解决了崩溃。


**补充说明**：ACE 的核心，是将 DC (v1.0) 的“过载策展人”，拆分为“专用反思器”+“简化策展人”。

<img width="731" height="374" alt="image" src="https://github.com/user-attachments/assets/1d7fa7a4-4735-45f6-aa4f-61bea5aceee4" />

### 三大角色的“人设” (Prompts) - 宏观总结

| 角色 (Fig) | Generator       | Reflector             | Curator               |
|------------|-----------------|-----------------------|-----------------------|
| 人设       | “超级智能AI助手” | “专家级编码教育家”     | “知识的策展大师”       |
| 关键输入   | 1. 任务(Query) <br> 2. `{{playbook}}` | 1. 轨迹 (Trajectory) <br> 2. 反馈 (Feedback) | 1. 洞见 (Insights) <br> 2. 当前手册 |
| 关键任务   | “开始做题”       | “诊断轨迹，找出根源。” <br> (输出: 结构化“错题报告”) | “只添加‘新’洞见。” <br> (输出: 结构化“增量包”) |

### 拆解 1：Reflector (反思器) 的 Prompt
- **人设**：“你是一个专家级的 AppWorld 编码代理和教育家...”
- **任务**：“诊断当前的轨迹... 找出哪里出了问题...”
- **关键输入**：轨迹、预测结果、真实答案 / 执行反馈 / 测试报告

- **强制输出格式 (Answer in this exact JSON format):**
```json
{
  "reasoning": "[Your chain of thought / reasoning...]",
  "error_identification": "[What specifically went wrong...]",
  "root_cause_analysis": "[Why did this error occur?...]",
  "correct_approach": "[What should the model have done instead?]",
  "key_insight": "[What strategy... should be remembered?]",
  "bullet_tags": [...]
}
```

### 拆解 2：Curator (策展人) 的 Prompt
- **人设**：“你是一个知识的策展大师...”
- **任务**：“只识别出那些‘缺失的’(Missing)、‘新的’(NEW)洞见...”
- **关键输入**：`Reflector`的“反思(Insights)”、`Current Playbook`(当前手册)

- **强制输出格式 (Output ONLY this JSON structure):**
```json
{
  "reasoning": "[Your chain of thought...]",
  "operations": [
    {
      "type": "ADD",
      "section": "[e.g., verification_checklist]",
      "content": "[New checklist item...]"
    }
    // ... 可能有多个 "ADD" 操作
  ]
}
```

### 创新 1：“迭代精炼” (Iterative Refinement)
- **机制（类比）**：
  Reflector 可以“自我反思”：
  - 第一轮：“代码报错了”（粗浅）。
  - 第二轮（精炼）：“…不，根源是‘用字符串匹配了时间戳’！”（高质量洞见）。
- **证据**：
  消融实验证明，“专用反思器”和“迭代精炼”各自都带来了显著的性能提升。
  → 证明了该机制的必要性。

**补充说明**：这是ACE能够提取“高质量、非简洁”洞见，对抗“简洁性偏见”的核心武器。

### 创新 2：“增量更新” (Incremental Delta Updates)
**目的**：彻底解决“上下文崩溃”

**机制：“Git Commit”模式 (第3.1节)**
- **1. 结构化 (Structure)**：“战术手册”被定义为“结构化的要点 (itemized bullets)”。
- **2. 元数据 (Metadata)**：每个“要点”都包含：`[ID]`、`[Helpful Counter]`、`[Harmful Counter]`、`[Content]`。
- **3. 操作 (Operation)**：`Curator` 只输出 `ADD` 指令 (如上页所示)。
- **4. 合并 (Merge)**：一个“非LLM的”轻量级逻辑，只负责“追加”(Append) 新条目，或更新旧条目的“计数器”。

**补充说明**：ACE 用两次“小调用”(R+C) + 一次“非LLM合并”，取代了 DC 的一次“超级大调用”。


### 创新 3：“生长优化机制” (Grow-and-Refine)
**目的**：解决“增量更新”的副作用：上下文无限膨胀

**机制：“生长”与“优化”的平衡 (第3.2节)**
- **1. 生长 (Grow)**：
  默认只“追加”(Append) 新策略，或更新旧策略的“元数据”(如`helpful`计数器)。
- **2. 优化 (Refine)**：
  周期性地启动“优化”(可以主动触发，也可以“懒惰”触发)。

**核心：如何“优化”？ → “去重” (De-duplication)**
通过“语义嵌入比较” (comparing via semantic embeddings)，修剪掉重复冗余的策略。
(类比：比如设置一个很高的相似度阈值，如0.95，相似则删除一个)

**补充说明**：这就在“持续积累细节”和“控制冗余”之间找到了完美的平衡。
