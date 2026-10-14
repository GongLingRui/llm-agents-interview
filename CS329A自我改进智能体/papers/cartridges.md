# Cartridges: Long Context Representations via Self-Study

- **链接**：https://arxiv.org/abs/2506.06266  
- **关联讲次**：L14；L20（非权重 continual）  
- **机构**：Stanford Hazy Research 等

## 问题

同一大语料被反复 ICL → KV 显存与 prefill 成本爆炸。能否离线训小 KV（Cartridge）在推理时模拟 ICL？

## 机制

**Self-Study**：

1. 对语料生成合成对话（分块 + seed prompts 促结构推理）  
2. **Context distillation**：对齐「全文 ICL」教师分布  
3. 冻结 LLM，训练 prefix-style KV Cartridge  

反例：语料 NTP 能背但毁通用问答。

## 关键数字

| 指标 | 值 |
|------|-----|
| 记忆 | 约 **38.6×** 少于 ICL KV |
| 吞吐 | 约 **26.4×**↑（多语料服务场景） |
| 外推 | 484k textbook Cartridge 可超 130k ICL（MTOB） |
| 组合 | 多 Cartridge 可拼接无需重训 |

## 工程 Takeaway

- 适合稳定、高复用语料（repo、政策、手册）。  
- 离线算力换在线记忆；与 serving 的 per-user KV 模型契合。  
- KV 参数化优于 LoRA 注入（文内）。

## 一句话

用自问自答蒸馏把大语料压成可热加载的小 KV，近似 ICL 且便宜一个数量级。
