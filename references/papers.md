# Paper References

> 条目格式见 [reference-management.md](../standards/reference-management.md)。ID 前缀：`paper-`

### paper-react-2022
- Title: ReAct: Synergizing Reasoning and Acting in Language Models
- Type: paper
- Authors/Org: Shunyu Yao, Jeffrey Zhao, Dian Yu, Nan Du, Izhak Shafran, Karthik Narasimhan, Yuan Cao
- Year: 2022
- URL: https://arxiv.org/abs/2210.03629
- DOI/Repo: arXiv:2210.03629
- Topics: core/agent-loop, core/reasoning-planning, core/tool-use
- Summary: 提出推理轨迹与动作交错生成（ReAct），使 LLM 在与外部工具/环境交互中同时规划与行动；是现代 Agent Loop 的重要思想源头之一。
- Added: 2026-10-07
- Last checked: 2026-10-07

### paper-cot-2022
- Title: Chain-of-Thought Prompting Elicits Reasoning in Large Language Models
- Type: paper
- Authors/Org: Jason Wei, Xuezhi Wang, Dale Schuurmans, Maarten Bosma, Brian Ichter, Fei Xia, Ed Chi, Quoc Le, Denny Zhou
- Year: 2022
- URL: https://arxiv.org/abs/2201.11903
- DOI/Repo: arXiv:2201.11903
- Topics: core/reasoning-planning
- Summary: 通过 few-shot 示例诱导模型生成中间推理步骤（CoT），提升算术/常识/符号推理表现；是「显式推理痕迹」的重要来源，但本身不是 Agent Planning。
- Added: 2026-10-07
- Last checked: 2026-10-07

### paper-reflexion-2023
- Title: Reflexion: Language Agents with Verbal Reinforcement Learning
- Type: paper
- Authors/Org: Noah Shinn, Federico Cassano, Edward Berman, Ashwin Gopinath, Karthik Narasimhan, Shunyu Yao
- Year: 2023
- URL: https://arxiv.org/abs/2303.11366
- DOI/Repo: arXiv:2303.11366
- Topics: core/reasoning-planning, core/agent-loop, core/memory
- Summary: 用语言化反思与情节记忆缓冲改进后续 trial，而非更新权重；对应工程上的 reflection + episodic memory 钩子，不是训练式 RL。
- Added: 2026-10-07
- Last checked: 2026-10-07

### paper-tot-2023
- Title: Tree of Thoughts: Deliberate Problem Solving with Large Language Models
- Type: paper
- Authors/Org: Shunyu Yao, Dian Yu, Jeffrey Zhao, Izhak Shafran, Thomas L. Griffiths, Yuan Cao, Karthik Narasimhan
- Year: 2023
- URL: https://arxiv.org/abs/2305.10601
- DOI/Repo: arXiv:2305.10601
- Topics: core/reasoning-planning
- Summary: 将中间「thought」作为搜索节点，支持多路径探索、评估与回溯；对应工程上的 search/exploration 控制，成本显著高于单链推理。
- Added: 2026-10-07
- Last checked: 2026-10-07

### paper-plan-and-solve-2023
- Title: Plan-and-Solve Prompting: Improving Zero-Shot Chain-of-Thought Reasoning by Large Language Models
- Type: paper
- Authors/Org: Lei Wang, Wanyu Xu, Yihuai Lan, Zhiqiang Hu, Yunshi Lan, Roy Ka-Wei Lee, Ee-Peng Lim
- Year: 2023
- URL: https://arxiv.org/abs/2305.04091
- DOI/Repo: arXiv:2305.04091
- Topics: core/reasoning-planning
- Summary: 先制定计划再按计划求解（PS / PS+），针对 Zero-shot-CoT 的缺步等问题；更接近「显式计划痕迹」，但仍主要是 prompting 范式，不等于生产 Agent 的 plan 状态机。
- Added: 2026-10-07
- Last checked: 2026-10-07

### paper-self-refine-2023
- Title: Self-Refine: Iterative Refinement with Self-Feedback
- Type: paper
- Authors/Org: Aman Madaan et al.
- Year: 2023
- URL: https://arxiv.org/abs/2303.17651
- DOI/Repo: arXiv:2303.17651
- Topics: core/reasoning-planning
- Summary: 同一模型生成→自我反馈→迭代精炼；对应 reflection/refinement 控制流，需硬终止条件以防无限反思。
- Added: 2026-10-07
- Last checked: 2026-10-07

