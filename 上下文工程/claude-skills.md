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
