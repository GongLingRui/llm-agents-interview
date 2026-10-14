# CacheBlend: Cached Knowledge Fusion for RAG Serving

- **链接**：https://arxiv.org/abs/2405.16444  
- **关联讲次**：L14；客座 Junchen Jiang  
- **会议**：EuroSys 2025

## 问题

RAG 输入含多段检索文本时，prefix cache 只能覆盖第一段；full KV reuse 忽略跨段 cross-attention，伤质量。

## 机制

- 预计算各 chunk KV  
- Prefill 时 **选择性重算** 一小部分 token 的 KV（常 <15%）以恢复跨注意力  
- 重算与从慢存储取 KV **pipeline**，便于 CPU/磁盘卸载

## 关键数字

| vs full recompute | TTFT **2.2–3.3×**↓；吞吐 **2.8–5×**↑ |
| 质量 | 接近 full prefill；优于 naive full reuse |

## 工程 Takeaway

- 多文档 agent / RAG 应默认考虑非前缀复用。  
- 「少量重算」是质量–速度折中的关键旋钮。  
- 实现见 LMCache（`LMCACHE_ENABLE_BLENDING` 等）。

## 一句话

融合多段预计算 KV：只重算关键 token，既快又接近全量 prefill 质量。
