# Contributing

欢迎贡献。本仓库优先保证 **结构清晰、主题唯一、可长期维护**，而不是短时间堆内容。

本仓库采用 [MIT](LICENSE) 许可（根目录 `LICENSE`）。

## 贡献前必读

1. [仓库架构](docs/architecture.md) — 主题应该落在哪一层
2. [知识依赖关系](docs/knowledge-graph.md) — 是否写在正确依赖位置
3. [文档质量标准](docs/quality-standards.md) — 什么叫「可合并」
4. 对应写作规范：
   - [论文分析](standards/paper-analysis.md)
   - [开源项目分析](standards/open-source-analysis.md)
   - [文章笔记](standards/article-notes.md)
   - [Reference 管理](standards/reference-management.md)

## 贡献类型

| 类型 | 放哪里 | 要求 |
|------|--------|------|
| 概念 / 机制讲解 | `foundations/` `core/` `runtime/` `production/` | 说明依赖前置、边界、工程含义 |
| 场景 Agent | `agent-types/` | 必须关联到 Agent Loop / Tools / Memory 等核心机制 |
| 协议 | `protocols/` | 与 Tool Use 区分：协议 ≠ 调用策略 |
| 论文笔记 | `research/papers/` | 遵守论文分析规范 |
| 开源拆解 | `research/open-source-analysis/` | 遵守开源分析规范 |
| 厂商工程实践 | `research/industry-practices/` | 原文 + 工程提炼，不做软文搬运 |
| 案例 | `research/case-studies/` | 端到端系统视角 |
| 项目练习 | `practice/agent-projects/` | 目标、验收标准、依赖模块明确 |
| 面试题 | `practice/interview-questions/` | 考点映射到模块，避免纯背诵题 |
| 系统设计 | `practice/system-design/` | 有架构图位、权衡、失败模式 |
| 引用条目 | `references/` | 统一 Reference 规范 |

## 禁止事项

- 在多个目录复制同一主题正文
- 为了「显得完整」创建无职责空文件
- 把 Prompt 技巧、RAG tricks、博客摘要伪装成 Agent Engineering
- 未声明来源的大段转载
- 与 Agent Engineering 无关的通用 ML 教程灌入

## PR 检查清单

- [ ] 主题落在唯一主目录；交叉引用用链接
- [ ] README / 模块索引已更新（如新增主题）
- [ ] 前置依赖写清楚
- [ ] 引用按 `standards/reference-management.md` 登记
- [ ] 符合 `docs/quality-standards.md`

## 建议工作流

1. 先开 Issue 说明：主题、目标读者、落点目录、依赖模块
2. 按规范写骨架（标题 / 范围 / 非目标 / 大纲）
3. 再填正文；正文优先「机制 → 工程含义 → 失败模式」
4. 更新相关路线图链接（如有必要）
