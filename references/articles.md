# Article References

> ID 前缀：`article-`

### article-anthropic-building-effective-agents-2024
- Title: Building effective agents
- Type: article
- Authors/Org: Anthropic
- Year: 2024
- URL: https://www.anthropic.com/engineering/building-effective-agents
- Topics: core/agent-loop, core/reasoning-planning, core/agent-architecture, production/design-patterns, core/tool-use, agent-types/multi-agent, agent-types/coding-agent, runtime/sandbox
- Summary: 主张用简单可组合模式构建 Agent；区分 workflows 与 agents；强调工具文档（ACI）、可观测与按需增加复杂度。
- Added: 2026-10-07
- Last checked: 2026-10-07

### article-anthropic-writing-tools-for-agents-2025
- Title: Writing effective tools for AI agents—using AI agents
- Type: article
- Authors/Org: Anthropic
- Year: 2025
- URL: https://www.anthropic.com/engineering/writing-tools-for-agents
- Topics: core/tool-use, core/prompt-engineering, production/evaluation, agent-types/coding-agent, runtime/sandbox
- Summary: 工具设计与评测驱动迭代：清晰职责、上下文效率、可组合工作流；用评测改进工具描述与规格。
- Added: 2026-10-07
- Last checked: 2026-10-07

### article-anthropic-context-engineering-2025
- Title: Effective context engineering for AI agents
- Type: article
- Authors/Org: Anthropic
- Year: 2025
- URL: https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents
- Topics: core/context-engineering, core/agent-loop, core/memory, agent-types/coding-agent
- Summary: 将 context 视为有限注意力预算；Context Engineering = 在推理时策展最优 token 集（含 prompt 之外动态信息）；强调高信号最小集、JIT 检索、compaction 等长程策略。
- Added: 2026-10-07
- Last checked: 2026-10-07

### article-anthropic-multi-agent-research-2025
- Title: How we built our multi-agent research system
- Type: article
- Authors/Org: Anthropic
- Year: 2025
- URL: https://www.anthropic.com/engineering/multi-agent-research-system
- Topics: agent-types/multi-agent, core/agent-architecture, production/evaluation, production/observability, core/context-engineering, core/tool-use
- Summary: 生产 Research 功能的 orchestrator–worker 多 Agent；并行子 Agent 压缩搜索；强调委派提示、成本（~15× chat）、Eval 与 tracing、可恢复执行。
- Added: 2026-10-07
- Last checked: 2026-10-07

### article-cursor-dynamic-context-discovery-2026
- Title: Dynamic context discovery
- Type: article
- Authors/Org: Cursor
- Year: 2026
- URL: https://cursor.com/blog/dynamic-context-discovery
- Topics: core/context-engineering, agent-types/coding-agent, protocols/mcp, core/tool-use
- Summary: 主张少静态预置、多按需发现；用文件承载长工具结果/历史/Skills/MCP 描述/终端；Host 侧削减 MCP 工具上下文（文内 A/B 约 −46.9% tokens）。
- Added: 2026-10-07
- Last checked: 2026-10-07

### article-openai-skills-shell-compaction-2026
- Title: Shell + Skills + Compaction: Tips for long-running agents that do real work
- Type: article
- Authors/Org: OpenAI
- Year: 2026
- URL: https://developers.openai.com/blog/skills-shell-tips
- Topics: agent-types/coding-agent, core/context-engineering, runtime/sandbox, production/security
- Summary: 长程 Agent 三原语：Skills（按需程序）、Shell（执行面）、Compaction（续窗）；含网络双层 allowlist 与 domain_secrets 安全建议。
- Added: 2026-10-07
- Last checked: 2026-10-07
