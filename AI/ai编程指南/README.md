<p align="center">
<img src="https://github.com/tukuaiai.png" alt="Vibe Coding 指南" width="80px">
</p>

<div align="center">

# Vibe Coding 终极指南 V1.2

**一个通过与 AI 结对编程，将想法变为现实的终极工作流程。**

**作者：** [Nicolas Zullo, https://x.com/NicolasZu](https://x.com/NicolasZu)  
**译者：** [tukuaiai](https://github.com/tukuaiai), [https://x.com/123olp](https://x.com/123olp)

**创建日期：** 2025年3月12日  
**最后更新日期：** 2025年10月6日

---

<p>
  <a href="https://github.com/tukuaiai/vibe-coding-cn/actions"><img src="https://img.shields.io/github/actions/workflow/status/tukuaiai/vibe-coding-cn/main.yml?style=for-the-badge" alt="构建状态"></a>
  <a href="https://github.com/tukuaiai/vibe-coding-cn/releases"><img src="https://img.shields.io/github/v/release/tukuaiai/vibe-coding-cn?style=for-the-badge" alt="最新版本"></a>
  <a href="LICENSE"><img src="https://img.shields.io/github/license/tukuaiai/vibe-coding-cn?style=for-the-badge" alt="许可证"></a>
  <a href="https://github.com/tukuaiai/vibe-coding-cn"><img src="https://img.shields.io/github/languages/top/tukuaiai/vibe-coding-cn?style=for-the-badge" alt="主要语言"></a>
  <a href="https://github.com/tukuaiai/vibe-coding-cn"><img src="https://img.shields.io/github/languages/code-size/tukuaiai/vibe-coding-cn?style=for-the-badge" alt="代码大小"></a>
  <a href="https://github.com/tukuaiai/vibe-coding-cn/graphs/contributors"><img src="https://img.shields.io/github/contributors/tukuaiai/vibe-coding-cn?style=for-the-badge" alt="贡献者"></a>
</p>

[🚀 入门指南](#-入门指南) •
[⚙️ 完整设置流程](#️-完整设置流程) •
[🎮 Vibe Coding 开发](#-vibe-coding-开发基础游戏) •
[🐞 问题修复](#-修复-bug-与卡壳情况) •
[📚 相关文档](#-相关文档) •
[🤝 参与贡献](#-参与贡献)

</div>

---

## 🚀 入门指南
要开始 Vibe Coding，你只需要以下两种工具之一：
- **Claude Sonnet 4.5**，在 Claude Code 中使用
- **gpt-5-codex (high)**，在 Codex CLI 中使用

本指南同时适用于 CLI 终端版本和 VSCode 扩展版本（Codex 和 Claude Code 都有扩展，且界面更新）。

*(注：本指南早期版本使用的是 **Grok 3**，后来切换到 **Gemini 2.5 Pro**，现在我们使用的是 **Claude 4.5**（或 **gpt-5-codex (high)**）)*

*(注2：如果你想使用 Cursor，请查看本指南的 [1.1 版本](https://github.com/EnzeD/vibe-coding/tree/1.1.1)，但我们认为它目前不如 Codex CLI 或 Claude Code 强大)*

正确设置一切至关重要。如果你认真想做一个功能完整、外观精美的游戏（或应用），请花时间打好基础。

> **核心原则：** *规划就是一切。* 绝不要让 AI 自主规划，否则你的代码库会变成一团无法管理的乱麻。

---

## ⚙️ 完整设置流程

<details>
<summary><strong>1. 游戏设计文档（Game Design Document）</strong></summary>

- 把你的游戏创意交给 **GPT-5** 或 **Sonnet 4.5**，让它生成一份简洁的 **游戏设计文档**，格式为 Markdown，文件名为 `game-design-document.md`。
- 自己审阅并完善，确保与你的愿景一致。初期可以很简陋，目标是给 AI 提供游戏结构和意图的上下文。不要过度设计，后续会迭代。
</details>

<details>
<summary><strong>2. 技术栈与 <code>CLAUDE.md</code> / <code>Agents.md</code></strong></summary>

- 让 **GPT-5** 或 **Sonnet 4.5** 为你的游戏推荐最合适的技术栈（例如：多人3D游戏用 ThreeJS + WebSocket），保存为 `tech-stack.md`。
  - 要求它提出 **最简单但最健壮** 的技术栈。
- 在终端中打开 **Claude Code** 或 **Codex CLI**，使用 `/init` 命令，它会读取你已创建的两个 .md 文件，生成一套规则来正确引导大模型。
- **关键：一定要审查生成的规则。** 确保规则强调 **模块化**（多文件）和禁止 **单体巨文件**（monolith）。可能需要手动修改或补充规则。
  - **极其重要：** 某些规则必须设为 **"Always"**（始终应用），确保 AI 在生成任何代码前都强制阅读。例如添加以下规则并标记为 "Always"：
    > ```
    > # 重要提示：
    > # 写任何代码前必须完整阅读 memory-bank/@architecture.md（包含完整数据库结构）
    > # 写任何代码前必须完整阅读 memory-bank/@game-design-document.md
    > # 每完成一个重大功能或里程碑后，必须更新 memory-bank/@architecture.md
    > ```
  - 其他（非 Always）规则要引导 AI 遵循你技术栈的最佳实践（如网络、状态管理等）。
  - *如果想要代码最干净、项目最优化，这一整套规则设置是强制性的。*
</details>

<details>
<summary><strong>3. 实施计划（Implementation Plan）</strong></summary>

- 将以下内容提供给 **GPT-5** 或 **Sonnet 4.5**：
  - 游戏设计文档（`game-design-document.md`）
  - 技术栈推荐（`tech-stack.md`）
- 让它生成一份详细的 **实施计划**（Markdown 格式），包含一系列给 AI 开发者的分步指令。
  - 每一步要小而具体。
  - 每一步都必须包含验证正确性的测试。
  - 严禁包含代码——只写清晰、具体的指令。
  - 先聚焦于 **基础游戏**，完整功能后面再加。
</details>

<details>
<summary><strong>4. 记忆库（Memory Bank）</strong></summary>

- 新建项目文件夹，并在 VSCode 中打开。
- 在项目根目录下创建子文件夹 `memory-bank`。
- 将以下文件放入 `memory-bank`：
  - `game-design-document.md`
  - `tech-stack.md`
  - `implementation-plan.md`
  - `progress.md`（新建一个空文件，用于记录已完成步骤）
  - `architecture.md`（新建一个空文件，用于记录每个文件的作用）
</details>

---

## 🎮 Vibe Coding 开发基础游戏
现在进入最爽的阶段！

<details>
<summary><strong>确保一切清晰</strong></summary>

- 在 VSCode 扩展中打开 **Codex** 或 **Claude Code**，或者在项目终端启动 Claude Code / Codex CLI。
- 提示词：阅读 `/memory-bank` 里所有文档，`implementation-plan.md` 是否完全清晰？你有哪些问题需要我澄清，让它对你来说 100% 明确？
- 它通常会问 9-10 个问题。全部回答完后，让它根据你的回答修改 `implementation-plan.md`，让计划更完善。
</details>

<details>
<summary><strong>你的第一个实施提示词</strong></summary>

- 打开 **Codex** 或 **Claude Code**（扩展或终端）。
- 提示词：阅读 `/memory-bank` 所有文档，然后执行实施计划的第 1 步。我会负责跑测试。在我验证测试通过前，不要开始第 2 步。验证通过后，打开 `progress.md` 记录你做了什么供后续开发者参考，再把新的架构洞察添加到 `architecture.md` 中解释每个文件的作用。
- **永远** 先用 "Ask" 模式或 "Plan Mode"（Claude Code 中按 `shift+tab`），确认满意后再让 AI 执行该步骤。
- **极致 Vibe：** 安装 [Superwhisper](https://superwhisper.com)，用语音随便跟 Claude 或 GPT-5 聊天，不用打字。
</details>

<details>
<summary><strong>工作流</strong></summary>

- 完成第 1 步后：
  - 把改动提交到 Git（不会用就问 AI）。
  - 新建聊天（`/new` 或 `/clear`）。
  - 提示词：阅读 memory-bank 所有文件，阅读 progress.md 了解之前的工作进度，然后继续实施计划第 2 步。在我验证测试前不要开始第 3 步。
- 重复此流程，直到整个 `implementation-plan.md` 全部完成。
</details>

---

## ✨ 添加细节功能
恭喜！你已经做出了基础游戏！可能还很粗糙、缺少功能，但现在可以尽情实验和打磨了。
- 想要雾效、后期处理、特效、音效？更好的飞机/汽车/城堡？绝美天空？
- 每增加一个主要功能，就新建一个 `feature-implementation.md`，写短步骤+测试。
- 继续增量式实现和测试。

---

## 🐞 修复 Bug 与卡壳情况
<details>
<summary><strong>常规修复</strong></summary>

- 如果某个提示词失败或搞崩了项目：
  - Claude Code 用 `/rewind` 回退；用 GPT-5 的话多提交 git，需要时 reset。
- 报错处理：
  - **JavaScript 错误：** 打开浏览器控制台（F12），复制错误，贴给 AI；视觉问题截图发给它。
  - **懒人方案：** 安装 [BrowserTools](https://browsertools.agentdesk.ai/installation)，自动复制错误和截图。
</details>

<details>
<summary><strong>疑难杂症</strong></summary>

- 实在卡住：
  - 回退到上一个 git commit（`git reset`），换新提示词重试。
- 极度卡壳：
  - 用 [RepoPrompt](https://repoprompt.com/) 或 [uithub](https://uithub.com/) 把整个代码库合成一个文件，然后丢给 **GPT-5 或 Claude** 求救。
</details>

---

## 💡 技巧与窍门

<details>
<summary><strong>Claude Code & Codex 使用技巧</strong></summary>

- **终端版 Claude Code / Codex CLI：** 在 VSCode 终端里运行，能直接看 diff、喂上下文，不用离开工作区。
- **Claude Code 的 `/rewind`：** 迭代跑偏时一键回滚到之前状态。
- **自定义命令：** 创建像 `/explain $参数` 这样的快捷命令，触发提示词：“深入分析代码，彻底理解 $参数 是怎么工作的。理解完告诉我，我再给你任务。” 让模型先拉满上下文再改代码。
- **清理上下文：** 经常用 `/clear` 或 `/compact`（保留历史对话）。
- **省时大法（风险自负）：** 用 `claude --dangerously-skip-permissions` 或 `codex --yolo`，彻底关闭确认弹窗。
</details>

<details>
<summary><strong>其他实用技巧</strong></summary>

- **小修改：** 用 GPT-5 (medium)
- **写顶级营销文案：** 用 Opus 4.1
- **生成优秀 2D 精灵图：** 用 ChatGPT + Nano Banana
- **生成音乐：** 用 Suno
- **生成音效：** 用 ElevenLabs
- **生成视频：** 用 Sora 2
- **提升提示词效果：**
  - 加一句：“慢慢想，不着急，重要的是严格按我说的做，执行完美。如果我表达不够精确请提问。”
  - 在 Claude Code 中触发深度思考的关键词强度：`think` < `think hard` < `think harder` < `ultrathink`。
</details>

---

## ❓ 常见问题解答
<details>
<summary><strong>Q：我在做应用不是游戏，这个流程一样吗？</strong></summary>
**A：** 基本完全一样！把 GDD 换成 PRD（产品需求文档）即可。你也可以先用 v0、Lovable、Bolt.new 快速原型，再把代码搬到 GitHub，然后克隆到本地用本指南继续开发。
</details>

<details>
<summary><strong>Q：你那个空战游戏的飞机模型太牛了，但我一个提示词做不出来！</strong></summary>
**A：** 那不是一个提示词，是 ~30 个提示词 + 专门的 `plane-implementation.md` 文件引导的。用精准指令如“在机翼上为副翼切出空间”，而不是“做一个飞机”这种模糊指令。
</details>

<details>
<summary><strong>Q：为什么现在 Claude Code 或 Codex CLI 比 Cursor 更强？</strong></summary>
**A：** 完全看个人喜好。我们强调的是：Claude Code 能更好发挥 Claude Sonnet 4.5 的实力，Codex CLI 能更好发挥 GPT-5 的实力，而 Cursor 对这两者的利用都不如原生终端版。终端版还能在任意 IDE、使用 SSH 远程服务器等场景工作，自定义命令、子代理、钩子等功能也能长期大幅提升开发质量和速度。最后，即使你只是低配 Claude 或 ChatGPT 订阅，也完全够用。
</details>

<details>
<summary><strong>Q：我不会搭建多人游戏的服务器怎么办？</strong></summary>
**A：** 问你的 AI。
</details>

---

## 📚 相关文档

- [**原始中文版文档.md**](./原始中文版文档.md) - Vibe Coding 指南的原始中文版本。
- [**系统提示词构建 105 原则.md**](./系统提示词构建%20105%20原则.md) - 构建高效、可靠的AI系统提示词的105条核心原则。
- [**经验.md**](./经验.md) - 包含一系列编程原则、架构思想和技术栈（如微服务、Redis）的经验总结。

---

## 🤝 参与贡献

我们热烈欢迎各种形式的贡献！如果您对本项目有任何想法或建议，请随时开启一个 [Issue](https://github.com/tukuaiai/vibe-coding-cn/issues) 或提交一个 [Pull Request](https://github.com/tukuaiai/vibe-coding-cn/pulls)。

### ✨ 贡献者们

感谢所有为本项目做出贡献的开发者！

<a href="https://github.com/tukuaiai/vibe-coding-cn/graphs/contributors">
  <img src="https://contrib.rocks/image?repo=tukuaiai/vibe-coding-cn" />
</a>

---

## 📜 许可证

本项目采用 [MIT](LICENSE) 许可证。

---

<div align="center">

**如果这个项目对您有帮助，请不要吝啬您的 Star ⭐！**

## Star History

<a href="https://www.star-history.com/#tukuaiai/vibe-coding-cn&type=date&legend=top-left">
 <picture>
   <source media="(prefers-color-scheme: dark)" srcset="https://api.star-history.com/svg?repos=tukuaiai/vibe-coding-cn&type=date&theme=dark&legend=top-left" />
   <source media="(prefers-color-scheme: light)" srcset="https://api.star-history.com/svg?repos=tukuaiai/vibe-coding-cn&type=date&legend=top-left" />
   <img alt="Star History Chart" src="https://api.star-history.com/svg?repos=tukuaiai/vibe-coding-cn&type=date&legend=top-left" />
 </picture>
</a>

**Made with ❤️ and a lot of ☕ by [Nicolas Zullo](https://x.com/NicolasZu), [tukuaiai](https://github.com/tukuaiai) and [123olp](https://x.com/123olp)**

[⬆ 回到顶部](#vibe-coding-终极指南-v12)

</div>
