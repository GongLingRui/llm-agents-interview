# Skills 编写完全指南 —— 小白也能看懂的版本

> 基于一泽Eze的《Agent Skills 终极指南》整理
>
> 全文约 1.2 万字，应该是全网最好的 Skills 中文指南与教程

---

## 目录

- [一、Skills 是什么？](#一skills-是什么)
  - [1.1 用人话解释 Skills](#11-用人话解释-skills)
  - [1.2 Skills 和 MCP 有什么区别？](#12-skills-和-mcp-有什么区别)
  - [1.3 理解 Skill 的结构](#13-理解-skill-的结构)
- [二、Skills 的真实价值](#二skills-的真实价值)
  - [2.1 零代码、自然语言编写](#21-零代码自然语言编写)
  - [2.2 突破预设限制，灵活应对](#22-突破预设限制灵活应对)
  - [2.3 多 Skills 自由联用](#23-多-skills-自由联用)
- [三、Skills 是如何运行的？](#三skills-是如何运行的)
  - [3.1 渐进式披露机制](#31-渐进式披露机制)
  - [3.2 三层披露结构](#32-三层披露结构)
- [四、如何使用 Skills](#四如何使用-skills)
  - [4.1 安装 Claude Code](#41-安装-claude-code)
  - [4.2 替换模型（可选）](#42-替换模型可选)
  - [4.3 安装并使用 Skills](#43-安装并使用-skills)
  - [4.4 如何找到好用的 Skills](#44-如何找到好用的-skills)
- [五、如何制作一个 Skill](#五如何制作一个-skill)
  - [5.1 使用 skill-creator 自动创建](#51-使用-skill-creator-自动创建)
  - [5.2 Skill 文件结构详解](#52-skill-文件结构详解)
  - [5.3 编写 SKILL.md 的技巧](#53-编写-skillmd-的技巧)
  - [5.4 安装自制的 Skill](#54-安装自制的-skill)
- [六、什么时候应该用 Skills](#六什么时候应该用-skills)
  - [6.1 反复解释同一件事](#61-反复解释同一件事)
  - [6.2 需要特定知识、模板、材料](#62-需要特定知识模板材料)
  - [6.3 需要多流程协同](#63-需要多流程协同)
- [七、常见问题 FAQ](#七常见问题-faq)

---

## 一、Skills 是什么？

### 1.1 用人话解释 Skills

想象你要把一项工作交给新同事，而且你不能口口相传，只能用文档交接（你想一次性交接完成，以后不被打扰）。

你会准备什么？

- ✅ 任务的执行 SOP 与必要背景知识（这件事大致怎么做）
- ✅ 工具的使用说明（用什么软件、怎么操作）
- ✅ 要用到的模板、素材（历史案例、格式规范）
- ✅ 可能遇到的问题、规范、解决方案（细节指引补充）

**Skill 就是这个"工作交接大礼包"的数字版本！**

```
┌─────────────────────────────────────────────────┐
│                  Skill 文件夹                    │
├─────────────────────────────────────────────────┤
│  📄 SKILL.md          ← 核心指导文档             │
│  📁 scripts/          ← 工具脚本                 │
│  📁 reference/        ← 参考文档                 │
│  📁 assets/           ← 素材资源                 │
└─────────────────────────────────────────────────┘
```

**简单来说**：Skills 是给 AI Agent 准备的"扩展包"，Agent 加载不同的 Skills 包，就能具备不同的专业能力，稳定完成特定任务。

**举个例子**：

| Skill 名称 | 功能说明 |
|-----------|----------|
| **PDF Skill** | 包含 PDF 合并、拆分、文本提取等脚本，教会 AI 如何处理 PDF 文件 |
| **Brand Guidelines Skill** | 包含品牌设计规范、Logo 资源等，AI 设计时会自动遵循企业规范 |
| **Skill Creator** | 把创建 Skill 的方法打包成元 Skill，让 AI 帮你创建新的 Skill |

---

### 1.2 Skills 和 MCP 有什么区别？

这是最常见的疑惑，用一个表格来对比：

| 对比项 | MCP（Model Context Protocol） | Skill |
|--------|------------------------------|-------|
| **本质** | 开放标准的**协议** | 模块化的**能力包** |
| **关注点** | AI 如何调用外部工具、数据和服务 | AI 如何完整处理特定工作 |
| **是否定义逻辑** | ❌ 不定义任务逻辑或执行流程 | ✅ 封装执行方法、工具调用和知识材料 |
| **包含内容** | 统一的接口规范 | 指令文档 + 代码脚本 + 参考资源 |

**一句话总结**：
- **MCP** = 统一的"插座标准"
- **Skill** = 插上插座就能用的"电器"

---

### 1.3 理解 Skill 的结构

一个标准的 Skill 文件夹长这样：

```
my-skill/
├── SKILL.md              ⭐ 必需！核心指导文档
├── scripts/              可选：可执行脚本
│   ├── script1.py
│   └── script2.sh
├── reference/            可选：参考文档
│   └── guide.md
└── assets/               可选：素材资源
    ├── template.png
    └── logo.svg
```

#### SKILL.md 的结构

```markdown
---
name: pdf
description: 全面的 PDF 操作工具包，用于提取文本和表格、创建新 PDF、合并/拆分文档以及处理表单。当 Claude 需要填写 PDF 表单或大规模地程序化处理、生成或分析 PDF 文档时使用。
---

# PDF Skill

## 什么时候使用
- 需要处理 PDF 文件时
- 需要提取 PDF 内容时
- 需要合并或拆分 PDF 时

## 如何使用
1. 首先读取用户上传的 PDF 文件
2. 使用 scripts/ 目录下的脚本进行处理
3. 返回处理结果

## 注意事项
- 大文件处理可能需要较长时间
- 某些加密 PDF 可能无法处理
```

**SKILL.md 分为两部分**：
- **YAML 元数据**（`---` 包围的部分）：名称和描述，约 100 tokens
- **MD 正文**：主要技能指令，建议少于 5000 tokens

---

## 二、Skills 的真实价值

### 2.1 零代码、自然语言编写

**对比传统的 AI 应用开发方式**：

| 开发方式 | 技术门槛 | 示例 |
|---------|---------|------|
| 程序编写 | ⭐⭐⭐⭐⭐ 必须懂编程 | Python、Java 开发 AI 应用 |
| Workflow 平台 | ⭐⭐⭐ 需理解节点配置 | Coze、Dify、N8N |
| **Skills** | **⭐ 只会写文档** | **自然语言编写 SKILL.md** |

#### 简单案例：Brand Guidelines Skill

整个 Skill 只有一个 `SKILL.md`，纯自然语言写成：

```markdown
---
name: brand-guidelines
description: Anthropic 官方品牌设计规范，包含颜色、字体、Logo 使用指南
---

# Anthropic Brand Guidelines

## 品牌颜色
- 主色：#D97757（陶土色）
- 辅助色：#1A1A1A（深灰色）

## 字体
- 标题：SF Pro Display
- 正文：SF Pro Text

## Logo 使用
- 确保 Logo 周围有足够的留白空间
- 不要拉伸或变形 Logo
```

就这么简单！AI 就能自动遵循这个设计规范来创建网站、海报、PPT。

#### 复杂案例：AI Partner Skill

一个完整的 Skill 可以包含：

```
ai-partner-skill/
├── SKILL.md              核心指导（自然语言）
├── scripts/              向量数据库脚本
│   ├── build_db.py
│   └── search_db.py
├── reference/            使用指南
│   └── vector_db_guide.md
└── assets/               模板资源
    ├── persona_template.json
    └── user_profile_template.json
```

**功能**：让 AI 深度学习你的记忆，塑造懂你的 AI 伴侣

- 引导用户上传包含个人记忆的文档
- 智能切分笔记片段
- 构建向量数据库
- 提供个性化的对话体验

**结论**：单靠 Skill + Agent，就能实现甚至超过同类 AI 产品的效果，而且**不需要编写任何代码**！

---

### 2.2 突破预设限制，灵活应对

#### Workflow/传统程序的问题

它们假设所有情况都能预设：

```
用户上传文件 → 检查格式 → 是否符合预期？
                              ↓
                    是 → 处理入库
                    否 → 报错，要求用户修改
```

**现实情况往往是**：
- 用户只有预期之外的格式（预期支持 md，实际只有 doc）
- 数据字段不符（预期每个文件需要标题，但用户文件没有标题）
- 出现了预设之外的边缘情况

#### Skill + Agent 的优势

```
用户输入 → Agent 理解意图 → 智能调用 Skill
                              ↓
                    格式不对？自动转换脚本
                    字段缺失？AI 自行补充
                    边缘情况？LLM 推理解决
```

**实际案例**：AI-Partner-Chat 的自适应切片

- **固定方式**：所有文件都按照固定分隔符或字数切分
- **Skill 方式**：Agent 根据文件类型智能选择切片方式
  - DailyNotes → 按日期标题切分
  - 项目笔记 → 按标题级别与语义切分

---

### 2.3 多 Skills 自由联用

Skills 可以在一次任务中调用多个 Skill，就像搭积木一样灵活。

**举例：制作一份产品分析报告**

```
Step 1: Web Scraping Skill    → 抓取竞品数据
          ↓
Step 2: PDF Skill             → 提取用户反馈
          ↓
Step 3: Data Analysis Skill   → 分析数据并生成图表
          ↓
Step 4: Brand Guidelines Skill → 按品牌规范制作 PPT
          ↓
Step 5: PPTX Skill            → 生成最终 PPT
```

每多一个 Skill，就多一种能力。N 个 Skill 可以应对远超 N 的应用场景！

---

## 三、Skills 是如何运行的？

### 3.1 渐进式披露机制

这是 Skills 运行的核心机制。

**问题**：上下文过长会导致模型能力下降

**解决方案**：Skill 不会一次性全部加载，而是根据需要"渐进式披露"

```
┌────────────────────────────────────────────────────┐
│               Agent 架构                            │
├────────────────────────────────────────────────────┤
│                                                     │
│   ┌─────────────────────────────────────┐         │
│   │      Context Window（上下文窗口）     │         │
│   │   ┌─────────────────────────────┐   │         │
│   │   │  Level 1: 元数据（始终加载）  │   │         │
│   │   └─────────────────────────────┘   │         │
│   │   ┌─────────────────────────────┐   │         │
│   │   │  Level 2: 指令（触发时加载）  │   │         │
│   │   └─────────────────────────────┘   │         │
│   │   ┌─────────────────────────────┐   │         │
│   │   │  Level 3: 资源（按需加载）   │   │         │
│   │   └─────────────────────────────┘   │         │
│   └─────────────────────────────────────┘         │
│                                                     │
│   ┌─────────────────────────────────────┐         │
│   │      Agent 文件系统                  │         │
│   │   （所有 Skill 文件放在这里）        │         │
│   └─────────────────────────────────────┘         │
│                                                     │
└────────────────────────────────────────────────────┘
```

### 3.2 三层披露结构

| 层级 | 内容 | 大小限制 | 加载时机 |
|-----|------|---------|---------|
| **Level 1** | SKILL.md 的元数据（name + description） | 约 100 tokens | Agent 启动时始终加载 |
| **Level 2** | SKILL.md 的完整正文 | 建议少于 5000 tokens | 用户消息匹配时加载 |
| **Level 3** | 子技能、脚本、参考文档、资源 | 无限制 | 按需动态读取 |

#### 加载流程

```
用户发送消息
      ↓
AI 读取 Level 1 元数据（始终在内存中）
      ↓
判断是否需要该 Skill？
      ↓ 是
加载 Level 2 指令文档
      ↓
执行任务，需要时加载 Level 3 具体资源
```

**好处**：
- 可以给一个 Agent 同时安装很多 Skills，但不影响上下文性能
- Level 3 因为按需加载，文件在被访问前不占用 Context，所以没有大小限制

---

## 四、如何使用 Skills

### 4.1 安装 Claude Code

> Claude Code 是 Anthropic 推出的通用 Agent 框架，能代替我们操作电脑

#### Step 1：打开终端

- **Mac**: 打开"终端"应用
- **Windows**: 打开"命令提示符"或"PowerShell"
- **Linux**: 打开任意终端模拟器

#### Step 2：运行安装命令

**Mac/Linux**：
```bash
curl -fsSL https://code.claude.com/install.sh | sh
```

**Windows**：
```powershell
powershell -c "irm https://code.claude.com/install.ps1 | iex"
```

#### Step 3：验证安装

```bash
claude --version
```

看到版本号就说明安装成功了！

```
claude-code version 1.0.0
```

> 💡 **小白提示**：如果遇到问题，把终端的报错信息发给任意 AI（ChatGPT、Kimi 都行），让它们帮你解决。

---

### 4.2 替换模型（可选）

现在大部分国产模型都支持 Skills，不一定非要用 Claude。

#### 方法一：手动配置

搜索"模型名称 + Claude Code 接入教程"，比如：
- "GLM 4.7 Claude Code"
- "Kimi K2-thinking Claude Code"

#### 方法二：使用 CC Switch 工具

项目地址：https://github.com/farion1231/cc-switch

这个工具可以方便地管理和切换不同的模型。

---

### 4.3 安装并使用 Skills

#### Step 1：准备工作目录

```bash
# 创建一个测试文件夹
mkdir test-skills

# 进入该文件夹
cd test-skills

# 启动 Claude Code
claude
```

看到以下界面说明启动成功：

```
╭──────────────────────────────────────────────────╮
│              Welcome to Claude Code               │
╰──────────────────────────────────────────────────╯

How can I help you today?
```

#### Step 2：获取 Skill 文件包

**官方 Skills 仓库**：https://github.com/anthropics/skills/tree/main

里面有各种现成的 Skills：
- PDF：PDF 处理工具
- PPTX：PowerPoint 创建和编辑
- Brand-guidelines：品牌设计规范
- Skill-creator：创建新 Skill 的工具

#### Step 3：安装 Skill

**方法 A：让 CC 自动安装**

在 Claude Code 中输入：

```
安装 skill，skill 项目地址为：https://github.com/anthropics/skills/tree/main/skills/pdf
```

**方法 B：手动安装**

1. 下载 Skill 文件包（zip 或文件夹）
2. 解压后放到 Skills 安装目录：

**项目专用目录**：
```
test-skills/.claude/skills/pdf-skill/
```

**全局目录**（所有项目共享）：
```
~/.claude/skills/pdf-skill/
```

#### Step 4：重启 Claude Code

```bash
# 按 Ctrl+C 退出，然后重新输入
claude
```

#### Step 5：使用 Skill

**显式调用**：
```
开始使用 pdf skill
```

**隐式调用**（自动匹配）：
```
帮我提取这个 PDF 文件的文本内容
```

AI 会自动识别需要使用 PDF Skill。

---

### 4.4 如何找到好用的 Skills

#### 官方仓库

- **Anthropic 官方**：https://github.com/anthropics/skills
- 包含官方维护的 Skills，质量有保证

#### 第三方市场

- **Skills MP**：https://skillsmp.com/zh
- **Mulerun**（即将支持 Skills）：
  - 全球性的 Agent 开发与交易市场
  - 一键运行测试 GitHub 上的 Skill
  - 自动评分和精选机制

---

## 五、如何制作一个 Skill

### 5.1 使用 skill-creator 自动创建

这是最简单的方法！

#### Step 1：安装 skill-creator

```
安装 skill，skill 项目地址为：https://github.com/anthropics/skills/tree/main/skills/skill-creator
```

#### Step 2：重启 Claude Code

```bash
# 按 Ctrl+C 退出，然后重新输入
claude
```

#### Step 3：创建 Skill

直接告诉 AI 你想要什么 Skill：

```
创建一个 skill，能将 PDF 文件转换为 Word 文档
```

AI 会自动：
1. 编写 SKILL.md
2. 创建必要的转换脚本
3. 生成完整的 Skill 文件包

#### 其他例子

```
创建 skill，能按照我写文章的行文风格写文章

创建 skill，能自动整理近期 AI 领域的新闻日报

创建 skill，能分析 Excel 数据并生成可视化图表
```

---

### 5.2 Skill 文件结构详解

#### 完整的 Skill 结构

```
my-skill/
├── SKILL.md              ⭐ 核心文档（必需）
├── scripts/              📜 可执行脚本（可选）
│   ├── process.py
│   └── utility.sh
├── reference/            📚 参考文档（可选）
│   ├── api-guide.md
│   └── best-practices.md
└── assets/               🎨 素材资源（可选）
    ├── templates/
    └── images/
```

#### SKILL.md 模板

```markdown
---
name: my-skill
description: 简短描述这个 Skill 的用途，以及什么时候应该使用它
---

# Skill 名称

## 概述
简要说明这个 Skill 的功能和目的。

## 什么时候使用
- 场景 1
- 场景 2
- 场景 3

## 使用步骤

### 步骤 1：准备工作
描述第一步需要做什么。

### 步骤 2：执行任务
描述如何执行核心任务。

### 步骤 3：输出结果
描述如何输出最终结果。

## 可用资源
- `scripts/process.py`：用于处理数据
- `reference/guide.md`：参考指南
- `assets/templates/`：模板文件

## 注意事项
- 注意事项 1
- 注意事项 2

## 示例

### 输入
```
示例输入
```

### 输出
```
示例输出
```
```

---

### 5.3 编写 SKILL.md 的技巧

#### 1. 元数据要清晰

```yaml
---
name: pdf-to-word
description: 将 PDF 文档转换为可编辑的 Word 文档，保留原始格式、图片和表格。当用户需要转换 PDF 文件时使用。
---
```

**要点**：
- `name` 简短易记
- `description` 包含功能 + 使用场景

#### 2. 使用清晰的结构

```markdown
## 使用步骤

### 第一步：读取 PDF
使用 `scripts/read_pdf.py` 读取 PDF 文件内容。

### 第二步：转换格式
使用 `scripts/convert.py` 将 PDF 转换为 Word 格式。

### 第三步：保存文件
将转换后的文件保存到原文件同目录。
```

#### 3. 提供具体示例

```markdown
## 示例

**用户输入**：
```
把 report.pdf 转换成 Word 文档
```

**执行过程**：
1. 读取 report.pdf
2. 使用 convert.py 转换
3. 生成 report.docx

**输出**：
```
✅ 转换完成！文件已保存为 report.docx
```
```

#### 4. 说明可用资源

```markdown
## 可用脚本

| 脚本 | 功能 | 使用方法 |
|-----|------|---------|
| `read_pdf.py` | 读取 PDF 内容 | `python scripts/read_pdf.py <文件路径>` |
| `convert.py` | PDF 转 Word | `python scripts/convert.py <输入文件> <输出文件>` |
```

---

### 5.4 安装自制的 Skill

#### 如果是 .skill 格式

```
直接安装 skill，文件路径为：/path/to/my-skill.skill
```

#### 如果是文件夹

将文件夹复制到 Skills 目录：

```bash
# 项目专用
cp -r my-skill .claude/skills/

# 全局
cp -r my-skill ~/.claude/skills/
```

#### 如果是 zip 文件

```bash
# 解压
unzip my-skill.zip -d .claude/skills/

# 或者在 Windows 上手动解压到对应目录
```

---

## 六、什么时候应该用 Skills？

### 6.1 反复解释同一件事

**典型信号**：为了完成某个任务，需要不断向 AI 解释同一件事。

#### 例子 1：技术文档写作

```
你：帮我写一份技术文档
AI：好的...
你：不对，我们公司的格式是这样的...
AI：好的，重新写...
你：还有，代码示例要按这个模板来...
AI：好的，再改...
你：章节标题要用三级标题...
AI：明白了...
```

**解决方案**：创建一个 `tech-doc-skill`

```markdown
---
name: tech-doc
description: 按照公司技术文档规范编写文档
---

# 技术文档规范

## 文档结构
1. 一级标题：文档标题
2. 二级标题：章节
3. 三级标题：小节

## 代码示例格式
\```语言类型
代码内容
\```

## 注意事项
- 使用中文编写
- 每个章节要有示例
```

#### 例子 2：数据分析

```
你：帮我分析这个数据
AI：好的...
你：先把 > 100 的异常值筛掉
AI：好的...
你：不对，应该用中位数，不是平均值
AI：明白了...
你：图表要按我们公司的配色方案...
AI：好的...
```

**解决方案**：创建一个 `data-analysis-skill`

```markdown
---
name: data-analysis
description: 按照公司数据分析规范处理数据
---

# 数据分析规范

## 数据预处理
1. 剔除 > 100 的异常值
2. 使用中位数而非平均值
3. 保留 2 位小数

## 图表配色
- 主色：#1890FF
- 辅助色：#52C41A
- 警告色：#FAAD14
```

---

### 6.2 需要特定知识、模板、材料

**典型场景**：AI 的通用能力够了，但缺"特定场景的知识材料"。

#### 场景对照表

| 场景 | 需要的材料 | Skill 组织方式 |
|-----|-----------|---------------|
| 技术文档写作 | 代码规范、术语表、文档模板 | 放入 `reference/` 和 `assets/` |
| 品牌设计 | 品牌手册、色彩规范、Logo | 放入 `assets/`，规范写入 SKILL.md |
| 数据分析 | 指标定义、计算公式、报表模板 | 放入 `reference/`，公式写入 SKILL.md |
| 法律文书 | 法律条款库、文书模板 | 放入 `assets/templates/` |

#### 示例：法律文书 Skill

```
legal-doc-skill/
├── SKILL.md                    # 法律文书写作指导
├── reference/                  # 法律参考资料
│   ├── contract-law.md        # 合同法要点
│   └── glossary.md            # 法律术语表
└── assets/                     # 文书模板
    ├── employment-contract.docx
    ├── nda-template.docx
    └── service-agreement.docx
```

---

### 6.3 需要多流程协同

**复杂任务**：需要"组合多个流程"才能完成。

#### 例子：竞品分析报告

```
竞品分析报告
    ↓
┌─────────────────────────────────────────┐
│ 1. 检索竞品数据（Web Scraping Skill）   │
│ 2. 提取用户反馈（PDF Skill）            │
│ 3. 分析数据生成图表（Data Analysis）    │
│ 4. 按品牌规范制作 PPT（Brand + PPTX）   │
└─────────────────────────────────────────┘
```

可以把每个环节的指令、脚本、材料打包成 Skill，让 Agent 根据任务智能调用。

---

## 七、常见问题 FAQ

### Q1：Skill 必须会编程才能写吗？

**A：不需要！** 简单的 Skill 只需要一个 `SKILL.md` 文件，用自然语言写成即可。

复杂 Skill 如果需要脚本，可以让 AI 帮你写，或者使用 skill-creator 自动生成。

### Q2：Skill 和 GPTs 有什么区别？

| 对比项 | GPTs | Skills |
|-------|------|--------|
| 平台绑定 | 只能在 OpenAI 使用 | 多平台支持（Claude、Cursor、VS Code 等） |
| 开发方式 | 图形界面配置 | 文件 + 自然语言 |
| 扩展性 | 有限 | 可以包含代码、资源、文档 |
| 可移植性 | 不易移植 | 标准文件格式，易于分享 |

### Q3：一个 Skill 文件夹多大合适？

**A：没有硬性限制**，但建议：
- `SKILL.md` 正文 < 5000 tokens
- 子技能文档按需拆分
- 脚本和资源文件因为按需加载，大小不影响性能

### Q4：如何测试我的 Skill？

**A：在 Claude Code 中**：

```
开始使用 my-skill

（然后输入测试任务）
```

或者让 AI 帮你测试：

```
请测试 my-skill 的功能，给它一个典型任务，看看执行效果如何
```

### Q5：Skill 可以商业化吗？

**A：可以！**

- Mulerun 等平台即将支持 Skills 交易
- 你可以将 Skill 打包成 Agent 产品出售
- 也可以提供 Skill 定制服务

### Q6：如何分享我的 Skill？

**A：几种方式**：

1. **GitHub**：创建仓库，分享 Skill 文件
2. **Skill 市场**：上传到 skillsmp.com 或 Mulerun
3. **直接分享**：打包成 zip 或 .skill 文件发送

### Q7：Skill 会替代传统开发吗？

**A：不会完全替代**，但会改变很多场景：

| 场景 | 传统开发 | Skills |
|-----|---------|--------|
| 大型应用 | ✅ 更合适 | - |
| 快速原型/MVP | ❌ 周期长 | ✅ 几小时搞定 |
| 内部小工具 | ❌ 成本高 | ✅ 打包 Skill 即可 |
| 需要极致性能 | ✅ 可优化 | ⚠️ 取决于设计 |

### Q8：Skill 的性能怎么样？

**A：取决于设计**：

- **Prompt 型 Skill**：依赖模型推理，较慢但灵活
- **脚本型 Skill**：直接运行代码，接近原生性能

> Skills 慢起来是 prompt，快起来也可以是 workflow

---

## 总结

### Skills 的核心价值

1. **零代码门槛**：只会写文档就能做 Agent 应用
2. **智能上限高**：借助通用 Agent 的智能能力
3. **灵活应变**：能应对预设之外的边缘情况
4. **可组合性强**：多个 Skill 可以自由联用

### 适合制作 Skill 的时机

- ✅ 发现自己在向 AI 反复解释同一件事
- ✅ 某些任务需要特定知识、模板、材料
- ✅ 发现一个任务要多个流程协同完成

### 快速上手路径

```
1. 安装 Claude Code
        ↓
2. 安装 skill-creator
        ↓
3. 让 AI 帮你创建第一个 Skill
        ↓
4. 测试并优化
        ↓
5. 分享或商业化
```

---

## 参考资源

### 官方文档

- **Claude Skills 官方说明**：https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview
- **Agent Skills 开放标准**：https://agentskills.io/home
- **Anthropic 技术博客**：https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills

### 工具和平台

- **Claude Code**：https://code.claude.com
- **Anthropic 官方 Skills**：https://github.com/anthropics/skills
- **Skills MP 市场**：https://skillsmp.com/zh
- **CC Switch**（模型管理）：https://github.com/farion1231/cc-switch

### 社区

- **Mulerun 社区**：https://community.mulerun.com
- **GitHub Discussions**：https://github.com/anthropics/skills/discussions

---

*最后更新：2026-01-09*
*整理自：一泽Eze《Agent Skills 终极指南》*
