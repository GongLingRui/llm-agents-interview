<img width="473" height="288" alt="image" src="https://github.com/user-attachments/assets/68817fd7-496b-41f6-bf8b-bd877e9452f4" />

<img width="641" height="450" alt="image" src="https://github.com/user-attachments/assets/c6aa9d9b-151a-471c-baaf-b6f261d98b6e" />

https://www.bilibili.com/video/BV1iSxVz2Epq?spm_id_from=333.788.videopod.sections&vd_source=c9ff634ae673bbd311926a353b26d3b5

<img width="679" height="461" alt="image" src="https://github.com/user-attachments/assets/5ea0adee-7701-4642-8126-f5b1d9e16fa2" />

https://www.bilibili.com/video/BV1gJxVzMEfW?spm_id_from=333.788.player.switch&vd_source=c9ff634ae673bbd311926a353b26d3b5&trackid=web_related_0.router-related-2206419-gx8f2.1760868125993.843

<img width="586" height="428" alt="image" src="https://github.com/user-attachments/assets/bee3338a-8c95-4f65-82ef-408f26372f78" />


<img width="654" height="408" alt="image" src="https://github.com/user-attachments/assets/bd08c930-3753-4577-b2da-1b77da5460c9" />

<img width="720" height="437" alt="image" src="https://github.com/user-attachments/assets/67ac0e7b-b987-4ace-8ee5-52894adf135c" />

https://www.bilibili.com/video/BV1Q7xVzBEbf?spm_id_from=333.788.player.switch&vd_source=c9ff634ae673bbd311926a353b26d3b5&trackid=web_related_0.router-related-2206419-2xpqp.1760868401530.567

<img width="594" height="453" alt="image" src="https://github.com/user-attachments/assets/013d2c96-3e4e-4753-9a41-cab152b7e249" />


<img width="607" height="423" alt="image" src="https://github.com/user-attachments/assets/f13bd39f-1e65-4e3e-9b7e-7f8d4303221d" />

| 类别           | 核心类比                     | 机制                                                         | 作用                                                         | 适用                     |
|----------------|------------------------------|--------------------------------------------------------------|--------------------------------------------------------------|--------------------------|
| 压缩(Compaction) | “AI为自己写会议纪要”         | 当上下文接近饱和时，通过“元调用”让模型自身对历史进行高保真总结，形成摘要。 | 管理“工作记忆”，在不丢失核心信息的前提下，为新信息腾出空间。 | 高连贯性的对话流任务     |
| 结构化笔记     | “为AI配备外部大脑”           | 赋予Agent读写外部文件或数据库的工具，使其主动记录关键决策与状态。| 构建“持久化记忆”，实现跨会话、跨周期的知识积累和状态恢复。| 有明确里程碑的迭代式任务 |
| Sub-agent架构  | “AI组建自己的专家团队”       | 由“主Agent”规划分解，将子任务分派给多个并行的“专家子Agent”处理。 | 实现“关注点分离”，通过分而治之，解决单一Agent无法处理的规模化复杂性。 | 可并行探索的复杂研究分析 |


<img width="489" height="406" alt="image" src="https://github.com/user-attachments/assets/f5b7a429-c8a2-4a35-b0be-15d51592603e" />



https://www.bilibili.com/video/BV1dbxvz6E4N?spm_id_from=333.788.player.switch&vd_source=c9ff634ae673bbd311926a353b26d3b5&trackid=web_related_0.router-related-2206419-45xsh.1760868731890.552


<img width="635" height="436" alt="image" src="https://github.com/user-attachments/assets/bd8b9ca4-de4d-422a-86c8-2c4324405423" />


<img width="707" height="448" alt="image" src="https://github.com/user-attachments/assets/93ace7c6-e2b6-4768-a755-f04bcadf7e8a" />

https://www.bilibili.com/video/BV1itxvz3ES6?spm_id_from=333.788.player.switch&vd_source=c9ff634ae673bbd311926a353b26d3b5&trackid=web_related_0.router-related-2206419-xc9bj.1760869549638.254

<img width="740" height="451" alt="image" src="https://github.com/user-attachments/assets/7683b4db-d37c-415b-83bf-ee37ad3b30dd" />

<img width="720" height="497" alt="image" src="https://github.com/user-attachments/assets/bc443eda-6e52-476e-8819-711ef48d3526" />


https://www.bilibili.com/video/BV16qxCzTEww?spm_id_from=333.788.player.switch&vd_source=c9ff634ae673bbd311926a353b26d3b5&trackid=web_related_0.router-related-2206419-k4qpm.1760870147378.831

