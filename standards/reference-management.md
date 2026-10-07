# Reference 管理规范

> 主题：统一管理论文、文章、仓库、文档引用，保证可追溯与可去重。

## 原则

1. **引用集中登记**：权威条目进入 `references/`
2. **正文只保留短引用**：文末用稳定 ID 链接到 references
3. **一源一条**：同一 URL / DOI / repo 不重复建条目
4. **可复核**：每条包含获取日期与类型

## ID 格式

```text
<type>-<short-slug>-<year?>

类型前缀：
- paper-   论文
- article- 文章/博客
- repo-    开源仓库
- doc-     官方文档
- talk-    演讲
- book-    书籍
- std-     标准/规范（如 MCP spec）
```

示例：

- `paper-react-2022`
- `article-anthropic-building-effective-agents-2024`
- `repo-openai-swarm`
- `std-mcp-specification`

## 条目模板

在 `references/` 中按类型分文件维护索引（见该目录 README）。单条字段：

```markdown
### <ID>
- Title:
- Type:
- Authors/Org:
- Year:
- URL:
- DOI/Repo: (optional)
- Topics: (handbook modules)
- Summary: (1–2 sentences, mechanism-focused)
- Added: YYYY-MM-DD
- Last checked: YYYY-MM-DD
```

## 正文引用写法

```markdown
参见 [paper-react-2022](../references/papers.md#paper-react-2022)。
```

（正文若在更深目录，按相对路径调整，例如 `../../references/papers.md#…`。）

或在 References 小节：

```markdown
## References
- [paper-react-2022](../references/papers.md#paper-react-2022)
- [repo-...](...)
```

## 去重流程

新增引用前：

1. 在 `references/` 搜索 URL / 标题 / slug
2. 已存在 → 复用 ID，可补充 Topics
3. 不存在 → 新建条目，再在正文引用

## 与研究目录的关系

| 产物 | 是否替代 references 条目 |
|------|--------------------------|
| 论文深度分析 | 否，分析文 + reference 条目都要 |
| 文章笔记 | 否 |
| 开源分析 | 否，repo 条目仍要登记 |

## 禁止

- 在多个模块各维护一份「神级链接列表」且内容漂移
- 无 Last checked 的永久死链堆
- 用「参考资料很多」代替结构化 ID
