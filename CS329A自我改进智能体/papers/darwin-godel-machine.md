# 论文卡：Darwin Gödel Machine (DGM)

| 字段 | 内容 |
|------|------|
| **标题** | Darwin Gödel Machine: Open-Ended Evolution of Self-Improving Agents |
| **作者** | （见 arXiv 页面） |
| **出处** | arXiv:2505.22954 (2025) |
| **链接** | https://arxiv.org/abs/2505.22954 |
| **课程** | CS329A L07（用户材料 PDF 链到此文；标题有时被误写成 ADAS） |

## 问题

如何让 AI **改写自己的代码**并持续变强，同时用开放式档案避免局部最优？——经验版 Gödel Machine。

## 机制（怎么做）

1. 初始化带工具（Bash、edit）的 coding agent。  
2. 从 archive 按表现与「可继续自修改的子代数」选 parent。  
3. Parent 分析评测 log → 提议功能 → **修改自身 codebase** → child。  
4. Staged 评测（先 10 题功能门闩，再 50 题等）；只保留仍能 edit 的个体。  
5. 对比基线：无 self-improve（固定 meta≈ADAS 风格）/ 无 open-ended archive。

## 关键实验 / 数字

- SWE-bench Verified：**20.0% → 50.0%**。  
- Polyglot：**14.2% → 30.7%**。  
- 自改进与开放档案二者消融均伤持续增益；发现的功能可跨 LLM 迁移。

## 局限 / 安全

- 未真正重训 foundation model（算力与复杂度）。  
- 强调 sandbox、可追溯谱系；危险自修改必须隔离。  
- 评测噪声与子集估计带来方差。

## 工程 takeaway

- 「会改自己」+「保留历史 stepping stones」是递归自改进的两根支柱。  
- 落地先做 **可回滚的 agent 代码版本库 + 沙箱评测门闩**。

## 一句话

**经验主义 Gödel Machine：用评测代替证明，用 Darwin 档案代替单线爬山，让 coding agent 自我演化。**