### paper-generative-agents-2023
- Title: Generative Agents: Interactive Simulacra of Human Behavior
- Type: paper
- Authors/Org: Joon Sung Park, Joseph C. O'Brien, Carrie J. Cai, Meredith Ringel Morris, Percy Liang, Michael S. Bernstein
- Year: 2023
- URL: https://arxiv.org/abs/2304.03442
- DOI/Repo: arXiv:2304.03442
- Topics: core/memory, core/reasoning-planning
- Summary: 以自然语言存储完整经历，经检索与反思合成高层记忆并驱动规划；展示 memory stream + retrieval + reflection 的工程可映射架构。
- Added: 2026-10-07
- Last checked: 2026-10-07

### paper-memgpt-2023
- Title: MemGPT: Towards LLMs as Operating Systems
- Type: paper
- Authors/Org: Charles Packer, Sarah Wooders, Kevin Lin, Vivian Fang, Shishir G. Patil, Ion Stoica, Joseph E. Gonzalez
- Year: 2023
- URL: https://arxiv.org/abs/2310.08560
- DOI/Repo: arXiv:2310.08560
- Topics: core/memory, core/context-engineering
- Summary: 借鉴 OS 分层存储做虚拟上下文管理：在有限窗口与外置存储间分页式调度，支撑长会话记忆与文档分析。
- Added: 2026-10-07
- Last checked: 2026-10-07

### paper-rag-2020
- Title: Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks
- Type: paper
- Authors/Org: Patrick Lewis et al.
- Year: 2020
- URL: https://arxiv.org/abs/2005.11401
- DOI/Repo: arXiv:2005.11401
- Topics: core/rag
- Summary: 提出 RAG：参数化生成模型 + 非参数化检索记忆（稠密索引）联合用于知识密集型任务；奠定「检索增强生成」术语与基本范式。
- Added: 2026-10-07
- Last checked: 2026-10-07

### paper-hyde-2022
- Title: Precise Zero-Shot Dense Retrieval without Relevance Labels
- Type: paper
- Authors/Org: Luyu Gao, Xueguang Ma, Jimmy Lin, Jamie Callan
- Year: 2022
- URL: https://arxiv.org/abs/2212.10496
- DOI/Repo: arXiv:2212.10496
- Topics: core/rag
- Summary: HyDE：先生成假设文档再嵌入检索，以文档邻域锚定真实语料；零样本稠密检索技术，非完整 RAG 系统。
- Added: 2026-10-07
- Last checked: 2026-10-07

### paper-lost-in-the-middle-2023
- Title: Lost in the Middle: How Language Models Use Long Contexts
- Type: paper
- Authors/Org: Nelson F. Liu, Kevin Lin, John Hewitt, Ashwin Paranjape, Michele Bevilacqua, Fabio Petroni, Percy Liang
- Year: 2023
- URL: https://arxiv.org/abs/2307.03172
- DOI/Repo: arXiv:2307.03172
- Topics: core/context-engineering
- Summary: 长上下文中相关信息位于开头/结尾时表现较好，位于中间时显著下降；更长输入也会损害表现——支撑「More Context ≠ Better」。
- Added: 2026-10-07
- Last checked: 2026-10-07

### paper-gptq-2022
- Title: GPTQ: Accurate Post-Training Quantization for Generative Pre-trained Transformers
- Type: paper
- Authors/Org: Elias Frantar, Saleh Ashkboos, Torsten Hoefler, Dan Alistarh
- Year: 2022
- URL: https://arxiv.org/abs/2210.17323
- DOI/Repo: arXiv:2210.17323
- Topics: runtime/model-optimization
- Summary: 面向生成式 Transformer 的训练后权重量化（一步一层近似），在较低比特下保持生成质量；是 PTQ 工程落地的重要方法族之一。
- Added: 2026-10-07
- Last checked: 2026-10-07

### paper-awq-2023
- Title: AWQ: Activation-aware Weight Quantization for LLM Compression and Acceleration
- Type: paper
- Authors/Org: Ji Lin, Jiaming Tang, Haotian Tang, Shang Yang, Wei-Ming Chen, Wei-Chen Wang, Guangxuan Xiao, Xingyu Dang, Chuang Gan, Song Han
- Year: 2023
- URL: https://arxiv.org/abs/2306.00978
- DOI/Repo: arXiv:2306.00978
- Topics: runtime/model-optimization
- Summary: 按激活显著性保护重要权重通道的权重量化，强调硬件友好的低比特推理；常用于 LLM 压缩与加速选型对比。
- Added: 2026-10-07
- Last checked: 2026-10-07

