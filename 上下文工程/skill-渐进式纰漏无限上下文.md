### PDF Skill · 渐进式披露与近似“无限上下文”
**说明**：从真实技能包出发，拆解 SKILL.md、三层加载模型，以及“认知 vs 执行”的架构思想。

#### 本期重点
- Skill.md 的结构：YAML 元数据如何让系统“知道我拥有哪些技能”。
- 渐进式披露：目录 → 章节 → 附录，按需展开、随取随用。
- 认知空间 vs 执行空间：为什么不需要真正的“无限上下文”。
- 上下文动态：Claude 如何读取 SKILL.md、forms.md，并触发脚本。

#### 补充观点
“无限上下文”不是窗口变大，而是 Skill 架构让模型像翻手册一样取用信息。

### Skill ≠ Tool，它是 Tool 的 SOP
**说明**：Tool 像烤箱，是底层执行能力；Skill 则是一份妈妈的烤鸡食谱，本身不烤鸡，却写明时间、温度、原料和小诀窍。

#### Tool · 执行器（烤箱）
- 回答：What can I do?
- 组成：单个 API / 函数（Bash、Python Runner 等）。
- 角色：被动等待调用，执行指定动作。
- 目标：提供原子能力。

#### Skill · 指挥官（食谱）
- 回答：How do I get it done?
- 组成：指令 + 脚本 + 资源 + 经验性提示。
- 角色：Orchestrator，引导 LLM 何时调用哪个 Tool。
- 目标：封装可移植、可复用的过程知识。

**补充**：Skill 不替代 Tool，而是调用 Tool 的 SOP；下一期将精读“渐进式披露”如何按需加载这些能力。

### Skill 究竟是什么？

#### Agent = 聪明但没经验的新员工
Claude 很强，但落地真实业务时会遭遇两大瓶颈：
- 过程知识缺失：不知道“如何做事”，比如报销、提交流程。
- 组织背景缺失：不知道“东西在哪”，比如 API 密钥、PPT 模板。
→ 新员工需要 SOP 入职指南；对 Agent 来说，这本指南就是 Skill。


#### Skill = SOP 入职指南
**skill定义**：“由指令、脚本和资源组成的结构化文件夹，智能体能够动态发现并加载这些内容，以提升特定任务的表现”

- **Instructions**：手册文字，告诉 Agent“一步步怎么做”，填补过程知识。
- **Scripts**：`.py`/`.sh` 脚本让 Agent 真正执行流程，是“如何做事”的具体化。
- **Resources**：`.json` 配置、`.docx` 模板等组织资产，回答“东西在哪”。

- **动态发现并加载**：Skill 不再写死在 Prompt 里；Claude 按任务检索最相关的 Skill，并用工具读取 `SKILL.md`/脚本/模板即时挂载。
- **提升特定任务表现**：加载后的 SOP 让输出更专业、一致、可复现，相当于随任务“装上能力模块”。


**总结**：Agent (LLM) = 新员工；Skill = 标准化入职手册。三要素对齐两大缺口，并支持按需加载能力模块。

### Agent + Skills + Virtual Machine
**说明**：LLM 通过 Agent 配置找到技能，再让虚拟机加载技能目录并执行脚本。

#### Agent 配置（Configuration）
- 包含：Core system prompt
- 技能标签：bigquery、docx、nda-review、pdf、pptx、xlsx
- 服务器选项：MCP server 1、MCP server 2、MCP server 3
- 说明：1 Remote MCP servers on the Internet


#### Agent 虚拟机（Virtual Machine）
- 支持环境：Bash、Python、Node.js
- 技能目录示例：
  - skills/bigquery/：SKILL.md、datasources.md、rules.md
  - skills/docx/：SKILL.md、ooxml/、spec.md、editing.md
  - skills/nda-review/：SKILL.md
  - skills/pdf/：SKILL.md、forms.md、reference.md、extract_fields.py
- 说明：技能目录真实存放在 Agent “电脑”(VM) 的文件系统中。

### Skill ≠ Tool，它是 Tool 的 SOP
**说明**：Tool 像烤箱，是底层执行能力；Skill 则是一份妈妈的烤鸡食谱，本身不烤鸡，却写明时间、温度、原料和小诀窍。

#### Tool · 执行器（烤箱）
- 回答：What can I do?
- 组成：单个 API / 函数（Bash、Python Runner 等）。
- 角色：被动等待调用，执行指定动作。
- 目标：提供原子能力。

#### Skill · 指挥官（食谱）
- 回答：How do I get it done?
- 组成：指令 + 脚本 + 资源 + 经验性提示。
- 角色：Orchestrator，引导 LLM 何时调用哪个 Tool。
- 目标：封装可移植、可复用的过程知识。

**补充**：Skill 不替代 Tool，而是调用 Tool 的 SOP；下一期将精读“渐进式披露”如何按需加载这些能力。

### PDF Skill · 渐进式披露与近似“无限上下文”
**说明**：从真实技能包出发，拆解 SKILL.md、三层加载模型，以及“认知 vs 执行”的架构思想。

#### 本期重点
- Skill.md 的结构：YAML 元数据如何让系统“知道我拥有哪些技能”。
- 渐进式披露：目录 → 章节 → 附录，按需展开、随取随用。
- 认知空间 vs 执行空间：为什么不需要真正的“无限上下文”。
- 上下文动态：Claude 如何读取 SKILL.md、forms.md，并触发脚本。

