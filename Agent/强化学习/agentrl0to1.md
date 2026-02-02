# Agent RL 零基础入门教程

> 面向完全零基础的同学，从生活例子到代码实现

---

## 目录

1. [什么是智能体（Agent）？](#1-什么是智能体agent)
2. [什么是强化学习（RL）？](#2-什么是强化学习rl)
3. [核心概念详解](#3-核心概念详解)
4. [手把手看代码](#4-手把手看代码)
5. [实战案例](#5-实战案例)
6. [学习路线图](#6-学习路线图)

---

# 1. 什么是智能体（Agent）？

## 1.1 从生活例子理解

### 例子1：玩游戏的小狗

```
想象你正在训练一只小狗玩"找球"游戏：

┌────────────────────────────────────────┐
│                                        │
│   🐶 小狗                              │
│                                        │
│   你说："去找球！"                     │
│      ↓                                │
│   小狗跑向草丛 → 没找到（你不说啥）     │
│   小狗跑向树下 → 找到了（你给它零食🦴） │
│      ↓                                │
│   小狗学会了：去树下能找到球！         │
│                                        │
└────────────────────────────────────────┘

在这个例子中：
- 小狗 = 智能体（Agent）
- "找球" = 任务
- 草丛、树下 = 环境（Environment）
- 零食 = 奖励（Reward）
- 小狗学会了策略 = 训练
```

### 例子2：外卖骑手

```
🏃 外卖骑手也是个智能体：

状态：当前位置、目的地、天气、交通状况
      ↓
动作：选择哪条路线
      ↓
奖励：准时送达=好评+收入，迟到=差评-收入
      ↓
目标：学会选择最优路线，赚更多钱

骑手通过不断送餐（试错），学会了：
- 哪怕近路堵车，远一点的路可能更快
- 下雨天要骑慢一点
- 某些小区的门不好找
```

### 总结：智能体的特征

```
智能体 = 能感知环境 + 做决策 + 从经验中学习

┌──────────────────────────────────────┐
│                                      │
│   👀 感知  →  🧠 做决策  →  ✋ 行动   │
│      ↓            ↓           ↓      │
│   看到状态      思考怎么办     执行动作  │
│      ↓            ↓           ↓      │
│   ┌────────────────────────┐        │
│   │  环境（世界）          │──► 奖励/惩罚│
│   └────────────────────────┘        │
│            ↑                           │
│            └──────────────┘           │
│              从经验中学习              │
│                                      │
└──────────────────────────────────────┘
```

---

# 2. 什么是强化学习（RL）？

## 2.1 为什么要用强化学习？

### 传统训练方法的问题

```
问题： "1+1等于多少？"

❌ 死记硬背（监督学习）：
模型："2" → ✅ 对了
模型："3" → ❌ 错了
问题：只能背诵已知答案，遇到新题不会

✅ 强化学习：
模型尝试："1+1=1.5" → ❌ 错了，给-1分
模型尝试："1+1=2" → ✅ 对了，给+1分
模型尝试："1+1=1.8" → 接近了，给+0.5分
...
模型学会了：要往"2"这个方向调整
```

### 强化学习的本质

```
强化学习 = 通过试错 + 奖励反馈 来学习最优策略

┌────────────────────────────────────────┐
│                                        │
│   尝试行动 → 得到反馈 → 调整策略        │
│      ↓           ↓           ↓        │
│   (做选择)   (好/坏?)   (改进)         │
│                                        │
│   就像学骑自行车：                     │
│   1. 骑上去 → 摔倒 → 疼痛（惩罚）       │
│   2. 再试 → 平衡了一点 → 没摔（奖励）   │
│   3. 再试 → 骑得更稳 → 成功（大奖励）   │
│   4. 学会了骑自行车！                  │
│                                        │
└────────────────────────────────────────┘
```

---

# 3. 核心概念详解

## 3.1 MDP（马尔可夫决策过程）

这是强化学习的**数学框架**，但其实就是把"做决策"这件事形式化地描述出来。

### 用通俗的话理解MDP

```
想象你在玩一个闯关游戏：

┌────────────────────────────────────────┐
│                                        │
│   【状态 State】                       │
│   你在第1关，有3条命，100分           │
│                                        │
│   【动作 Action】                      │
│   你可以：跳跃、射击、蹲下             │
│                                        │
│   【转移 Transition】                  │
│   你选择"跳跃" → 到达平台              │
│   你选择"射击" → 打中敌人+100分        │
│                                        │
│   【奖励 Reward】                      │
│   跳到平台 = +10分                     │
│   被子弹打中 = -20分                   │
│   到达终点 = +100分                    │
│                                        │
│   【策略 Policy】                      │
│   遇到敌人时射击，遇到悬崖时跳跃       │
│   这是你学会的"玩法"                  │
│                                        │
└────────────────────────────────────────┘
```

### MDP的数学表示（简单理解）

```python
# 伪代码：MDP的循环

状态 = 当前情况
while 没有结束:
    # 根据当前状态，选择一个动作
    动作 = 策略(状态)

    # 执行动作，环境给出反馈
    新状态, 奖励 = 环境.执行(动作)

    # 从经验中学习
    策略.更新(状态, 动作, 奖励, 新状态)

    # 进入下一轮
    状态 = 新状态
```

## 3.2 策略（Policy）

**策略就是"怎么做决定的规则"**

```
生活中的策略：

🚗 开车策略：
   - 红灯 → 停
   - 绿灯 → 行
   - 前车减速 → 减速
   - 要转弯 → 打转向灯

🤖 AI的策略：
   在强化学习中，策略就是一个函数：
   输入：当前状态
   输出：要做什么动作

   策略(状态) = 动作

   例如：
   策略("前方有障碍物") = "绕行"
   策略("问题需要计算") = "使用代码工具"
```

### 代码中的策略

```python
# 在项目中，策略就是LLM模型本身

class Policy:
    def __init__(self, model):
        self.model = model

    def decide_action(self, state):
        """
        根据状态决定动作
        """
        # 把状态变成提示词
        prompt = f"当前状态：{state}\n请选择下一步动作："

        # 模型生成
        action = self.model.generate(prompt)

        return action

# 使用
policy = Policy(llm_model)
state = "这是一个数学题：123 × 456 = ?"
action = policy.decide_action(state)
# 可能输出："使用Python代码计算"
```

## 3.3 奖励（Reward）

**奖励就是告诉智能体"你做得怎么样"**

```
奖励设计的重要性：

❌ 不好奖励设计：
   只奖励"答案对了"
   问题：智能体可能学会"猜答案"

✅ 好奖励设计：
   - 答案对了 = +1分
   - 使用了工具 = +0.3分（鼓励工具使用）
   - 推理步骤清晰 = +0.2分
   - 搜索相关 = +0.2分
   - 搜索不相关 = -0.1分
```

### 代码中的奖励函数

```python
# 来自R1-Searcher项目

class RewardCalculator:
    def calculate_reward(self, question, answer, ground_truth):
        """
        计算奖励分数
        """
        reward = 0.0

        # 1. 答案正确性检查（最重要，70%）
        if self.answer_correct(answer, ground_truth):
            reward += 1.0
        else:
            reward += 0.0

        # 2. 格式检查（格式错会扣很多分）
        if not self.valid_format(answer):
            reward -= 2.0  # 严重惩罚！
            return reward  # 格式错了就不用看了

        # 3. 是否使用了搜索（鼓励探索）
        if self.has_search(answer):
            reward += 0.3

        return reward

    def answer_correct(self, pred, truth):
        """检查答案是否正确"""
        # 使用F1分数，不是严格匹配
        return f1_score(pred, truth) > 0.9

    def valid_format(self, answer):
        """检查格式"""
        has_thinking = "<thinking>" in answer
        has_search = "<search>" in answer
        has_answer = "<answer>" in answer
        return all([has_thinking, has_search, has_answer])
```

## 3.4 价值函数（Value Function）

**价值函数评估"这个状态有多好"**

```
生活中的价值判断：

🎮 游戏中的价值判断：
   "我有3条命，在关卡中间" → 价值高
   "我只有1条命，快死了" → 价值低
   "我快到终点了" → 价值很高！

🤖 AI中的价值函数：
   价值(状态) = 从这个状态开始，未来能得多少奖励

   例如：
   价值("找到了答案") = 1.0（非常好）
   价值("完全错误的方向") = -1.0（很糟）
   价值("刚开始思考") = 0.5（还可以）
```

## 3.5 优势函数（Advantage Function）

**优势函数回答"这个动作比平均好多少"**

```
通俗理解：

想象你参加考试：
- 你得了85分
- 班平均分是75分
- 那你的"优势"就是 85 - 75 = +10分

在RL中：
优势(动作) = 这个动作的奖励 - 平均奖励

如果优势 > 0：这个动作比平均好，多做！
如果优势 < 0：这个动作比平均差，少做！
```

### GRPO中的优势计算

```python
# 来自RAGEN项目

def compute_grpo_advantage(rewards, group_size):
    """
    计算GRPO优势

    GRPO的特点：用组内相对优势代替绝对优势
    """
    advantages = []

    for i in range(len(rewards)):
        # 找到同组的其他答案
        group_start = (i // group_size) * group_size
        group_end = group_start + group_size
        group_rewards = rewards[group_start:group_end]

        # 计算组内平均
        mean_reward = sum(group_rewards) / len(group_rewards)

        # 计算标准差
        import math
        std_reward = math.sqrt(
            sum((r - mean_reward) ** 2 for r in group_rewards) /
            len(group_rewards)
        )

        # 优势 = (当前奖励 - 平均) / 标准差
        advantage = (rewards[i] - mean_reward) / (std_reward + 1e-8)
        advantages.append(advantage)

    return advantages

# 例子
rewards = [0.8, 0.5, 0.3, 0.9]  # 4个答案
# 答案0比平均好，优势为正
# 答案2比平均差，优势为负
```

---

# 4. 手把手看代码

## 4.1 一个完整的训练循环

让我们从最基础的代码看起，逐步理解RL训练。

### 版本1：最简单的循环（伪代码）

```python
"""
这是一个超级简化的RL训练循环
帮助理解核心流程
"""

def train_agent_simple():
    # 1. 创建智能体（策略）
    agent = Agent(model="gpt-4")

    # 2. 训练循环
    for epoch in range(100):  # 训练100轮
        print(f"=== 第{epoch}轮训练 ===")

        # 3. 生成阶段：让智能体尝试
        for question in training_data:
            # 生成多个不同的答案（探索）
            answers = []
            for i in range(10):  # 每个问题生成10个答案
                answer = agent.generate(question)
                answers.append(answer)

            # 4. 评估阶段：给每个答案打分
            scores = []
            for answer in answers:
                score = calculate_reward(question, answer)
                scores.append(score)

            # 5. 学习阶段：从好答案中学习
            for answer, score in zip(answers, scores):
                if score > 0.5:  # 只从好答案学习
                    agent.learn(question, answer, score)

        # 6. 测试一下效果
        test_score = evaluate(agent, test_data)
        print(f"第{epoch}轮，测试得分：{test_score}")

def calculate_reward(question, answer):
    """计算奖励分数"""
    # 这里写你的奖励规则
    correct = check_answer(question, answer)
    if correct:
        return 1.0
    else:
        return 0.0
```

### 版本2：加入PPO算法

```python
"""
PPO（Proximal Policy Optimization）的核心思想
"""

def train_agent_ppo():
    agent = Agent(model="gpt-4")

    for epoch in range(100):
        # ========== 第一步：生成答案 ==========
        trajectories = []

        for question in training_data:
            for i in range(10):  # 10个不同的答案
                answer = agent.generate(question)

                # 记录轨迹
                trajectories.append({
                    'question': question,
                    'answer': answer,
                    'log_prob': agent.get_log_prob(question, answer)  # 记录概率
                })

        # ========== 第二步：计算奖励 ==========
        for traj in trajectories:
            traj['reward'] = calculate_reward(
                traj['question'],
                traj['answer']
            )

        # ========== 第三步：计算优势 ==========
        all_rewards = [t['reward'] for t in trajectories]

        for traj in trajectories:
            # 优势 = 这个答案的奖励 - 平均奖励
            traj['advantage'] = traj['reward'] - (sum(all_rewards) / len(all_rewards))

        # ========== 第四步：更新策略（PPO核心）==========
        for traj in trajectories:
            # 重新计算概率
            new_log_prob = agent.get_log_prob(
                traj['question'],
                traj['answer']
            )

            # PPO的关键：限制更新幅度
            # 概率比 = 新概率 / 旧概率
            ratio = math.exp(new_log_prob - traj['log_prob'])

            # 如果ratio在[0.8, 1.2]之间，就正常更新
            # 如果超出范围，就裁剪（不要更新太多）
            clipped_ratio = max(0.8, min(1.2, ratio))

            # PPO损失
            advantage = traj['advantage']

            # 两种选择，取较小的（保守）
            loss1 = ratio * advantage
            loss2 = clipped_ratio * advantage
            loss = -min(loss1, loss2)  # 负号是因为要最大化

            # 更新模型
            agent.update(loss)
```

### 版本3：真实项目中的训练循环

```python
"""
来自RAGEN项目的实际训练代码（简化版）
"""

class PPOTrainer:
    def __init__(self, actor_model, critic_model, config):
        self.actor = actor_model      # 策略模型：生成答案
        self.critic = critic_model    # 价值模型：评估状态
        self.config = config

    def train(self, train_data, val_data):
        """完整训练流程"""

        for epoch in range(self.config.total_epochs):
            print(f"=== Epoch {epoch} ===")

            # ====== 阶段1：Rollout（生成） ======
            print("生成轨迹...")
            trajectories = self._rollout_phase(train_data)

            # ====== 阶段2：计算奖励和优势 ======
            print("计算奖励...")
            self._compute_advantages(trajectories)

            # ====== 阶段3：更新模型 ======
            print("更新模型...")
            for batch in self._create_batches(trajectories):
                # 更新Actor
                self._update_actor(batch)

                # 更新Critic
                self._update_critic(batch)

            # ====== 阶段4：评估 ======
            if epoch % self.config.eval_every == 0:
                score = self.evaluate(val_data)
                print(f"评估得分：{score}")

    def _rollout_phase(self, data):
        """生成轨迹"""
        trajectories = []

        for item in data:
            question = item['question']

            # 生成多个答案
            for _ in range(self.config.num_rollouts):
                # Actor生成答案
                answer, log_prob = self.actor.generate(question)

                # Critic评估状态价值
                value = self.critic.evaluate(question)

                trajectories.append({
                    'question': question,
                    'answer': answer,
                    'log_prob': log_prob,
                    'value': value
                })

        return trajectories

    def _compute_advantages(self, trajectories):
        """计算优势（使用GAE）"""

        for traj in trajectories:
            # 1. 获取奖励
            reward = self._calculate_reward(traj)

            # 2. 计算TD误差
            # TD误差 = 奖励 + 折扣因子 × 下一状态价值 - 当前状态价值
            td_error = (
                reward +
                self.config.gamma * traj['value'] -
                traj['value']
            )

            # 3. 计算优势
            traj['advantage'] = td_error

    def _update_actor(self, batch):
        """更新Actor（策略模型）"""

        for traj in batch:
            # 重新计算log概率
            new_log_prob = self.actor.get_log_prob(
                traj['question'],
                traj['answer']
            )

            # 计算ratio
            ratio = math.exp(new_log_prob - traj['log_prob'])

            # PPO裁剪
            clipped_ratio = max(
                1.0 - self.config.clip_ratio,
                min(1.0 + self.config.clip_ratio, ratio)
            )

            # 计算损失
            advantage = traj['advantage']
            policy_loss = -min(
                ratio * advantage,
                clipped_ratio * advantage
            )

            # 更新
            self.actor.update(policy_loss)

    def _update_critic(self, batch):
        """更新Critic（价值模型）"""

        for traj in batch:
            # 目标价值 = 奖励 + 折扣因子 × 下一状态价值
            target_value = (
                self._calculate_reward(traj) +
                self.config.gamma * traj['value']
            )

            # MSE损失
            value_loss = (traj['value'] - target_value) ** 2

            # 更新
            self.critic.update(value_loss)
```

---

# 5. 实战案例

## 5.1 案例1：教AI学会使用计算器

让我们看一个完整的例子：教AI在做数学题时学会使用Python代码。

```python
"""
完整的Agent RL训练示例
目标：教AI在做数学题时使用Python代码
"""

# ==================== 第一步：定义环境 ====================

import re
import subprocess
import tempfile

class MathEnv:
    """
    数学题环境

    状态：数学题
    动作：写代码或不写代码
    奖励：答案对不对 + 是否使用了代码
    """

    def __init__(self, question):
        self.question = question
        self.max_steps = 5

    def step(self, action):
        """
        执行动作

        action格式：
        - 直接给答案："答案是：42"
        - 写代码：```python\nprint(123*456)\n```
        """

        # 1. 检查是否使用代码
        has_code = "```python" in action

        # 2. 如果有代码，执行它
        code_result = None
        if has_code:
            code = self._extract_code(action)
            code_result = self._execute_code(code)

        # 3. 提取答案
        answer = self._extract_answer(action, code_result)

        # 4. 计算奖励
        reward = self._calculate_reward(answer, has_code)

        # 5. 检查是否结束
        done = self._is_correct(answer)

        return {
            'answer': answer,
            'code_result': code_result,
            'reward': reward,
            'done': done
        }

    def _extract_code(self, text):
        """提取Python代码"""
        pattern = r'```python\n(.*?)```'
        match = re.search(pattern, text, re.DOTALL)
        if match:
            return match.group(1)
        return None

    def _execute_code(self, code):
        """执行Python代码"""
        try:
            # 创建临时文件
            with tempfile.NamedTemporaryFile(
                mode='w',
                suffix='.py',
                delete=False
            ) as f:
                f.write(code)
                temp_file = f.name

            # 执行
            result = subprocess.run(
                ['python', temp_file],
                capture_output=True,
                text=True,
                timeout=5
            )

            # 清理
            import os
            os.unlink(temp_file)

            return result.stdout.strip()

        except Exception as e:
            return f"Error: {str(e)}"

    def _extract_answer(self, action, code_result):
        """提取答案"""
        # 如果代码执行成功，用代码结果
        if code_result and not code_result.startswith("Error"):
            return code_result

        # 否则从文本中提取
        pattern = r'答案是[:：]\s*([0-9.]+)'
        match = re.search(pattern, action)
        if match:
            return match.group(1)

        return None

    def _calculate_reward(self, answer, has_code):
        """计算奖励"""
        reward = 0.0

        # 1. 答案正确性（70%）
        if self._is_correct(answer):
            reward += 0.7
        else:
            reward += 0.0

        # 2. 使用了代码（30%）
        if has_code:
            reward += 0.3

        return reward

    def _is_correct(self, answer):
        """检查答案是否正确"""
        # 这里简化处理
        # 实际中需要和标准答案比较
        correct_answer = self._get_correct_answer()
        return str(answer) == str(correct_answer)

    def _get_correct_answer(self):
        """获取正确答案"""
        # 从问题中提取（简化）
        if "123 × 456" in self.question:
            return 123 * 456
        # 可以添加更多规则
        return None


# ==================== 第二步：定义Agent ====================

class MathAgent:
    """
    数学智能体
    """

    def __init__(self, model):
        self.model = model
        self.temperature = 0.8  # 采样温度

    def generate(self, question):
        """生成答案"""
        # 构建提示
        prompt = f"""问题：{question}

你可以：
1. 直接给出答案
2. 使用Python代码计算

请回答："""

        # 生成
        response = self.model.generate(
            prompt,
            temperature=self.temperature
        )

        return response

    def update(self, question, answer, reward):
        """从经验中学习"""
        # 这里应该是实际的梯度更新
        # 简化版：记录到经验回放缓冲区
        print(f"学习：问题={question}, 答案={answer}, 奖励={reward}")


# ==================== 第三步：训练循环 ====================

def train_math_agent():
    """训练数学Agent"""

    # 初始化
    agent = MathAgent(model="gpt-4")
    questions = [
        "123 × 456 = ?",
        "789 × 234 = ?",
        "456 × 789 = ?",
        # ... 更多题目
    ]

    # 训练循环
    for epoch in range(10):
        print(f"\n=== Epoch {epoch} ===")

        total_reward = 0

        for question in questions:
            # 生成多个答案
            for _ in range(5):
                # Agent生成答案
                answer = agent.generate(question)

                # 环境执行
                env = MathEnv(question)
                result = env.step(answer)

                # 学习
                agent.update(
                    question,
                    answer,
                    result['reward']
                )

                total_reward += result['reward']

                if result['done']:
                    print(f"✅ 答对了！")
                    break

        print(f"总奖励：{total_reward}")


# 运行
if __name__ == "__main__":
    train_math_agent()
```

## 5.2 案例2：教AI学会搜索

```python
"""
教AI学会在做题时搜索答案
"""

import requests
import json

class SearchEnv:
    """
    搜索环境

    状态：问题
    动作：搜索或直接回答
    奖励：答案对不对 + 是否有效搜索
    """

    def __init__(self, question, search_api_key):
        self.question = question
        self.search_api_key = search_api_key
        self.search_history = []

    def step(self, action):
        """执行动作"""

        # 1. 检查是否搜索
        has_search = "<search>" in action

        # 2. 如果搜索，执行搜索
        search_result = None
        if has_search:
            query = self._extract_search_query(action)
            search_result = self._search(query)

        # 3. 提取答案
        answer = self._extract_answer(action, search_result)

        # 4. 计算奖励
        reward = self._calculate_reward(
            answer,
            has_search,
            search_result
        )

        return {
            'answer': answer,
            'search_result': search_result,
            'reward': reward
        }

    def _extract_search_query(self, action):
        """提取搜索查询"""
        pattern = r'<search>(.*?)</search>'
        match = re.search(pattern, action)
        if match:
            return match.group(1).strip()
        return None

    def _search(self, query):
        """执行搜索"""
        # 调用搜索API（例如Google Serper）
        url = "https://google.serper.dev/search"
        params = {
            "q": query,
            "key": self.search_api_key
        }

        response = requests.get(url, params=params)
        data = response.json()

        # 返回搜索结果
        return data.get('organic', [])[:5]

    def _extract_answer(self, action, search_result):
        """提取答案"""
        # 如果有搜索结果，基于搜索结果回答
        if search_result:
            # 这里简化，实际应该让LLM基于搜索结果生成
            pass

        # 提取<answer>标签
        pattern = r'<answer>(.*?)</answer>'
        match = re.search(pattern, action)
        if match:
            return match.group(1).strip()

        return None

    def _calculate_reward(self, answer, has_search, search_result):
        """计算奖励"""
        reward = 0.0

        # 1. 答案正确性（60%）
        if self._is_correct(answer):
            reward += 0.6

        # 2. 是否搜索（20%）
        if has_search:
            reward += 0.2

        # 3. 搜索质量（20%）
        if has_search and search_result:
            # 检查搜索结果是否相关
            relevance = self._check_search_relevance(
                self.question,
                search_result
            )
            reward += relevance * 0.2

        return reward

    def _is_correct(self, answer):
        """检查答案"""
        # 和标准答案比较
        correct_answer = self._get_correct_answer()
        return str(answer) == str(correct_answer)

    def _check_search_relevance(self, question, search_results):
        """检查搜索相关性"""
        # 简化：检查搜索结果是否包含问题关键词
        keywords = self._extract_keywords(question)

        relevant_count = 0
        for result in search_results:
            title = result.get('title', '')
            snippet = result.get('snippet', '')

            for keyword in keywords:
                if keyword.lower() in title.lower() or keyword.lower() in snippet.lower():
                    relevant_count += 1
                    break

        return relevant_count / len(search_results)

    def _extract_keywords(self, question):
        """提取关键词"""
        # 简化：分词
        import jieba
        return jieba.lcut(question)


# ==================== 训练 ====================

def train_search_agent():
    """训练搜索Agent"""

    agent = MathAgent(model="gpt-4")
    search_api_key = "your_api_key"

    questions = [
        "中国的首都是哪里？",
        "Python是谁创造的？",
        # ...
    ]

    for epoch in range(10):
        print(f"\n=== Epoch {epoch} ===")

        for question in questions:
            # 生成答案
            answer = agent.generate(question)

            # 环境
            env = SearchEnv(question, search_api_key)
            result = env.step(answer)

            # 学习
            agent.update(
                question,
                answer,
                result['reward']
            )


if __name__ == "__main__":
    train_search_agent()
```

---

# 6. 学习路线图

## 6.1 第一阶段：基础概念（1-2周）

```
Week 1: 理解Agent和RL
├── 阅读本文档，理解核心概念
├── 观看视频教程
└── 玩简单的RL游戏（如CartPole）

Week 2: Python基础
├── 学习Python语法
├── 学习PyTorch基础
└── 实现一个简单的RL循环
```

## 6.2 第二阶段：框架学习（2-3周）

```
Week 3-4: 学习项目代码
├── 从RAGEN开始（环境简单）
├── 理解MDP、策略、奖励
└── 运行示例代码

Week 5: veRL框架
├── 学习veRL文档
├── 理解PPO、GRPO算法
└── 运行完整训练
```

## 6.3 第三阶段：项目实践（3-4周）

```
Week 6-7: 简单项目
├── RAGEN（Sokoban环境）
├── ReTool（代码执行）
└── 理解奖励设计

Week 8-9: 复杂项目
├── R1-Searcher（搜索集成）
├── DeepResearcher（真实环境）
└── 尝试改进
```

## 6.4 第四阶段：深入研究（持续）

```
Week 10+: 论文和改进
├── 阅读原始论文
├── 复现实验结果
├── 设计新的奖励函数
└── 发表自己的研究
```

---

## 7. 常见问题解答

### Q1: Agent RL和普通ChatGPT有什么区别？

```
普通ChatGPT：
- 你问，它答
- 每次回答是独立的
- 不会从反馈中学习

Agent RL：
- 可以主动行动（搜索、计算）
- 可以多轮交互
- 会从奖励/惩罚中学习
- 目标是最大化累积奖励
```

### Q2: 为什么要生成多个答案？

```
原因1: 探索
如果只生成一个答案，可能运气好或不好
生成多个答案，可以看到"不同尝试"

原因2: 优势计算
GRPO需要多个答案来计算"相对优势"
答案A比其他答案好 → 正优势
答案B比其他答案差 → 负优势

例子：
问题: "1+1=?"
生成的10个答案：
[2, 2, 2, 2, 3, 2, 2, 1, 2, 2]
平均 ≈ 1.8

答案"1"的优势 = 1 - 1.8 = -0.8（差）
答案"3"的优势 = 3 - 1.8 = +1.2（好）
答案"2"的优势 ≈ 2 - 1.8 = +0.2（好）
```

### Q3: 什么是"Rollout"？

```
Rollout就是"让模型试着做一遍"

类比：
你在学骑自行车
Rollout就是"骑一次"的过程

在代码中：
def rollout(agent, question, max_steps=10):
    """执行一次完整的rollout"""

    history = []
    for step in range(max_steps):
        # 模型生成动作
        action = agent.generate(question, history)

        # 执行动作，得到反馈
        feedback = environment.step(action)

        # 记录
        history.append({
            'action': action,
            'feedback': feedback
        })

        # 检查是否结束
        if feedback['done']:
            break

    return history
```

### Q4: 什么是"Episode"（回合）？

```
Episode = 从开始到结束的一次完整经历

例子1: 打游戏
Episode = 从开始游戏到通关或失败

例子2: 数学题
Episode = 从看到题目到给出最终答案

例子3: 对话
Episode = 从用户提问到问题解决

在训练中：
for episode in range(1000):
    state = env.reset()

    while not done:
        action = agent.act(state)
        next_state, reward, done = env.step(action)
        agent.learn(state, action, reward, next_state)
        state = next_state
```

---

## 8. 核心代码详解

### 8.1 来自DeepResearcher的Handler

这是最真实的Agent RL代码，让我们仔细看看：

```python
"""
DeepResearcher的Handler代码详解
这个类负责协调搜索和阅读智能体
"""

class Handler:
    """
    Handler是整个系统的"大脑"

    职责：
    1. 接收模型的工具调用请求
    2. 执行搜索
    3. 阅读网页
    4. 返回结果给模型
    """

    def __init__(self, agent_config, client):
        # 两个智能体
        self.web_search_agent = WebSearchAgent(config, client)
        self.reading_agent = ReadingAgent(config, client)

        # 缓存
        self.api_result_dict = {}  # 搜索结果缓存
        self.id_to_context = {}      # 对话上下文

    def handle_execution_api(self, query_contents):
        """
        主处理函数

        query_contents示例：
        [
            {
                "idx": 0,
                "question": "量子计算机最新进展？",
                "tool_call": {
                    "name": "web_search",
                    "arguments": {
                        "query": ["quantum computing 2024"]
                    }
                }
            }
        ]
        """

        # ====== 第一阶段：搜索 ======
        print("开始处理搜索请求...")

        # 检查缓存（7天有效期）
        cache_hit = 0
        total_search = 0

        for query_content in query_contents:
            if query_content['tool_call']['name'] != 'web_search':
                continue

            query_list = query_content['tool_call']['arguments']['query']
            total_search += len(query_list)

            for query in query_list:
                # 检查缓存
                if query in self.api_result_dict:
                    timestamp = self.api_result_dict[query]['timestamp']
                    # 7天内有效
                    if time.time() - timestamp <= 7 * 24 * 3600:
                        cache_hit += 1
                        continue

                # 需要搜索
                self.api_result_dict[query] = {
                    "timestamp": time.time(),
                    "organic": []  # 稍后填充
                }

        print(f"缓存命中率：{cache_hit}/{total_search}")

        # 并发执行搜索（1000个worker！）
        with ThreadPoolExecutor(max_workers=1000) as executor:
            futures = []
            for query in self.api_result_dict:
                if self.api_result_dict[query]['organic']:
                    continue  # 已有缓存

                future = executor.submit(
                    self._search_and_add_to_dict,
                    query,
                    self.api_result_dict
                )
                futures.append(future)

            # 等待所有搜索完成
            for future in futures:
                future.result()

        # 保存缓存
        with open(self.query_save_path, 'w') as f:
            json.dump(self.api_result_dict, f)

        # ====== 第二阶段：处理每个查询 ======
        print("开始处理查询结果...")

        with ThreadPoolExecutor(max_workers=100) as executor:
            futures = []
            for query_content in query_contents:
                future = executor.submit(
                    self.handle_single_query,
                    query_content,
                    self.api_result_dict
                )
                futures.append(future)

            # 收集结果
            for i, future in enumerate(futures):
                query_contents[i]["content"] = future.result()

        print("处理完成")
        return query_contents

    def handle_single_query(self, query_content, api_result_dict):
        """
        处理单个查询
        """
        idx = query_content["idx"]
        question = query_content["question"]
        tool_call = query_content["tool_call"]

        func_name = tool_call["name"]
        arguments = tool_call["arguments"]

        if func_name == "web_search":
            # ====== 执行搜索 ======
            search_query_list = arguments["query"]

            # 批量搜索
            web_page_info_list_batch = self.web_search_agent.search_web_batch(
                user_query=question,
                search_query_list=search_query_list,
                api_result_dict=api_result_dict
            )

            # 格式化结果
            content = []
            for search_query, web_page_info_list in zip(
                search_query_list,
                web_page_info_list_batch
            ):
                ret_list = []
                for web_page_info in web_page_info_list:
                    ret_list.append({
                        "title": web_page_info.title,
                        "url": web_page_info.url,
                        "quick_summary": web_page_info.quick_summary
                    })

                content.append({
                    "search_query": search_query,
                    "web_page_info_list": ret_list
                })

            return content

        elif func_name == "browse_webpage":
            # ====== 阅读网页 ======
            url_list = arguments["url_list"]

            # 批量阅读
            read_webpage_list = self.reading_agent.read_batch(
                user_query=question,
                search_result_info_list=self.id_to_context[idx][-1].search_result_info_list,
                url_list=url_list,
                web_search_agent=self.web_search_agent
            )

            # 格式化结果
            content = []
            for read_webpage in read_webpage_list:
                information = []
                for page_read_info in read_webpage.page_read_info_list:
                    information.append({
                        "page_number": page_read_info.page_number,
                        "page_summary": page_read_info.page_summary
                    })

                content.append({
                    "url": read_webpage.url,
                    "information": information
                })

            return content
```

### 8.2 来自R1-Searcher的奖励计算

```python
"""
奖励函数设计详解

这是R1-Searcher的核心：如何给答案打分
"""

class RewardCalculator:
    """
    奖励计算器

    核心思想：用奖励引导模型学会搜索
    """

    def get_reward(self, queries):
        """
        计算奖励

        queries: 模型生成的多个答案
        返回: 每个答案的奖励分数
        """
        scores = []

        for i, query in enumerate(queries):
            # ====== 提取问题和答案 ======
            question, solution = self.parse_query(query)
            pred_answer = self.extract_answer(solution)
            true_answer = self.get_ground_truth(question)

            # ====== 计算F1分数 ======
            f1_score = self.calculate_f1(pred_answer, true_answer)
            scores.append(float(f1_score))

            # ====== 检查格式 ======
            # 格式检查：必须有特定标签
            if not self.has_valid_format(solution):
                scores[i] = 0.0  # 格式错误，0分！
                continue

            # ====== 检查标签配对 ======
            if not self.check_tag_pairs(solution):
                scores[i] -= 2.0  # 标签不配对，重罚！
                continue

            # ====== 检查是否完成 ======
            if self.is_complete(solution):
                # 完成且有正确格式，奖励更高
                scores[i] += 0.5

        return scores

    def calculate_f1(self, pred, truth):
        """
        计算F1分数

        F1 = 2 × (精确率 × 召回率) / (精确率 + 召回率)
        """
        # 标准化
        pred = self.normalize(pred)
        truth = self.normalize(truth)

        # 分词
        pred_tokens = pred.split()
        truth_tokens = truth.split()

        # 计算共同token数
        common = set(pred_tokens) & set(truth_tokens)
        num_common = len(common)

        if num_common == 0:
            return 0.0

        # 精确率 = 共同/预测
        precision = num_common / len(pred_tokens)

        # 召回率 = 共同/真实
        recall = num_common / len(truth_tokens)

        # F1分数
        f1 = (2 * precision * recall) / (precision + recall)

        return f1

    def has_valid_format(self, solution):
        """检查格式"""
        required_tags = [
            "<thinking>",
            "<search>",
            "<documents>",
            "<answer>"
        ]

        for tag in required_tags:
            if tag not in solution:
                return False

        return True

    def check_tag_pairs(self, solution):
        """检查标签配对"""
        tags = ["<search>", "<documents>", "<answer>"]

        for tag in tags:
            open_tag = tag
            close_tag = tag.replace("<", "</").replace(">", ">")

            count_open = solution.count(open_tag)
            count_close = solution.count(close_tag)

            if count_open != count_close:
                return False  # 不配对！

        return True
```

---

## 总结

### Agent RL的核心流程

```
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│   1. 观察        2. 决策       3. 行动       4. 反馈        │
│   状态 ────► 策略 ────► 动作 ────► 奖励               │
│      ↑                                          │            │
│      │                                          ↓            │
│      └────────────── 更新策略 ←────────────────────────┘   │
│                                                             │
│   就像学习骑自行车：                                     │
│   - 观察：路况、自行车状态                                │
│   - 决策：要不要加速、转向                                 │
│   - 行动：转动车把、踩踏板                                 │
│   - 反馈：骑稳了还是摔了                                 │
│   - 更新：调整骑车策略                                     │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 学习建议

1. **从简单开始**
   - 先理解概念
   - 再看代码
   - 最后动手实践

2. **循序渐进**
   - 从RAGEN开始（最简单）
   - 逐步学习其他项目
   - 不要急于求成

3. **多做实验**
   - 修改参数
   - 观察效果
   - 理解原因

4. **善用资源**
   - 项目README
   - 论文
   - 社区讨论

祝你学习顺利！🚀
