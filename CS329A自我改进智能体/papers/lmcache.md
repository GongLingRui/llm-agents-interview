# LMCache（扩展阅读）

- **链接**：https://arxiv.org/abs/2510.09665 ；https://github.com/LMCache/LMCache  
- **关联讲次**：L14 客座 Junchen Jiang  
- **机构**：UChicago / Tensormesh 等

## 问题

企业级推理中 KV 总量远超 GPU；需要跨请求、跨引擎、跨存储层的 KV 编排。

## 机制（公开材料）

- KV 从 GPU offload 到 CPU / 本地盘 / 远端  
- 与 vLLM / SGLang 等解耦的缓存层  
- Prefix reuse + **CacheBlend 非前缀 reuse**  
- Prefill–decode 分离时的传输优化  
- 批处理搬运、compute/IO 流水线、可观测性

## 关键数字（论文宣称）

- 与 vLLM 结合，多轮 QA / 文档分析等负载吞吐最高约 **15×** 量级提升（视负载）  
- 产业观察：截断上下文等做法可严重伤害 prefix hit rate

## 工程 Takeaway

- Agent 产品要把 **KV 命中率** 当核心 SLO。  
- 记忆策略（MemGPT/Cartridge）最终仍落在 serving 能否便宜复用 KV。

## 一句话

把 KV cache 提升为可持久、可共享、可分层的推理基础设施。
