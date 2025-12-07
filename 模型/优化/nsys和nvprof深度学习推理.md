https://medium.com/%40rc3729/profiling-deep-learning-inference-with-nsight-systems-and-nvprof-a-practical-guide-c1e5b49f0627

# 使用Nsight Systems和nvprof分析深度学习推理：实用指南 - 网页总结
## 一、核心背景与意义
1. **性能需求**：现代机器学习、科学计算及图形应用需充分挖掘GPU性能，但即使优化后的代码也可能存在内核启动、内存传输、CPU-GPU同步等隐藏瓶颈。
2. **推理优化必要性**：随着深度学习向实时边缘部署发展（如Jetson Nano嵌入式设备、云端高性能GPU），优化推理性能从“可选项”变为“必需品”，性能分析工具是发现瓶颈、指导系统设计的关键。
3. **性能分析价值**：解决深度学习推理中的延迟、内存效率低、计算单元利用率不足问题，具体可实现：了解GPU与CPU利用率、识别内存瓶颈、测量逐层推理时间、优化主机-设备数据传输、微调内核启动与执行时间线，避免训练良好的模型在实际应用中表现不佳。

## 二、核心分析工具概述
|工具名称|类型|核心功能|适用场景|局限性/备注|
|----|----|----|----|----|
|NVIDIA Nsight Systems|全系统分析器（含GUI）|提供GPU与CPU活动时间线视图，呈现内核执行重叠、同步停滞、CPU线程行为等高级性能洞察|识别性能下降的时间与位置，进行端到端性能分析|需安装后通过命令行启动，依赖CUDA环境|
|nvprof|命令行分析器|提供GPU内核详细统计数据，包括内存访问、浮点运算、内核启动指标|轻量级分析，快速获取基础性能数据|从CUDA 11起已弃用，长期兼容性需改用Nsight Systems/Nsight Compute|

## 三、环境设置与先决条件
1. **硬件要求**：配备兼容CUDA的GPU。
2. **软件要求**：安装Nsight Systems（`nsys`）和nvprof（均为CUDA工具包组成部分），准备深度学习模型脚本（如基于PyTorch或TensorRT的YOLOv10推理脚本）。
3. **Ubuntu桌面安装Nsight Systems步骤**：
    1. 从[NVIDIA官网](https://developer.nvidia.com/nsight-systems)获取对应架构（x86_64/Arm SBSA）的安装包；
    2. 执行命令：
    ```py pz qa qb qc qd qe qf bq qg bc bl
    sudo apt-key adv --fetch-keys https://developer.download.nvidia.com/compute/cuda/repos/ubuntu1804/x86_64/7fa2af80.pub
    sudo add-apt-repository "deb https://developer.download.nvidia.com/devtools/repos/ubuntu$(source /etc/lsb-release; echo "$DISTRIB_RELEASE" | tr -d .)/$(dpkg --print-architecture)/ /"
    sudo apt install nsight-systems
    ```

## 四、工具使用步骤与分析重点
### （一）Nsight Systems使用
1. **步骤1：运行分析器**
    - 命令：`nsys profile -o profile_output --trace=cuda,nvtx,osrt python file.py --input input.mp4`
    - 参数解释：`--trace=cuda,nvtx,osrt`捕获CUDA活动、自定义NVTX范围及操作系统运行时行为；`-o profile_output`将结果保存为`profile_output.nsys-rep`文件。
2. **步骤2：打开报告**
    - 命令：`nsys-ui profile_report.nsys-rep`（本地启动Nsight图形界面）。
3. **步骤3：时间线分析重点**
    - 内核重叠情况（判断计算任务是否并行化）；
    - 内存传输（主机到设备、设备到主机）；
    - 同步延迟（如CUDA内存复制、`cudaDeviceSynchronize()`函数）。
4. **分析结果价值**：输出吞吐量、CPU/GPU延迟分布（最小/最大/平均/中位数）、百分位细分等关键指标。例如，批量大小为1时，YOLO模型（TensorRT）可实现122.871 QPS吞吐量，CPU平均延迟8.14ms、GPU平均延迟8.10ms，反映计算利用率平衡情况，还可辅助分析延迟一致性、定位瓶颈组件。

### （二）nvprof使用（轻量级）
1. **基础命令**：`nvprof --unified-memory-profiling off --dependency-analysis --log-file profiling_log.txt python3 inference.py ./input.mp4`（将分析结果保存至日志文件）。
2. **示例输出**：
    ```py pz qa qb qc qd qe qf bq qg bc bl
    GPU activities:   75.3%  150.00ms  15  void kernel_conv2d(...)                   
    12.2%  24.32ms  3   void kernel_activation(...)
    Memory operations: 1.8GB transferred
    ```
3. **常用参数**：
    - `--print-gpu-trace`：生成时间线风格跟踪结果；
    - `--metrics flop_count_sp`：统计浮点运算数据；
    - `--csv`：导出日志至电子表格；
    - `nvprof --query-metrics`：查看可分析的指标列表。

## 五、基于分析的优化技巧
1. **批量大小调优**：避免过小（GPU利用率低）或过大（内存溢出），需结合实际场景平衡。
2. **层融合**：使用TensorRT或TorchScript融合模型层，减少内核启动次数，提升效率。
3. **数据传输最小化**：采用锁定内存（如通过`cudaHostAlloc()`函数或PyTorch DataLoader的`pin_memory=True`），减少主机-设备数据传输量。
4. **异步执行**：利用CUDA流，实现数据传输与计算任务并行处理。
5. **精度降低**：采用FP16（半精度）或INT8（整数）数据类型，减少计算量与内存占用，加快推理速度。

## 六、总结与参考资源
1. **核心结论**：性能分析是迭代过程，Nsight Systems和nvprof可帮助开发者脱离“猜测优化”，实现数据驱动优化，适用于深度学习训练、科学模拟、高性能图形开发等场景；建议从简单场景入手，标注代码，以数据指导优化方向。
2. **参考资源**：
    - [Nsight Systems 2025.3用户指南](https://docs.nvidia.com/nsight-systems/)
    - [Profiler 12.9文档（应用程序分析准备）](https://docs.nvidia.com/profiler/)
