# 开源项目分析规范

> 主题：如何拆解一个开源 Agent 项目，并沉淀为可复用工程知识。

## 目标

分析的终点不是「这个项目很火」，而是回答：

1. 它的 **Agent Loop** 怎么转？
2. **状态** 存在哪？
3. **工具面** 如何设计？
4. 哪些设计可迁移到我们的系统？

落点目录：`research/open-source-analysis/`

## 选型标准（值得写进仓库）

满足至少 3 条再立项：

- 有清晰的 Agent 控制回路（不是纯 chat UI）
- 工具调用 / 沙箱 / 记忆 / 多 Agent 至少深挖其一
- 代码可阅读，架构可映射到本仓库模块
- 对 Production 有启发（Eval、Obs、Security、Infra 任一）
- 社区或工程上有代表性（不必是 Star 最多）

## 文档模板（强制结构）

与本仓已采用笔记一致（见 `research/open-source-analysis/` 下 vLLM / MCP / Agents SDK）：

```markdown
# OSS: <Project Name>
（协议/规格可写 `# OSS / Spec: …`）

## Metadata
- Repo / License / Language:（链 `references/repos.md` ID；License 见上游）
- Last reviewed
- Canonical modules mapped:（handbook 模块名，逗号分隔）

## Scope / Non-goals
**范围**：机制导向一句话 + 本笔记要拆什么  
**非目标**：不复述官方教程、不替代 Canonical、不枚举生态清单等  
**核实基线**：repo / spec / 文档锚点（随版本演进需对照）

## Agent Loop 映射
| Agent / 手册概念 | 该项目角色 |
|------------------|------------|
| … | … |

（可选：控制面 / 数据面或 Host↔协议 的 ASCII 示意）

## Tool surface
工具接口、权限/沙箱、MCP 或插件模型；没有工具面则写清边界（如「不在本系统内执行工具」）

## State / Memory / Context
状态放哪、是否等于 Agent Memory；没有就写 N/A，不要硬编

## Eval / Obs hooks
可挂钩的字段、span、失败分类；与 observability / evaluation 模块对齐的建议

## Reusable designs
可迁移设计（编号列表）

## Anti-patterns
陷阱与不应照搬之处

## Mapped handbook modules
链到本仓 Canonical / 模块页（标明主模块）

## References
链 `references/*.md` ID；必要时补官方文档名（勿大段粘贴）
```

## 分析深度要求

| 级别 | 要求 |
|------|------|
| L1 地图 | Metadata + Scope + Loop 映射表 + 模块链接 |
| L2 机制 | 读关键控制路径（或协议路径），说明状态/终止/工具边界 |
| L3 对照 | 与至少一种替代设计对比权衡 |

合并为 Canonical 分析至少达到 **L2**。

## 禁止

- 只贴 README 翻译
- 按文件列表流水账
- 无代码/协议依据的架构脑补
- 把框架入门教程整页搬进分析
- 用营销语代替机制说明

## 与其他目录的边界

- 可复用模式提炼后，链到 `production/design-patterns/`，不要在分析文里开模式分册
- 项目练习复现放到 `practice/agent-projects/`，分析文只保留结论与链接
- 协议 Canonical 在 `protocols/`；OSS 笔记做互操作/映射，不替代 Canonical 全文
