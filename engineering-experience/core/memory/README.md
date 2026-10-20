# memory

> **Status**：有经验沉淀
>
> **概念主文**：[`concepts/core/memory/`](../../../concepts/core/memory/)

## 本目录只写

- 生产/项目中的可复现经验、门槛数字、失败案例、playbook
- 不重复 Canonical 机制定义

## 经验索引

- [Agent Memory 设计方法论：先判断必要性，再决定复杂度](./agent-memory-design-methodology.md) — 做 memory 前的第一问「为什么需要记忆」：必要性五维度、设计六环节（判断→抽取→去重→分层存储→检索→动态维护）、长期/短期记忆区分、复杂度按业务定
- [Agent Memory 架构：五层记忆 Playbook](./agent-memory-5-layer-playbook.md) — 五层记忆架构（工作/情节/语义/程序性/遗忘）的完整翻译整理，含 Mem0（90% token 削减）、Snowflake（本体 +20% 准确率）等生产证据、实现代码、决策框架与生产检查清单
- [Agent Memory 架构全景：从规则文件到反思与技能沉淀](./agent-memory-architecture-panorama.md) — 应用层 memory 四层架构（规则/常驻/历史/进化）+ 治理记忆，含四条设计原则、长上下文 vs Memory 的定位、Memory 带来的五类新问题