<img width="670" height="461" alt="image" src="https://github.com/user-attachments/assets/d6901a5e-d04c-4854-8e0f-a927d4e8b8b7" />

<img width="691" height="464" alt="image" src="https://github.com/user-attachments/assets/d13cce05-f594-496e-b54e-2adee412918e" />


<img width="726" height="438" alt="image" src="https://github.com/user-attachments/assets/ae5190aa-9aa7-41ee-9333-b8920e63e7da" />

<img width="674" height="412" alt="image" src="https://github.com/user-attachments/assets/b9656475-8b87-4e4a-abce-61d565aca326" />
# 长期记忆系统：知识的沉淀与进化
## 核心挑战
如何从海量观察中，智能地提炼高价值信息？

## 机制一：Reflexion（反思）
- 核心逻辑：从**错误**中学习，提炼规则（事件驱动）。
- 步骤：
  1. **失败检测（由框架完成）**：
     - 行动：`search("Apple revenue")`
     - 观察：返回了手机新闻，而非财报。
  2. **生成元提示（由框架完成）**：
     向LLM提问："回顾这次失败，生成一条可复用的规则来避免未来犯错。"
  3. **生成并写入记忆**：
     产出（行为策略）："当我搜索公司财报时，应加入‘季度收益’等关键词。"

## 机制二：Generative Agents
- 核心逻辑：从观察中总结，合成洞察（周期性）。
- 步骤：
  1. **原始日志收集**：
     - 10:05: 张三提到“预算紧张”。
     - 14:30: 收到李四关于“成本削减”的邮件。
  2. **启动合成流程（Prompt链）**：
     ① 提问："总结以上日志的核心动态。"
     → 回答："张三和李四都讨论了预算问题。"
     ② 提问："基于此总结，有何新洞察？"
  3. **生成并写入记忆**：
     产出（高级事实）："洞察：项目X正面临显著的预算风险。"
https://www.bilibili.com/video/BV1Zy47z8EKu?spm_id_from=333.788.player.switch&vd_source=c9ff634ae673bbd311926a353b26d3b5&trackid=web_related_0.router-related-2206419-7v86w.1760870690891.342

  <img width="665" height="486" alt="image" src="https://github.com/user-attachments/assets/7d43c20a-fc04-46a5-b3b1-7c2813df33cb" />


  <img width="668" height="472" alt="image" src="https://github.com/user-attachments/assets/567c6dc1-b545-4d56-a70c-a470f7a9b811" />

  <img width="588" height="418" alt="image" src="https://github.com/user-attachments/assets/6ecf491f-4a46-46e4-bea9-6be96f081892" />


  https://www.bilibili.com/video/BV1dH4xz7E1u?spm_id_from=333.788.player.switch&vd_source=c9ff634ae673bbd311926a353b26d3b5

  <img width="683" height="476" alt="image" src="https://github.com/user-attachments/assets/f12672a6-567b-40c0-870f-0dab2663985c" />




<img width="654" height="472" alt="image" src="https://github.com/user-attachments/assets/d9573430-de7d-41f8-8940-2eddf97eaa78" />


<img width="643" height="443" alt="image" src="https://github.com/user-attachments/assets/9e5c6feb-b5fc-427f-bc04-83ab6938261b" />

  

<img width="700" height="477" alt="image" src="https://github.com/user-attachments/assets/09b533b6-02d3-4715-8183-30a9e68f6332" />

https://www.bilibili.com/video/BV15o4gzuEs9?spm_id_from=333.788.player.switch&vd_source=c9ff634ae673bbd311926a353b26d3b5&trackid=web_related_0.router-related-2206419-2xpqp.1760878649051.50

<img width="624" height="471" alt="image" src="https://github.com/user-attachments/assets/aab29665-0876-4064-af97-0d7fab8093ad" />


<img width="706" height="469" alt="image" src="https://github.com/user-attachments/assets/dfe59b71-a5e6-41a3-9120-1f37a8d7e94e" />

<img width="678" height="467" alt="image" src="https://github.com/user-attachments/assets/57167a44-c7f5-4200-a596-bcedb5064776" />


<img width="595" height="448" alt="image" src="https://github.com/user-attachments/assets/3a34799a-539f-43b5-8c15-11860f6c0e0c" />

https://www.bilibili.com/video/BV1yc42zpEnq?spm_id_from=333.788.player.switch&vd_source=c9ff634ae673bbd311926a353b26d3b5&trackid=web_related_0.router-related-2206419-skq5q.1760879560939.934


<img width="665" height="471" alt="image" src="https://github.com/user-attachments/assets/3fbd70e9-1965-4bd7-b17d-ba80bc3fb818" />