### paper-smoothquant-2022
- Title: SmoothQuant: Accurate and Efficient Post-Training Quantization for Large Language Models
- Type: paper
- Authors/Org: Guangxuan Xiao, Ji Lin, Mickael Seznec, Hao Wu, Julien Demouth, Song Han
- Year: 2022
- URL: https://arxiv.org/abs/2211.10438
- DOI/Repo: arXiv:2211.10438
- Topics: runtime/model-optimization
- Summary: 通过平滑迁移权重/激活的量化难度，使 W8A8 等权重+激活量化更可行；对应工程上对激活离群值的处理思路。
- Added: 2026-10-07
- Last checked: 2026-10-07

### paper-speculative-decoding-2023
- Title: Fast Inference from Transformers via Speculative Decoding
- Type: paper
- Authors/Org: Yaniv Leviathan, Matan Kalman, Yossi Matias
- Year: 2023
- URL: https://arxiv.org/abs/2211.17192
- DOI/Repo: arXiv:2211.17192
- Topics: runtime/model-optimization, runtime/llm-inference
- Summary: 用草稿模型提议、目标模型验证的投机解码，在保持目标分布的前提下加速自回归采样；接受率决定实际加速。
- Added: 2026-10-07
- Last checked: 2026-10-07

### paper-frugalgpt-2023
- Title: FrugalGPT: How to Use Large Language Models While Reducing Cost and Improving Performance
- Type: paper
- Authors/Org: Lingjiao Chen, Matei Zaharia, James Zou
- Year: 2023
- URL: https://arxiv.org/abs/2305.05176
- DOI/Repo: arXiv:2305.05176
- Topics: runtime/model-optimization
- Summary: 通过提示适配、LLM 级联与不同模型组合，在降低成本的同时尽量保持或提升任务表现；对应路由/级联策略的早期系统化讨论。
- Added: 2026-10-07
- Last checked: 2026-10-07

### paper-distilbert-2019
- Title: DistilBERT, a distilled version of BERT: smaller, faster, cheaper and lighter
- Type: paper
- Authors/Org: Victor Sanh, Lysandre Debut, Julien Chaumond, Thomas Wolf
- Year: 2019
- URL: https://arxiv.org/abs/1910.01108
- DOI/Repo: arXiv:1910.01108
- Topics: runtime/model-optimization
- Summary: 经典知识蒸馏实例：用教师 BERT 训练更小的 DistilBERT，在大幅减小体积与延迟的同时保留大部分能力；蒸馏动机的教学锚点（非现代 Agent 轨迹蒸馏本身）。
- Added: 2026-10-07
- Last checked: 2026-10-07

### paper-webarena-2024
- Title: WebArena: A Realistic Web Environment for Building Autonomous Agents
- Type: paper
- Authors/Org: Shuyan Zhou et al.
- Year: 2024
- URL: https://arxiv.org/abs/2307.13854
- DOI/Repo: arXiv:2307.13854
- Topics: agent-types/browser-computer-use, production/evaluation
- Summary: 可复现自托管真实站点环境与长程 Web 任务；强调功能正确性/执行态评测，而非仅匹配动作文本序列。
- Added: 2026-10-07
- Last checked: 2026-10-07

### paper-mind2web-2023
- Title: Mind2Web: Towards a Generalist Agent for the Web
- Type: paper
- Authors/Org: Xiang Deng, Yu Gu, Boyuan Zheng, Shijie Chen, Sam Stevens, Boshi Wang, Huan Sun, Yu Su
- Year: 2023
- URL: https://arxiv.org/abs/2306.06070
- DOI/Repo: arXiv:2306.06070
- Topics: agent-types/browser-computer-use, production/evaluation
- Summary: 真实网站跨域任务集；指出原始 HTML 过大，需先过滤候选元素再交给 LLM 做 grounding/动作选择。
- Added: 2026-10-07
- Last checked: 2026-10-07

### paper-visualwebarena-2024
- Title: VisualWebArena: Evaluating Multimodal Agents on Realistic Visually Grounded Web Tasks
- Type: paper
- Authors/Org: Jing Yu Koh et al.
- Year: 2024
- URL: https://arxiv.org/abs/2401.13649
- DOI/Repo: arXiv:2401.13649
- Topics: agent-types/browser-computer-use, production/evaluation
- Summary: 在可复现 Web 环境上强调视觉 grounding 任务；说明纯文本通道不足以覆盖大量真实 GUI 任务。
- Added: 2026-10-07
- Last checked: 2026-10-07

