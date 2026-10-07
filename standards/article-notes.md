# 文章笔记规范

> 主题：如何记录官方博文、工程博客、演讲笔记，避免知识库变成链接坟场。

## 适用对象

- 厂商工程博客（Anthropic / OpenAI / Cursor 等）
- 团队技术分享、会议演讲
- 高质量个人工程长文

落点规则：

| 内容类型 | 目录 |
|----------|------|
| 厂商方法论 / 工程实践提炼 | `research/industry-practices/` |
| 端到端系统故事 | `research/case-studies/` |
| 直接增强某机制理解的短注 | 对应 `core/` / `production/` 主题文的「延伸阅读」小节（链到 references） |

## 笔记模板

```markdown
# <Article Title>

## Metadata
- Source / Author / Date
- URL
- Mapped modules
- Last reviewed

## Claim（作者主张）
## Mechanism（可验证的机制）
## Evidence（数据 / 案例 / 代码）
## Engineering translation（我们怎么用）
## Disagreement / Caveats（保留怀疑）
## Action items（可执行下一步）
## References
```

## 三层提取法（强制）

1. **Claim**：文章说了什么
2. **Mechanism**：底层机制是什么（能画图）
3. **Decision**：工程上因此做什么/不做什么

缺 Mechanism 的笔记不得标为 Canonical。

## 厂商文章特别规则

- 默认存在产品叙事偏差；必须写 Caveats
- 把术语对齐到本仓库词汇（例如把「agentic workflow」映射到 Loop / Tools / Eval）
- 同一主题多篇文章：合并对照，不一人一文无限新增文件

## 禁止

- 只收藏 URL + 一句话夸赞
- 无映射模块的「杂感」
- 大段复制原文（版权与可维护性双重问题）
