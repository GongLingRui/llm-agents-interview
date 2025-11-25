### Gemini 刷屏背后的秘密：如何让 AI 直接生成界面？3000词 Prompt 全景拆解
**Google Research 《Generative UI: LLMs are Effective UI Generators》论文解读**

- **核心突破**：从生成“静态文本”到生成“交互体验”
- **关键技术**：3000词系统指令 | 动态工具调用 | 自动化后处理
- **结果**：83% 用户偏好度 (vs Markdown)

### 01. 从“文字墙”到“即时应用”

| 维度   | 传统 LLM 交互                  | Generative UI (本文)                    |
|--------|--------------------------------|-----------------------------------------|
| 形式   | Markdown Wall of Text (静态)   | Interactive App (动态交互)              |
| 体验   | 枯燥阅读，被动接收             | 点击、滑动、模拟、探索                  |
| 本质   | 预定义的硬编码界面             | 即时组建的 AI 团队 (PM + Designer + Dev) |


*“Generative UI enables us to spin up an instant AI-based product team for a specific prompt.”*

<img width="1067" height="717" alt="image" src="https://github.com/user-attachments/assets/5659d2d7-4915-4958-8b52-e6b8fb9a1d2e" />

### 核心：System Instructions 的五大支柱

System Prompt 被严格划分为以下 5 个部分：

1. **Core Philosophy (核心哲学)**：定义“交互优先”的价值观

2. **Examples (少样本示例)**：通过具体案例对齐预期

3. **Planning Instructions (规划指令)**：强制思维链 (CoT) 步骤

4. **Technical Details (技术规范)**：HTML/Tailwind/API 使用手册（占比最大）

5. **Dynamic Context (动态信息)**：注入当前时间、用户位置

### Part 1: Core Philosophy (8 大核心原则)

#### A. 产品思维 (Product Mindset)
1. **Interactive First**
   拒绝静态文本，必须生成 App (Widgets/Tools)。
2. **No Walls of Text**
   视觉优先，避免长篇大论，多用 Card/Icons。
3. **Quality & Depth**
   追求高质量设计，不仅要能用，还要好用。
4. **Freshness**
   确保数据时效性，过期信息 = 0 价值。

#### B. 工程底线 (Engineering Standards)
5. **No Placeholders (严禁占位)**
   Absolutely Forbidden. 要么做真，要么砍掉。
6. **Fact Verification (强制验证)**
   实体/数据必须联网搜索，拒绝 Hallucination。
7. **Implement Fully**
   代码必须鲁棒 (Robust)，完整实现 JS 逻辑。
8. **Handle Data Creatively**
   先 Fetch 数据再 Design，拒绝虚构数据填坑。


### Part 2A: 示例教学 - 基础交互与数据可视化
Google 通过 Examples 重新定义了“简单查询”的交付标准：

- **Case 1: 简单查询**
  查询："What's the time?"
  文本形式（×）："It is 10 PM."
  App 形式（√）：JS 动态时钟组件
  经验：即使是微小数据，也要**组件化**。

- **Case 2: 复杂任务**
  查询："Jogging in Singapore"
  文本形式（×）：列出地点名称
  App 形式（√）：Search + Maps API + Polyline
  经验：**工具链组合**能力。

- **Case 3: 知识检索**
  查询："Barack Obama Family"
  文本形式（×）：枯燥的人物列表
  App 形式（√）：交互式时间轴 / 家谱树
  经验：数据的**可视化**。


  <img width="850" height="390" alt="image" src="https://github.com/user-attachments/assets/c44c0dee-326a-4d22-966d-1ff68a785e4e" />



<img width="1337" height="588" alt="image" src="https://github.com/user-attachments/assets/0cab3351-fce4-41fd-a773-fe3d0f0da4d9" />


### Part 2B: 示例教学 - 仿真、落地与一致性
针对高级场景，Prompt 重点训练了模型的仿真能力与防幻觉机制：

- **Case 4: 抽象概念**
  查询："Ant Colony" (蚁群)
  文本形式（×）：蚂蚁习性科普文
  App 形式（√）：Canvas 2D 仿真模拟器
  经验：**Simulation (仿真)**。

- **Case 5: 具体人物**
  查询："Yaniv Leviathan"
  风险：模型幻觉
  行动（√）：Mandatory Search + 事实引用
  经验：**Grounding (事实落地)**。

- **Case 6: 创意生成**
  查询："Graphic Novel"
  风险：角色长相不一致
  行动（√）：Prompt Engineering (固定描述)
  经验：**Consistency (一致性)**。


  ### Part 3: Planning Instructions (规划指令)
