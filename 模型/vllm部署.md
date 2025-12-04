

# 大规模vLLM部署中的重难点
- Kubernetes+容器化部署
- vLLM状态监测
  - `/health`及`/metrics`端口
  - 核心metrics监测
    - num waiting requests
    - num running requests
    - SLO related
      - Time to first token
      - “decoding throughput”
  - 实战：Prometheus+Grafana UI配置
- 负载均衡和路由算法
- 自动扩容/缩容
- 容灾处理
- 示例代码和教程（GitHub搜索）
  - vllm-project/vllm
  - vllm-project/production-stack
 
  # 大规模vLLM部署中的重难点
- Kubernetes+容器化部署
- vLLM状态监测
- 负载均衡和路由算法
  - 负载均衡：如何判断overload
  - 实战中路由算法实例
    - Prefix-caching
    - Round-robin
    - Session-based
    - 最大前缀匹配
- 自动扩容/缩容
- 容灾处理
- 示例代码和教程（GitHub搜索）
  - vllm-project/vllm
  - vllm-project/production-stack
 

# 大规模vLLM部署中的重难点
- Kubernetes+容器化部署
- vLLM状态监测
- 负载均衡和路由算法
- 自动扩容/缩容
  - 实例：使用Kubernetes HPA + Prometheus Adapter
  - 何时触发？
- 容灾处理

<img width="754" height="441" alt="image" src="https://github.com/user-attachments/assets/bc819df0-ece2-4bc8-b564-1b5272a80a7e" />

- Kubernetes容器化部署
- vLLM状态监测
- 负载均衡和路由算法
- 自动扩容/缩容
- 容灾处理
- 业务层异常捕获
- 动态更新路由
- 重新发送请求
- 
