https://huggingface.co/blog/martinigoyanes/llm-inference-at-scale-with-tgi

解码阶段体现了llm的自回归特性，模型会诸葛生成文本token来预填充的初始token为基础进行构建，每个新生成的token都会被添加到输入的序列，为莫相互处理创建新的上下文

eos令牌或者nax_new_tokens指定的最大的序列长度，生成的序列会在cpu上进行解令牌话处理，将令牌转换成为可读的文本

<img width="1096" height="853" alt="image" src="https://github.com/user-attachments/assets/fa55114d-b191-4ca5-8d8d-eefa61d577d9" />


预填充阶段只需要进行一次前向传播，解码阶段涉及多次传播，每次传播都依赖之前生成的令牌，解码阶段这种自回归的特性导致处理时间更长，计算成本随着总序列长度增加呈现二次方的增长

模型部署两个主要的组件：the engine and the server

engine:处理和模型相关的所有的事物和请求批处理

服务器则专注于转发用户请求

<img width="1116" height="537" alt="image" src="https://github.com/user-attachments/assets/954595c1-97a6-451c-b08c-5e7d35824da4" />


# 路由器：排队和连续批处理The router:queueing and continuous batching

The primary purpose of TGI router is to manage incoming requests and prevent the engine from encountering memory-related issues and ensuring smooth and efficient LLM inference. It employs a smart continuous batching algorithm, dynamically adding requests to the running batch to optimize performance. This dynamic batching approach strikes a balance between latency and throughput.

TGI 路由器的主要目的是管理传入请求，防止引擎遇到与内存相关的问题，并确保大型语言模型（LLM）推理过程流畅高效。它采用了一种智能连续批处理算法，动态地将请求添加到运行中的批处理中，以优化性能。这种动态批处理方法在延迟和吞吐量之间取得了平衡。

初始化的时候，路由器会触发推理引擎的预热阶段，路由器会确定所部署的大预言模型的底层杨戬的最大的容量


Upon initialization, the router triggers a warm-up phase on the inference engine. We’ll cover that on the next section, but basically during this phase, the router determines the maximum capacity of the underlying hardware (GPU) for the deployed LLM:
初始化时，路由器会触发推理引擎的预热阶段。我们将在下一节中介绍这一点，但基本上在这个阶段，路由器会确定所部署的大语言模型的底层硬件（GPU）的最大容量：

max_batch_prefill_tokens:Gpu在预填充阶段单次前向传播当中可以处理的最大令牌数量

max_batch_total_tokens；在预填充和解码阶段可以同时处理的最大令牌书

```
# Initialize the batch and token budgets
batch = []
token_budget = max_batch_total_tokens

# Function to add requests to the prefill batch until the max_tokens budget is reached
def add_requests_to_prefill_batch(requests, batch, max_tokens):
    while requests and sum(request.tokens for request in batch) < max_tokens:
        batch.append(requests.pop(0))
    return batch

# Add initial requests to the prefill batch
batch = add_requests_to_prefill_batch(request_queue, batch, max_batch_prefill_tokens)

# Prefill the batch
prefill(batch)

# Main loop to manage requests
while batch:
    # Update the token budget based on current batch
    batch_max_tokens = sum(request.input_tokens + request.max_new_tokens for request in batch)
    token_budget = max_batch_total_tokens - batch_max_tokens
    
    # Add new requests to the batch based on token budgets
    new_batch = add_requests_to_batch(request_queue, [], min(max_batch_prefill_tokens, token_budget))
    
    # If new requests were added, handle prefill and decoding
    if new_batch:
        # Stop decoding and prefill the new batch
        prefill(new_batch)
        
        # Extend the original batch with the new requests
        batch.extend(new_batch)
    
    # Decode the current batch
    decode(batch)
    
    # Filter out completed requests that have reached EOS or max_new_tokens
    batch = [request for request in batch if not request.reached_EOS and request.tokens_generated < request.max_new_tokens]
    
    # Update token budget by subtracting tokens from completed requests
    completed_requests = [request for request in batch if request.reached_EOS or request.tokens_generated >= request.max_new_tokens]
    for request in completed_requests:
        token_budget = token_budget - request.input_tokens + request.tokens_generated
```
这段代码是 **TGI（Text Generation Inference）持续批处理（Continuous Batching）核心逻辑的简化实现**，核心目标是：在不超过 GPU 显存预算（`max_batch_total_tokens`）的前提下，动态将新请求加入运行中的推理批次，平衡延迟与吞吐量（避免静态批处理“等待前一批结束才能加新请求”的低效问题）。

