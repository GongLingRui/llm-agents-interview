# Shell + Skills + Compaction（OpenAI）

## Metadata

- Source / Author / Date: OpenAI Developers Blog · Charlie Guo · 2026-02-11
- URL: https://developers.openai.com/blog/skills-shell-tips
- Mapped modules: coding-agent, context-engineering, sandbox, security, agent-loop, prompt-engineering
- Last reviewed: 2026-10-07
- Reference: [article-openai-skills-shell-compaction-2026](../../../references/articles.md#article-openai-skills-shell-compaction-2026)

## Claim（作者主张）

长程 Agent 需要三原语协同：

1. **Skills**：可版本化的程序化说明（按需加载，对齐 Agent Skills 开放标准）
2. **Shell**：托管/本地执行环境（安装依赖、跑脚本、写产物）
3. **Compaction**：服务端压缩长轨迹，避免撞窗

合在一起：可重复流程 + 真执行 + 连续长跑，而不是把系统提示撑成脆弱巨文。

## Mechanism（可验证的机制）

```text
Skill metadata（name/description/path）→ 路由是否加载 SKILL.md
Shell（hosted container 或 local executor）→ 副作用与产物
Compaction（in-stream 或 /responses/compact）→ 续跑
```

关键工程 tip（文中）：

- Skill 描述写成 **路由逻辑**（use when / don’t use when），非营销文案
- 负例降低误触发（Glean：初降触发后靠负例恢复）
- 模板放进 Skill，未触发时不占 token
- 容器复用 + `previous_response_id` + 默认 compaction
- 网络：org allowlist ⊇ request allowlist；`domain_secrets` 防凭证进模型
- `/mnt/data` 作为产物交接边界

## Evidence

- OpenAI 产品原语说明 + Glean 早期客户：Salesforce skill 评测准确率 73%→85%、TTFT −18.1%（客户报告，非跨域普遍常数）。
- 明确安全警告：Skills + 开放网络 = 高数据外泄风险。

## Engineering translation（我们怎么用）

| 原语 | 本手册 |
|------|--------|
| Skills | Prompt/Context：稳定程序外置；[coding-agent](../../../agent-types/coding-agent/coding-agent.md) |
| Shell | [sandbox](../../../runtime/sandbox/sandbox.md) 执行面；托管≠免威胁模型 |
| Compaction | [context-engineering](../../../core/context-engineering/context-engineering.md) |
| 网络/密钥 | [security](../../../production/security/security.md) |
| 确定性触发 | 「显式要求用某 skill」= 政策/合同，不是靠模型自觉 |

**可执行下一步**：长任务默认设计「Skill + 隔离执行 + compaction」；网络双层 allowlist。

## Disagreement / Caveats

- 产品字段名（Responses API）易变 → TODO: Verify 集成时钉版本。
- 客户指标不可直接抄进自家 SLA。
- Compaction 有损：关键句柄/决策仍需保留策略（见 Context Canonical）。

## Action items

1. 把系统提示里的长 SOP 迁到 Skills。
2. Shell 默认最小网络；密钥走 sidecar，不进 transcript。
3. 评测 Skill 路由：正例 + 负例 + held-out。

## References

- [article-openai-skills-shell-compaction-2026](../../../references/articles.md#article-openai-skills-shell-compaction-2026)
- 对照：[sandbox](../../../runtime/sandbox/sandbox.md) · [context-engineering](../../../core/context-engineering/context-engineering.md)
