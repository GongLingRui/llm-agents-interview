# Stanford CS329A — Self-Improving AI Agents（课程知识库）

> 课程官网：[https://cs329a.stanford.edu/](https://cs329a.stanford.edu/)  
> 学期：Autumn 2025  
> 授课：Aakanksha Chowdhery、Azalia Mirhoseini  
> 目标：把整门课的录音要点 + 必读论文机制整理成**可自学、可检索**的笔记，无需再看视频。  
> **状态：核心讲次已整理完成**（`notes/` 14 篇 + `papers/` ~40 张卡）。有录音的讲次以录音为主；L07/L13/L14/客座为论文+公开材料补全。

## 目录结构

```text
self-improvementagent待整理仓库/
├── README.md                 # 本索引
├── notes/                    # 按讲次整理的结构化笔记（主阅读入口）
├── papers/                   # 论文精读卡（机制 / 实验 / 工程 takeaway）
└── raw-transcripts/          # 原始录音稿备份（勿当笔记读）
```

## 阅读顺序（建议）

| 顺序 | 笔记 | 主题 |
|------|------|------|
| 1 | [L01 课程总览](notes/L01-course-overview.md) | LLM 缩放、Agent 定义、课程地图 |
| 2 | [L02 Test-Time Compute](notes/L02-test-time-compute.md) | 重复采样、inference scaling laws、Archon |
| 3 | [L03 Robust Verification](notes/L03-robust-verification.md) | Outcome/Process reward、弱验证器、Gen-Ver gap |
| 4 | [L04 Feedback + Tools/Code](notes/L04-feedback-tools-code.md) | ReAct、Constitutional AI、RLEF |
| 5 | [L05 Multi-step Planning](notes/L05-multi-step-planning.md) | LATS、ADaPT、SWiRL、SPRINT、AB-MCTS |
| 6 | [L06 Train-time RL](notes/L06-train-time-rl.md) | STaR、GRPO/DeepSeekMath、DAPO |
| 7 | [L07 Open-Ended Evolution](notes/L07-open-ended-evolution.md) | AI Scientist、AlphaEvolve、ADAS / DGM（课表 arXiv 易混，见笔记） |
| 8 | [L08 Search & Deep Research](notes/L08-search-deep-research.md) | AlphaCode/2、Search-o1、Search-R1 |
| 9 | [L13 Software Engineering Agents](notes/L13-swe-agents.md) | CodeMonkeys、KernelBench、并行优化 |
| 10 | [L14 Memory](notes/L14-memory.md) | MemGPT、Cartridges、CacheBlend |
| 11 | [L17 Agentic Eval](notes/L17-agentic-eval.md) | 长任务、GDPVal、DeepScholar-Bench |
| 12 | [L20 Future Directions](notes/L20-future-directions.md) | 开放问题与研究前沿 |
| — | [客座讲座摘要](notes/guest-lectures.md) | DeepMind / Reflection / PI 等 |
| — | [全课程知识地图](notes/knowledge-map.md) | 概念串联、公式与决策表 |
| — | [论文总表](papers/README.md) | 全部必读论文索引 |

## 笔记统一模板

每讲 `notes/LXX-*.md` 至少包含：

1. **一句话总览**
2. **核心概念与定义**（含课堂强调的细节）
3. **方法 / 机制**（步骤、伪流程、与前后讲次关系）
4. **关键数字与实验结论**（来自课上图表/口播）
5. **课堂 Q&A 与易错点**
6. **本讲论文速查** → 链接到 `papers/`
7. **工程可落地 checklist**
8. **若你只记 5 条**

## 原始录音稿对照

| 原始文件 | 对应讲次 |
|----------|----------|
| `raw-transcripts/1.course-overview.md` | L01 |
| `raw-transcripts/2.Test-TimeComputeScaling.md` | L02 |
| `raw-transcripts/3.robustveriftication.md` | L03 |
| `raw-transcripts/4.learnfromfeedbacwithtoolscode.md` | L04 |
| `raw-transcripts/5.planmulti-stepreansoning.md` | L05 |
| `raw-transcripts/6.traintimescalingRL.md` | L06 |
| `raw-transcripts/7.self-improvmentdeepreasechagent.md` | **L08**（文件名易混，实为 Search & Deep Research） |
| `raw-transcripts/8.evalandlonghorizontask.md` | L17 |
| `raw-transcripts/9.未来研究方向.md` | L20 |

> 仓库内**没有** L07 / L13 / L14 及多数客座讲座的录音稿；这些讲次笔记以官网论文列表 + arXiv 精读补全，并标注「无录音 / 论文整理」。
