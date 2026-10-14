# runtime/

> 执行与模型侧能力 — 隔离、推理服务、压缩加速、微调与 RL。

| 模块 | Canonical | 边界提醒 |
|------|-----------|----------|
| [sandbox](sandbox/sandbox.md) | ✅ | 隔离机制；威胁模型见 [security](../production/security/security.md) |
| [llm-inference](llm-inference/llm-inference.md) | ✅ | Serving / KV / 批处理；非 Agent Loop |
| [model-optimization](model-optimization/model-optimization.md) | ✅ | 量化、投机、路由 |
| [fine-tuning](fine-tuning/fine-tuning.md) | ✅ | SFT/适配；多数问题先 Prompt/Tool |
| [reinforcement-learning](reinforcement-learning/reinforcement-learning.md) | ✅ | **Eval 先于 RL**；≠ 推理时反思 |

阅读建议：Sandbox ↔ Security 对照；Inference → Optimization；FT → RL（有评测再上）。
