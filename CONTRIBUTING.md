# Contributing

欢迎贡献。本仓库优先保证 **结构清晰、主题唯一、可长期维护**，而不是短时间堆内容。

本仓库采用 [MIT](LICENSE) 许可（根目录 `LICENSE`）。

## 顶层落点（先选对支柱）

| 内容类型 | 放哪里 |
|----------|--------|
| 概念 / 机制 / 研究 / 练习 / 引用 | [`concepts/`](concepts/) |
| 落地经验、门禁数字、playbook、踩坑 | [`engineering-experience/`](engineering-experience/) |
| 课程笔记与作业 | **独立课程文件夹**（如根目录 `CS329A自我改进智能体/`），不进双支柱 |

禁止在概念文里夹私有经验流水账，也禁止在经验文里复制整章 Canonical。

## 贡献前必读

1. [仓库架构](concepts/docs/architecture.md) — 主题应该落在哪一层
2. [知识依赖关系](concepts/docs/knowledge-graph.md) — 是否写在正确依赖位置
3. [文档质量标准](concepts/docs/quality-standards.md) — 什么叫「可合并」

## 贡献类型（concepts/）

| 类型 | 放哪里 | 要求 |
|------|--------|------|
| 概念 / 机制讲解 | `concepts/foundations/` `concepts/core/` `concepts/runtime/` `concepts/production/` | 说明依赖前置、边界、工程含义 |
| 场景 Agent | `concepts/agent-types/` | 必须关联到 Agent Loop / Tools / Memory 等核心机制 |
| 协议 | `concepts/protocols/` | 与 Tool Use 区分：协议 ≠ 调用策略 |
| 论文笔记 | `concepts/research/papers/` | 遵守论文分析规范 |
| 开源拆解 | `concepts/research/open-source-analysis/` | 遵守开源分析规范 |
| 厂商工程实践提炼 | `concepts/research/industry-practices/` | 原文 + 工程提炼，不做软文搬运 |
| 案例 | `concepts/research/case-studies/` | 端到端系统视角 |
| 项目练习 | `concepts/practice/agent-projects/` | 目标、验收标准、依赖模块明确 |
| 面试题 | `concepts/practice/interview-questions/` | 考点映射到模块，避免纯背诵题 |
| 系统设计 | `concepts/practice/system-design/` | 有架构图位、权衡、失败模式 |
| 引用条目 | `concepts/references/` | 统一 Reference 规范 |
| 工程经验 | `engineering-experience/<layer>/<module>/` | 回链对应 Canonical；只写可复现经验 |

## 禁止事项

- 在多个目录复制同一主题正文
- 概念与工程经验混写在同一文件
- 为了「显得完整」创建无职责空文件（`engineering-experience/` 占位 README 除外）
- 把 Prompt 技巧、RAG tricks、博客摘要伪装成 Agent Engineering
- 未声明来源的大段转载
- 与 Agent Engineering 无关的通用 ML 教程灌入
- 把课程材料塞进 `concepts/` 或 `engineering-experience/`

## PR 检查清单

- [ ] 主题落在唯一主目录；交叉引用用链接
- [ ] 已选对支柱（concepts vs engineering-experience vs 课程目录）
- [ ] README / 模块索引已更新（如新增主题）
- [ ] 前置依赖写清楚
- [ ] 符合 `concepts/docs/quality-standards.md`

## 建议工作流

1. 先开 Issue 说明：主题、目标读者、落点目录、依赖模块
2. 按规范写骨架（标题 / 范围 / 非目标 / 大纲）
3. 再填正文；概念文优先「机制 → 工程含义 → 失败模式」；经验文优先「场景 → 决策 → 数字/结果 → 回链概念」
4. 更新相关路线图链接（如有必要）