下面从 **核心概念、代码逐段拆解、逻辑流程、关键设计亮点** 四个维度详细讲解：


## 一、先明确核心概念（理解代码的前提）
在看代码前，需先对应 TGI 中的关键参数和术语：
| 变量/函数               | 含义（对应 TGI 核心逻辑）                                                                 |
|-------------------------|------------------------------------------------------------------------------------------|
| `max_batch_total_tokens`| 全局显存预算上限：所有并发请求（Prefill + Decode 阶段）的总 Token 数不能超过该值（避免 OOM）。 |
| `max_batch_prefill_tokens` | Prefill 阶段单次前向传播的最大 Token 数：CPU 分词后传给 GPU 的初始上下文总长度上限。       |
| `request_queue`         | 待处理的用户请求队列（每个请求包含 `input_tokens` 输入长度、`max_new_tokens` 最大输出长度等属性）。 |
| `Prefill()`             | 预填充阶段：GPU 对输入上下文做单次前向传播，生成初始 Token 和 KV 缓存（对应前文 LLM 推理的 Prefill 阶段）。 |
| `Decode()`              | 解码阶段：GPU 基于 KV 缓存逐一生成新 Token（自回归迭代），直到触发 EOS 或达到 `max_new_tokens`。 |
| `request.tokens_generated` | 某请求已生成的输出 Token 数（初始为 0，每 Decode 一次 +1）。                               |
| `request.reached_EOS`   | 标记请求是否生成了 EOS（序列结束）Token（生成则任务完成）。                                |


## 二、代码逐段拆解（带执行逻辑）
### 1. 初始化：批次与 Token 预算
```python
# Initialize the batch and token budgets
batch = []  # 当前正在处理的“活跃批次”（包含多个并发请求）
token_budget = max_batch_total_tokens  # 初始 Token 预算 = 全局最大总 Token 数（未用任何显存）
```
- 作用：创建空的活跃批次（`batch`），并初始化显存预算（所有显存都可用）。
- 注意：`token_budget` 是“剩余可用的 Token 数”，后续会随请求加入/完成动态更新。


### 2. 核心工具函数：将请求加入 Prefill 批次
```python
# Function to add requests to the prefill batch until the max_tokens budget is reached
def add_requests_to_prefill_batch(requests, batch, max_tokens):
    # 循环条件：1. 待处理队列非空；2. 当前批次的总 Token 数 < 预算（max_tokens）
    while requests and sum(request.tokens for request in batch) < max_tokens:
        # 从队列头部取出一个请求，加入当前批次（FIFO 顺序）
        batch.append(requests.pop(0))
    return batch
```
- 功能：从请求队列中“批量捞取”请求，直到批次总 Token 数达到 `max_tokens`（或队列空）。
- 关键细节：
  - `request.tokens`：这里指请求的「输入 Token 数（input_tokens）」（因为 Prefill 阶段只处理输入上下文）。
  - 限制条件 `sum(...) < max_tokens`：确保 Prefill 阶段的总输入长度不超过 `max_batch_prefill_tokens`（避免 GPU 单次前向传播压力过大）。


### 3. 初始批次构建：填充第一批请求
```python
# Add initial requests to the prefill batch
batch = add_requests_to_prefill_batch(request_queue, batch, max_batch_prefill_tokens)
```
- 作用：从 `request_queue` 中捞取请求，构建第一批活跃批次（满足“总输入 Token 数 ≤ max_batch_prefill_tokens”）。
- 示例：假设 `max_batch_prefill_tokens=1024`，队列中有 3 个请求（输入 Token 数 512、300、400），则第一批会加入前 2 个（512+300=812 < 1024），第 3 个（400）因 812+400=1212>1024 暂时不加入。