#### 补充观点
“无限上下文”不是窗口变大，而是 Skill 架构让模型像翻手册一样取用信息。

### Skill.md = 目录化的技能声明
**说明**：系统只加载 YAML 元数据就能索引技能；正文和附录在触发时再展开。

#### pdf/SKILL.md（中文翻译）
- **YAML Frontmatter**
  ```yaml
  name: pdf
  description: 综合 PDF 工具包 —— 提取文本/表单、合并文档、填写表单。
  ```
- **Markdown 正文**
  - ## 概览
    本指南覆盖 PDF 处理的关键操作；更多示例见 reference.md。
  - ## Quick Start
    ```python
    from pypdf import PdfReader, PdfWriter
    reader = PdfReader("document.pdf")
    ```
  - 表单填写：阅读 ./forms.md 并按流程操作。


#### 元数据为何重要？
- Agent 启动时扫描所有 SKILL.md，只把 `name + description` 写进系统提示 → Claude 立即知道“我拥有哪些技能”，但不必一次读完正文。
- 当任务匹配（例如“处理 PDF”），Claude 才会请求该技能的 Markdown 正文，并跟随其中的引用（forms.md、reference.md……）。
- 这是渐进式披露的第一层：目录可见、内容按需展开。

### 渐进式披露：像翻一本手册
**说明**：先看目录，再读章节，最后只在需要时翻附录。

| Level | File                          | Context Window          | #Tokens  | 作用                                       |
|-------|-------------------------------|-------------------------|----------|--------------------------------------------|
| 1 目录  | SKILL.md 元数据 (YAML)         | Always loaded           | ~100     | 告诉 Claude “有哪些技能 / 用途”             |
| 2 章节  | SKILL.md 正文 (Markdown)       | Loaded when Skill triggers | <5k      | 技能被触发时加载 SOP                       |
| 3+ 附录 | Bundled files（文本、脚本、数据） | Loaded as needed        | unlimited* | forms.md、脚本、配置；被引用时再读取/执行 |

**总结**：目录被索引、章节被触发、附录被引用 —— Skill 让 Claude 像翻书一样按需加载。

### 认知 vs 执行：让“无限上下文”成为可能
**说明**：思考空间只保留必要文本；行动空间负责执行脚本、读取大文件。

#### 认知空间 · Cognition
- 包含系统提示、技能元数据、被触发技能的 Markdown 正文。
- 所有推理、总结都在这里完成；体量小、易压缩。
- 随着任务推进，仅保留本轮真正需要的信息。

#### 执行空间 · Execution
- 包含 forms.md、reference.md、.py 脚本、PDF 等真实文件。
- Claude 通过工具调用脚本，结果再回写上下文。
- 代码执行确定、可复现，也比“用语言描述”更高效。


#### 示例：forms.md 指令 & extract_fields.py
- **forms.md 指令**
  1. 检查 PDF 是否可填写。
  2. 若可填写 → 运行 `python scripts/extract_fields.py`。
  3. 输出 forms.json。
- **extract_fields.py**
  ```python
  from pypdf import PdfReader
  字段 → json.dump()
  ```
  通过 Bash 执行，Claude 只接收“脚本完成，生成 forms.json”的结果。

  <img width="721" height="539" alt="image" src="https://github.com/user-attachments/assets/a553b1bb-b804-4b41-a5f3-04e74c450203" />

### Skill 触发时，上下文如何刷新？
**说明**：忠实还原原文示意：System Prompt + 元数据 → 触发 pdf → 读取 forms → 执行脚本。

#### 步骤说明
1. **启动**：上下文 = 系统提示 + 技能元数据 + 用户输入。
2. **技能触发**：Claude 判断任务需 pdf → 读取 SKILL.md 正文。
3. **附录展开**：根据引用读取 forms.md。
4. **执行脚本**：若 SOP 要求，调用脚本，结果写回上下文。
5. **完成**：综合 SOP + 工具结果 + 用户需求给出回答。


#### 上下文窗口（Context Window）示例流程
- 系统提示包含技能元数据碎片（如 bigquery、docx、nda-review、pdf、pptx、xlsx 等）。
- 用户输入：*“基于你对我的了解填写这份 PDF 表单，附件：/mnt/uploads/order_form.pdf”*
- Claude 响应：*“没问题，我会用 PDF Skill 来帮忙。”*
- 工具调用：`Bash("cat /mnt/skills/pdf/SKILL.md")` → 工具结果返回 *“…SKILL.md 的正文…”*
- 工具调用：`Bash("cat /mnt/skills/pdf/forms.md")` → 工具结果返回 *“…forms.md 的内容…”*

### 结构・机制・原理 —— 三层结论
- **结构：Skill = 文件夹 + SKILL.md**
  YAML 元数据被预加载，正文/附录按需展开。
- **机制：渐进式披露**
  目录 (~100 tokens) → 章节 (<5k) → 附录 (unlimited)。Claude 像翻手册一样读取。
- **原理：认知与执行分离**
  思考空间保持轻量，执行空间可无限扩展；脚本带来确定性与效率。

**补充观点**：“近似无限上下文” = 让模型的上下文只装需要思考的那一页，把真实世界留在它随时可访问的文件系统里。




