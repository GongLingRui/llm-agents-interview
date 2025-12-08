https://medium.com/%40alishafique3/pytorch-training-optimizations-5-throughput-with-gpu-profiling-and-memory-analysis-31cb2b1f95cc

# PyTorch训练优化网页总结
## 一、文档基础信息
- **标题**：PyTorch training optimizations: 5× throughput with GPU profiling and memory analysis
- **作者**：Ali Shafique
- **发布时间**：2024年4月29日
- **阅读时长**：9分钟
- **核心目标**：基于PyTorch框架，在单个GPU上通过GPU分析和内存分析，结合多种优化技术将训练吞吐量提升5倍

## 二、环境配置与基础准备
### 1. 依赖环境
- 基于NVIDIA PyTorch容器镜像（23.10版本，可在NGC获取）构建
- 系统与库：Ubuntu 22.04（含Python 3.10）、NVIDIA cuDNN 8.9.5、PyTorch 23.10

### 2. Docker启动命令
| Docker版本 | 启动命令 |
|----|----|
| 19.03及以上 | `docker run --gpus all -it --rm nvcr.io/nvidia/pytorch:xx.xx-py3`（xx.xx为23.10） |
| 19.02及以下 | `nvidia-docker run -it --rm -v nvcr.io/nvidia/pytorch:xx.xx-py3`（xx.xx为23.10） |

### 3. 基准模型
- **基础代码来源**：PyTorch示例“PyTorch Profiler With TensorBoard”（可通过指定链接获取，访问时间2024年2月2日），完整代码见GitHub仓库
- **模型与数据集**：基于Mobilenet_V2架构的分类模型，训练数据集为CIFAR10
- **初始性能（未优化）**：步长时间117毫秒，GPU利用率73.28%，每训练步内存占用约2.5GB，吞吐量273样本/秒

### 4. 关键术语解释
- **GPU利用率/GPU繁忙时间**：“所有步骤时间”内GPU至少有一个内核运行的时间，值越高越好，但无法反映流多处理器（SM）使用数量
- **预估SM效率**：流多处理器实际计算工作量与理想条件下最大可能工作量的比值，衡量SM在并行工作负载执行中的利用效率
- **估计的已实现占用率**：特定时间内活动线程束与SM可容纳最大线程束数量的比值，反映GPU内核对硬件资源的利用效率


## 三、五大核心优化技术
### 1. 自动混合精度（AMP）
- **原理**：自动将模型特定部分转换为16位浮点数，在GPU张量核心执行；保留32位主权重累积权重更新，采用损失缩放维持小梯度值，充分利用GPU张量核心性能
- **代码修改**：在训练函数中添加`torch.autocast(device_type='cuda', dtype=torch.float16)`上下文管理器
- **优化效果**：张量核心利用率从0%升至19%，GPU内存占用从2.5GB降至1.3GB，吞吐量从273样本/秒提升至395样本/秒，步长时间从117毫秒缩短至81毫秒

### 2. 增大批次大小
- **原理**：AMP优化释放了GPU内存，为增大批次大小提供空间，提升GPU资源利用率
- **代码修改**：将`torch.utils.data.DataLoader`的`batch_size`参数从32调整为128
- **优化效果**：吞吐量从395样本/秒提升至551样本/秒，步长时间变为232毫秒（批次大小128时）

### 3. 减少主机到设备（H2D）数据复制
- **原理**：将数据类型转换（8位无符号整数转32位浮点数）和标准化操作延迟到数据传输至GPU后执行，减少CPU操作与主机到设备的数据传输瓶颈
- **代码修改**：在`transform`中移除`T.Normalize`，在训练函数中对GPU上的输入数据执行`(inputs.to(torch.float32) / 255. - 0.5) / 0.5`操作
- **优化效果**：CPU执行时间显著减少，GPU利用率提升，吞吐量从551样本/秒提升至631样本/秒，步长时间从232毫秒缩短至202毫秒

### 4. 多进程数据加载
- **原理**：利用多CPU核心并行加载和预处理数据，加快数据加载速度，避免GPU等待数据
- **代码修改**：在`torch.utils.data.DataLoader`中添加`num_workers=2`参数
- **注意事项**：Docker默认CPU多进程共享内存64MB不足，需通过`--shm-size=2gb`参数分配2GB共享内存（启动命令：`docker run --gpus=all --rm -it --net=host --shm-size=2gb nvcr.io/nvidia/pytorch:23.10-py3`）
- **优化效果**：步长时间从202.6毫秒缩短至145.8毫秒，吞吐量提升至877.84样本/秒

### 5. 内存锁定（Pinned Memory）
- **原理**：使用锁定内存提升主机到设备数据传输速度，支持异步操作，可在GPU执行当前批次训练时并行准备下一批次数据
- **代码修改**：一是将`DataLoader`的`pin_memory`参数设为`True`；二是在数据传输时添加`non_blocking=True`参数（`data[0].to(device=device, non_blocking=True)`）
- **优化效果**：GPU利用率从79.54%升至86.22%，步长时间从145.8毫秒缩短至92.6毫秒，吞吐量提升至1381.6样本/秒


## 四、优化结果汇总（基于Quadro RTX 4000 GPU，8GB内存）
| 优化技术 | 批次大小 | GPU内存占用（GB） | 平均步长时间（毫秒） | 每秒样本数 | 相对基准模型优化比例 |
|----|----|----|----|----|----|
| 基准模型 | 32 | 2.51 | 117.1 | 273.5 | 100% |
| 自动混合精度 | 32 | 1.32 | 81.1 | 395.1 | 144% |
| 增大批次大小 | 128 | 4.95 | 232.1 | 551.5 | 201% |
| 减少H2D复制 | 128 | 4.95 | 202.6 | 631.63 | 230% |
| 多进程数据加载 | 128 | 4.95 | 145.8 | 877.84 | 320% |
| 内存锁定 | 128 | 4.95 | 92.6 | 1381.6 | 505% |


## 五、结论与参考文献
### 1. 结论
通过自动混合精度、增大批次大小、减少H2D复制、多进程数据加载、内存锁定五项优化技术，结合PyTorch Profiler和TensorBoard插件的性能分析，成功将PyTorch训练阶段性能提升5倍，这些技术对大型模型（如基础模型）的训练具有重要实用价值，仅需少量代码修改和内存分析即可实现显著优化效果。

### 2. 参考文献
- PyTorch Model Performance Analysis and Optimization：[https://towardsdatascience.com/pytorch-model-performance-analysis-and-optimization-10c3c5822869](https://towardsdatascience.com/pytorch-model-performance-analysis-and-optimization-10c3c5822869)
- PyTorch Profiler With TensorBoard：[https://pytorch.org/tutorials/intermediate/tensorboard_profiler_tutorial.html](https://pytorch.org/tutorials/intermediate/tensorboard_profiler_tutorial.html)
- 内存锁定相关：Lecture 21 — Pinned Memory and Streams（原文含指定链接）