### 4. 第一批 Prefill：启动初始推理
```python
# Prefill the batch
prefill(batch)
```
- 作用：对第一批请求执行 Prefill 阶段：
  1. CPU 对每个请求的输入文本分词（已在请求入队前完成，`input_tokens` 是分词结果）；
  2. 将所有请求的输入 Token 传给 GPU，执行单次前向传播；
  3. 生成初始 Token 和 KV 缓存（存入 GPU 显存，避免后续 Decode 重复计算）。


### 5. 主循环：动态管理批次（持续批处理核心）
```python
# Main loop to manage requests
while batch:  # 只要活跃批次非空，就持续处理（直到所有请求完成）
```
- 循环条件：`batch` 非空（有正在处理的请求），确保所有活跃请求都能完成解码。


#### 5.1 更新 Token 预算：计算当前批次已占用的 Token 数
```python
# Update the token budget based on current batch
# 计算当前批次所有请求的“总 Token 占用”：输入长度 + 最大输出长度（预估总占用，留足显存）
batch_max_tokens = sum(request.input_tokens + request.max_new_tokens for request in batch)
# 剩余可用预算 = 全局上限 - 当前批次已占用的 Token 数
token_budget = max_batch_total_tokens - batch_max_tokens
```
- 关键逻辑：用「输入长度 + 最大输出长度」预估每个请求的总 Token 占用（因为 Decode 阶段会生成最多 `max_new_tokens` 个 Token，KV 缓存会随输出增长），确保显存预算预留充足，避免 OOM。
- 示例：假设 `max_batch_total_tokens=4096`，当前批次有 2 个请求（A：input=512，max_new=1024；B：input=300，max_new=512），则 `batch_max_tokens=512+1024 + 300+512=2348`，`token_budget=4096-2348=1748`（还能容纳总占用 ≤1748 的新请求）。


#### 5.2 加入新请求：利用剩余预算动态扩容批次
```python
# Add new requests to the batch based on token budgets
# 新批次的预算上限：取两个值的较小者（避免 Prefill 超上限 + 避免总预算超上限）
new_batch_budget = min(max_batch_prefill_tokens, token_budget)
# 从队列中捞取新请求，构建新批次（满足新批次总输入 ≤ new_batch_budget）
new_batch = add_requests_to_batch(request_queue, [], new_batch_budget)
```
- 核心设计：`new_batch_budget` 是双重限制，确保：
  1. 新批次的 Prefill 阶段不超过 `max_batch_prefill_tokens`（GPU 单次 Prefill 能力上限）；
  2. 新批次的总占用（input + max_new）不超过剩余 `token_budget`（全局显存上限）。
- 注意：`add_requests_to_batch` 是 `add_requests_to_prefill_batch` 的简化写法（逻辑完全一致，都是“捞请求直到预算用尽”）。


#### 5.3 新请求 Prefill：合并到活跃批次
```python
# If new requests were added, handle prefill and decoding
if new_batch:
    # 对新批次执行 Prefill（生成初始 Token 和 KV 缓存）
    prefill(new_batch)
    # 将新批次合并到当前活跃批次（一起参与后续 Decode，实现“持续批处理”）
    batch.extend(new_batch)
```
- 关键：新请求不需要等当前批次解码完成，而是直接 Prefill 后合并到活跃批次，和老请求一起迭代解码——这就是 TGI 持续批处理的核心（区别于静态批处理“一批做完再做下一批”）。
- 示例：之前队列中未加入的第 3 个请求（input=400，max_new=800），总占用=400+800=1200 ≤ 剩余预算 1748，且 input=400 ≤ max_batch_prefill_tokens=1024，所以会被加入 `new_batch`，Prefill 后合并到 `batch`（此时 `batch` 有 3 个请求）。


#### 5.4 解码迭代：所有活跃请求同步生成 Token
```python
# Decode the current batch
decode(batch)
```
- 作用：对活跃批次中的所有请求执行 **一次解码迭代**（生成 1 个新 Token）：
  1. 每个请求基于自身的 KV 缓存和已生成 Token，计算下一个 Token；
  2. 更新每个请求的 `tokens_generated`（+1）；
  3. 若生成 EOS Token，标记 `request.reached_EOS=True`。