### paper-osworld-2024
- Title: OSWorld: Benchmarking Multimodal Agents for Open-Ended Tasks in Real Computer Environments
- Type: paper
- Authors/Org: Tianbao Xie et al.
- Year: 2024
- URL: https://arxiv.org/abs/2404.07972
- DOI/Repo: arXiv:2404.07972
- Topics: agent-types/browser-computer-use, production/evaluation, runtime/sandbox
- Summary: 真实计算机环境中的开放 Computer Use 任务基准；覆盖多应用桌面交互，支撑全环境 GUI Agent 评测。
- Added: 2026-10-07
- Last checked: 2026-10-07

### paper-orca-2022
- Title: Orca: A Distributed Serving System for Transformer-Based Generative Models
- Type: paper
- Authors/Org: Gyeong-In Yu, Joo Seong Jeong, Geon-Woo Kim, Soojeong Kim, Byung-Gon Chun
- Year: 2022
- URL: https://www.usenix.org/conference/osdi22/presentation/yu
- DOI/Repo: OSDI 2022
- Topics: runtime/llm-inference
- Summary: 提出 iteration-level（连续）批处理调度，使生成式 Transformer 服务在请求动态到达/完成时仍能高效批执行；是现代 LLM 连续批思想的重要来源。
- Added: 2026-10-07
- Last checked: 2026-10-07

### paper-pagedattention-2023
- Title: Efficient Memory Management for Large Language Model Serving with PagedAttention
- Type: paper
- Authors/Org: Woosuk Kwon, Zhuohan Li, Siyuan Zhuang, Ying Sheng, Lianmin Zheng, Cody Hao Yu, Joseph E. Gonzalez, Hao Zhang, Ion Stoica
- Year: 2023
- URL: https://arxiv.org/abs/2309.06180
- DOI/Repo: arXiv:2309.06180
- Topics: runtime/llm-inference
- Summary: 用分页管理 KV Cache（PagedAttention）降低显存碎片并提高并发；支撑高吞吐 LLM 服务，对应工程上「并发上限常由 KV 池决定」。
- Added: 2026-10-07
- Last checked: 2026-10-07

### paper-instructgpt-2022
- Title: Training language models to follow instructions with human feedback
- Type: paper
- Authors/Org: Long Ouyang et al. (OpenAI)
- Year: 2022
- URL: https://arxiv.org/abs/2203.02155
- DOI/Repo: arXiv:2203.02155
- Topics: runtime/reinforcement-learning, runtime/fine-tuning
- Summary: InstructGPT：SFT → 人类偏好奖励模型 → PPO 策略优化的 RLHF 流水线；奠定现代指令对齐工程范式，强调代理奖励与 KL 约束。
- Added: 2026-10-07
- Last checked: 2026-10-07

### paper-constitutional-ai-2022
- Title: Constitutional AI: Harmlessness from AI Feedback
- Type: paper
- Authors/Org: Yuntao Bai et al. (Anthropic)
- Year: 2022
- URL: https://arxiv.org/abs/2212.08073
- DOI/Repo: arXiv:2212.08073
- Topics: runtime/reinforcement-learning
- Summary: 用宪章原则驱动 AI 反馈做 harmless 对齐（RLAIF 思想源头之一）；减少对人类标注有害偏好的依赖，同时引入评判协议版本化与偏差风险。
- Added: 2026-10-07
- Last checked: 2026-10-07

### paper-rlaif-2023
- Title: RLAIF: Scaling Reinforcement Learning from Human Feedback with AI Feedback
- Type: paper
- Authors/Org: Harrison Lee et al. (Google Research)
- Year: 2023
- URL: https://arxiv.org/abs/2309.00267
- DOI/Repo: arXiv:2309.00267
- Topics: runtime/reinforcement-learning
- Summary: 系统研究用 AI 偏好替代/扩展人类偏好做 RL 对齐；表明在合适设定下可逼近 RLHF，但工程上必须以人类金标校准并隔离训练奖励与发布门禁。
- Added: 2026-10-07
- Last checked: 2026-10-07

### paper-dpo-2023
- Title: Direct Preference Optimization: Your Language Model is Secretly a Reward Model
- Type: paper
- Authors/Org: Rafael Rafailov, Archit Sharma, Eric Mitchell, Stefano Ermon, Christopher D. Manning, Chelsea Finn
- Year: 2023
- URL: https://arxiv.org/abs/2305.18290
- DOI/Repo: arXiv:2305.18290
- Topics: runtime/reinforcement-learning
- Summary: DPO 将偏好学习直接写成对策略的分类目标，避免显式奖励模型与在线 PPO；降低对齐工程负担，但强烈依赖离线偏好对目标行为分布的覆盖。
- Added: 2026-10-07
- Last checked: 2026-10-07