在编写任何 HTML 代码前，模型必须经历 **7步思维链 (Internal Thought Process)**：
1. **Interpret Query**：分析意图，确定App类型。
2. **Plan Application**：设计核心交互逻辑。
3. **Plan Content**：规划文案、故事线 (Internal only)。
4. **Identify Data Needs (关键一步)**：
   - 哪些数据需要 Search (事实/实体)？
   - 哪些图片需要 Generate (抽象概念)？
5. **Perform Searches**：执行工具调用，获取真实信息。
6. **Brainstorm Features**：列出功能清单。
7. **Filter & Integrate**：整合最终方案。

### Part 4A: Technical Details - 代码规范与沙盒

#### 输出格式 (Output Format)
- **Strict Wrapping**：
  代码必须包裹在 `'''html ... '''` 之间。
  **CRITICAL**：首尾严禁包含 markdown 或“废话”。
- **Full Document**：
  必须生成完整的 `<!DOCTYPE html>` 页面，包含 `head`。
- **Tailwind Only**：
  通过 Play CDN 引入 Tailwind CSS。
  `<script src="...tailwindcss.com">`

#### JS 限制与沙盒 (Restrictions)
- **Self-Contained**：
  JS 必须完全闭环。**FORBIDDEN** 访问 `window.parent` 或 `window.top`。
- **No Storage**：
  **Do NOT use** `localStorage` 或 `sessionStorage`。
- **Robustness**：
  关键逻辑必须包裹在 `try...catch` 中。
  使用 `DOMContentLoaded` 确保 DOM 就绪。


  ### Part 4B: Technical Details - 图像路由策略 (Image Strategy)
这是 Prompt 中最精细的逻辑：模型必须在 **Generate** 和 **Search** 之间做决策。

- **铁律**：使用标准 `<img src="...">`，严禁使用 `<div>` 占位符或 JS 加载。

#### Route 1: Generate (/gen)
适用场景：抽象概念、通用插图、无需真实的场景。
`src="/gen?prompt={PROMPT}&aspect={RATIO}"`
- **Prompt**：必须 URL-Encoded。描述要包含风格 (e.g. "cartoon style")。
- **Consistency**：多图生成同一角色时，必须**硬编码**重复角色的视觉描述。

#### Route 2: Search (/image)
适用场景：具体名人、特定地标、真实产品。
`src="/image?query={QUERY}"`
- **Logic**：如果用户问 "Obama"，绝对不能生成假照片，必须搜索真实图片。
- **Constraint**：所有的图片返回都是缩略图 (Thumbnails)，设计时需考虑清晰度。

### Part 4C: Technical Details - 音频策略 (Audio Strategy)
不仅要好看，还要好听：原生 TTS 与 Tone.js 的结合。

#### 1. Speech (语音合成)
适用场景：语言教学、朗读助手、无障碍交互。
`window.speechSynthesis.speak(...)`
- **Native API**：直接调用浏览器原生 API，无需外部依赖。
- **Usage**：当用户要求“教我读”或“朗读这段话”时触发。

#### 2. Music & SFX (音乐与音效)
适用场景：游戏背景乐、交互反馈音效。
`<script src="...Tone.js"></script>`
- **Tone.js**：必须引入 Tone.js 库来生成合成音。
- **Logic**：思考旋律和乐器 (Instruments)，用代码编写乐谱，而非加载 MP3。


### 03. 最后的防线：9 大自动化后处理机制
Appendix A.6 详细列出了保障代码可运行的 9 层过滤网：

#### 1. 基础设施层 (Infrastructure)
- **API Key Injection**：自动注入真实 Maps API Key
- **API Logic Fix**：修正 API 调用参数错误
- **Error Reporting**：注入前端错误监控脚本

#### 2. 代码完整性 (Code Integrity)
- **JS Syntax Fix**：自动修复语法解析错误
- **HTML Escaping**：转义属性中的特殊字符
- **Citation Removal**：移除错误的引用标记

#### 3. 样式与素材 (Styling & Assets)
- **Tailwind Fix**：补全缺失的 CSS 指令
- **Circular Dep.**：解除 CSS 循环依赖
- **Asset Hallucination**：检测并替换幻觉图标

### 04. 效果验证与未来

#### 用户偏好 (User Preference)
- **Generative UI vs Markdown**：**83% 胜率**
- **Generative UI vs Human Experts**：**44% 战平**

#### PAGEN Dataset
Google 在 Upwork 雇佣专业开发者建立了首个 "Gold Standard" 数据集。

#### Takeaway
未来的 UI 将是 **Ephemeral (转瞬即逝)** 的：按需生成，用完即走。














