# 客座讲座摘要（CS329A Autumn 2025）

> **信息来源有限**：仓库**无**客座讲座完整录音稿。本页依据 [课程官网日程](https://cs329a.stanford.edu/)、讲者公开博客 / 论文 / 产品文档，以及 L20 录音中对客座的回指整理。  
> **请勿当作逐字课堂笔记**；标「公开材料」处可核查原文，「课程回指」处来自本仓库 L20 录音。

---

## 总览

| # | 日期 | 讲者 | 主题 |
|---|------|------|------|
| L09 | Mon Oct 20 | Melvin Johnson (Google DeepMind) | Evolution of Post-training from Chatbots to Agents |
| L14* | Fri Nov 7 | Junchen Jiang (LMCache, UChicago) | Memory serving（与 L14 论文同场） |
| L15 | Mon Nov 10 | Denny Zhou (Google DeepMind) | LLM Reasoning |
| L16 | Fri Nov 14 | Thang Luong (Google DeepMind) | AlphaProof / AlphaGeometry / Gemini IMO |
| L18 | Fri Nov 21 | Misha Laskin (Reflection AI) | Building Agentic Systems for Autonomy |
| L19 | Mon Dec 1 | Danny Driess (Physical Intelligence) | Multimodal AI Agents in Robotics |

\*Junchen 为 L14 Memory 课客座。

---

## 1. Melvin Johnson — Evolution of Post-training from Chatbots to Agents

**课程标注**：Google DeepMind；L09。

### 公开可核对的背景

- 行业叙事主线：Instruction tuning / RLHF（chatbot 对齐）→ 工具使用与多步 agent → RLVR / 可验证奖励上的推理后训练。  
- DeepMind / Google 线与 Gemini 后训练、agent 产品化相关（具体幻灯以课堂为准）。

### 与本课的接口（按大纲推断 + L20 回指）

- L20 明确：学期含 **post-training** 客座；自改进飞轮大量发生在 **post-training** 而非仅 pretrain。  
- 连接 L04 Constitutional AI / RLEF、L06 train-time RL、L09 作为「从对话模型到 agent」的产业视角。

### 自学抓手

- 对照思考：奖励信号从「人类偏好」迁到「环境 / 测试可验证」时，数据与基础设施怎样变？  
- 与 Melvin 公开学术主页 / DeepMind 后训练相关 blog 交叉阅读（课堂细节缺失）。

> **缺口**：无录音，无幻灯；细节深度有限。

---

## 2. Junchen Jiang — LMCache / Memory Serving（随 L14）

**课程标注**：LMCache, UChicago；与 Cartridges / MemGPT / CacheBlend 同课。

### 公开材料（相对充分）

| 工作 | 要点 |
|------|------|
| **CacheBlend** (EuroSys’25) | 多 chunk KV 选择性重算融合；TTFT 2.2–3.3×，吞吐 2.8–5× |
| **LMCache** (arXiv:2510.09665 等) | 企业级 KV 层：GPU 外分层存储、跨引擎/跨请求复用、PD 分离、非前缀 reuse（含 Blend） |
| **工程** | https://github.com/LMCache/LMCache ；文档含 blending 配置 |

### 课程回指

- L20：讨论 continual learning 时点名 **Cartridges**；记忆服务与「有效上下文」是同一谱系。  
- L14 笔记已展开三篇必读；客座角色偏 **serving 系统如何把记忆做快、做便宜**。

### 若你只记 3 条

1. Prefix cache 不够撑 RAG agent → 需要非前缀 KV 复用。  
2. KV 应变成可调度的一等资源（CPU/磁盘/远端）。  
3. 质量近似靠**少量重算**，不是盲目 full reuse。

> **缺口**：现场 demo 与口播配置未记录；以论文与官方文档为准。

---

## 3. Denny Zhou — LLM Reasoning

**课程标注**：Google DeepMind；L15「LLM Reasoning」。

### 公开可核对的背景

- Denny Zhou 以 **Chain-of-Thought、Least-to-Most、Self-Consistency、指令推理** 等推理提示与算法工作知名（Google Research / DeepMind）。  
- 主题与本课 L02–L06 的 test-time / 验证 / RL 推理直接同族。

### 与本课的接口

- 推理 = 可被 test-time compute 放大的中间过程；  
- 与 L03 process reward、L20「为何重复采样会冒出正确答案」开放问题相连。

### 自学抓手

- 重读 CoT / Self-Consistency / Least-to-Most 经典论文，对照课上 RLVR 前后差异：提示技巧 → 训练出的推理策略。

> **缺口**：无 2025 秋实际幻灯；勿臆造课上定理或实验数字。

---

## 4. Thang Luong — AlphaProof / AlphaGeometry / Gemini IMO

**课程标注**：Towards AI Superhuman Reasoning: AlphaProof, AlphaGeometry & Gemini IMO Gold Medal。

### 公开材料（DeepMind 博客，可核对）

**IMO 2024（银牌水平）**

- **AlphaProof**：RL + 形式化（Lean 等）；大规模 auto-formalize；难题可用 test-time RL。  
- **AlphaGeometry 2**：神经符号几何；Gemini 基座 + 更大规模合成数据。  
- 合计解出 6 题中 4 题，约 28/42，银牌水平。  
- Thang Luong 领导 AlphaGeometry 2 与自然语言推理相关努力（博客致谢）。

**IMO 2025（金牌水平，Gemini Deep Think）**

- 高级 Gemini Deep Think：**自然语言端到端**，官方题面 → 严格证明；竞赛时限内；5/6 题，35 分金牌线。  
- 相对 2024：减少「人工译成形式语言 + 多日计算」依赖。  
- Thang Luong 任 IMO 2025 技术方向领导（DeepMind 博客）。

### 课程回指

- L20 讲 DeepSeekMath-V2 时：**「Gemini 版本曾由客座讲过；这是开源对应」**——即本场。  
- 形式化验证 ↔ 自然语言推理双轨，与 L03 / L20 元验证叙事一致。

### 若你只记 3 条

1. 形式化环境 = 极强 verifier，使 RL self-play 可行。  
2. 几何神经符号 + 合成数据可扩到奥赛难度。  
3. 2025：纯自然语言金牌路径出现，但仍与形式化系统并行发展。

---

## 5. Misha Laskin — Building Agentic Systems for Autonomy

**课程标注**：Reflection AI；Lessons & Open questions。

### 公开可核对的背景

- Misha Laskin：RL / agent 研究背景；Reflection AI 与课上讲师 Aakanksha 有关联（L01 录音亦提及 Reflection）。  
- 主题词：**autonomy、agentic systems、lessons、open questions**——偏系统建设与研究判断，而非单篇算法。

### 与本课的接口（推断）

- 自主性 = 目标管理 + 工具 + 记忆 + 评测 + 人机边界（贯穿 L13–L17）。  
- Open questions 可能重叠 L20：可靠性长尾、环境设计、数据飞轮。

### 自学抓手

- 用 L17 三基准问：你的 agent「自主」时测的是时长、经济胜率还是综合质量？  
- 用 L13 问：接口与 selection 是否先于「更大模型」？

> **缺口**：无录音；以下不编造 Reflection 内部架构细节。

---

## 6. Danny Driess — Multimodal AI Agents in Robotics

**课程标注**：Physical Intelligence；Multimodal AI Agents in Robotics。

### 公开可核对的背景

- Danny Driess：机器人基础模型 / 多模态动作模型方向（如 RT-X、PaLM-E 等历史工作线；现 Physical Intelligence）。  
- 具身智能：视觉–语言–动作，数据收集常成主假设（L17 课末亦提 robotics startups 赌数据采集）。

### 课程回指（L20 Q&A，较硬）

- 讨论 continual learning「改记忆还是改权重」时，Akanksha 点名：  
  > 周一机器人客座里的 **cross-embodiment generalization**，只靠 memory systems **不会出现**，需要 **更新权重** 做技能迁移。  
- 故本场在课程结构中的功能：展示 **技能 / 策略记忆 ≠ 文本档案记忆**。

### 若你只记 3 条

1. 多模态机器人 agent 的瓶颈常在**数据与具身多样性**。  
2. 文本 agent 的 MemGPT / Cartridge 不能直接替代策略学习。  
3. 与 L17：长尾可靠性 + 慢 / 贵的真实环境反馈。

---

## 7. 客座 × 主课对照表

| 客座 | 主课锚点 |
|------|----------|
| Melvin post-training | L04–L06 对齐与 RL |
| Junchen memory serving | L14；L20 IPW / KV |
| Denny reasoning | L02–L03；CoT 谱系 |
| Thang IMO | L03 验证；L20 DeepSeekMath-V2 |
| Misha autonomy | L13–L17 系统与评测 |
| Danny robotics | L20 权重 vs 记忆；L17 具身数据 |

---

## 8. 维护说明

若日后补上客座录音或幻灯：

1. 在本文件对应节加入「课堂要点」子节；  
2. 把关键数字迁入 `papers/` 或专页；  
3. 去掉该节的「信息来源有限」警告。