- 注意：Decode 是“批量并行”的——GPU 一次运算同时处理批次中所有请求的解码，而非逐个处理，这是提升吞吐量的关键。


#### 5.5 过滤完成请求：释放显存预算
```python
# Filter out completed requests that have reached EOS or max_new_tokens
batch = [request for request in batch if not request.reached_EOS and request.tokens_generated < request.max_new_tokens]

# Update token budget by subtracting tokens from completed requests
# （注：原代码此处有逻辑冗余，修正后更清晰）
# 计算已完成请求的“实际总占用”，释放对应预算
completed_requests = [req for req in batch if req.reached_EOS or req.tokens_generated >= req.max_new_tokens]
for request in completed_requests:
    # 释放的预算 = 该请求的“预估占用”（input + max_new）- 实际占用（input + tokens_generated）
    token_budget += (request.input_tokens + request.max_new_tokens) - (request.input_tokens + request.tokens_generated)
```
- 核心逻辑：
  1. 移除已完成的请求（EOS 或达到最大输出长度），避免无效解码；
  2. 释放已完成请求的“显存冗余”：因为之前预估占用是 `input + max_new`，但实际占用是 `input + 实际生成数`，差值就是可回收的预算，后续可用于加入更多新请求。
- 示例：请求 A 实际生成 800 个 Token 就触发 EOS（`max_new=1024`），则释放的预算= (512+1024) - (512+800) = 224，`token_budget` 会增加 224，可用于加入更小的新请求。


## 三、整体逻辑流程图（一目了然）
```
初始化：batch=[]，token_budget=全局最大总Token数
↓
从请求队列捞请求 → 构建第一批batch（≤ max_batch_prefill_tokens）
↓
对第一批batch执行Prefill（生成初始KV缓存）
↓
进入主循环（batch非空）：
    1. 计算当前batch的预估总占用 → 更新剩余token_budget
    2. 用剩余预算捞新请求 → 构建new_batch
    3. 若有new_batch → Prefill新请求 → 合并到当前batch
    4. 对当前batch执行一次Decode（生成1个Token）
    5. 过滤已完成请求 → 释放显存预算
↓
直到batch为空（所有请求完成）→ 循环结束
```


## 四、关键设计亮点（对应 TGI 核心优势）
这段代码完美体现了 TGI 持续批处理的核心设计，解决了静态批处理的痛点：
### 1. 动态扩容，提升吞吐量
新请求无需等待前一批结束，只要有显存预算就可以加入活跃批次，最大化 GPU 利用率（比如老请求快完成时，新请求可以“插空”利用剩余显存）。

### 2. 双重预算限制，避免 OOM
- `max_batch_prefill_tokens` 限制单次 Prefill 的输入长度，避免 GPU 单次前向传播压力过大；
- `max_batch_total_tokens` 限制所有请求的总占用，确保显存不溢出。

### 3. 批量并行解码，降低延迟
Decode 阶段同时处理批次中所有请求，而非逐个处理，充分利用 GPU 的并行计算能力，降低单 Token 生成时间（TPOT）。

### 4. 预算动态回收，优化显存利用
已完成请求的“预估占用 vs 实际占用”差值会被回收，避免显存浪费，可容纳更多请求。


## 五、潜在优化（原代码的小瑕疵）
原代码中 `completed_requests` 的筛选逻辑有冗余（先过滤再筛选已完成请求，此时 `batch` 中已无完成请求），修正后的逻辑更准确（见 5.5 节注释），核心是“先筛选完成请求→更新预算→再过滤批次”。


## 总结
这段代码是 TGI 持续批处理的“最小可行实现”，核心目标是 **在显存安全的前提下，通过动态批次管理最大化 GPU 利用率**。其本质是：将“静态的一批请求”变成“动态的、可扩容的请求集合”，让 GPU 始终处于高效运算状态，从而在大规模 LLM 推理中平衡低延迟和高吞吐量——这也是 TGI 能成为生产环境首选推理工具的核心原因。

