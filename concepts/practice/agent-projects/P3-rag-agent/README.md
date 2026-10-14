# P3 — RAG-backed Agent

## Goal

接入检索→引用→回答/行动，能区分检索失败与推理失败。

## Non-goals

- 自建大规模索引运维
- Agentic RAG 全家桶作为默认

## Depends on modules

- `P2`
- `core/rag`
- `core/context-engineering`

## Architecture sketch

```text
Retrieve → Evidence pack(untrusted) → Assembler → Answer with citations
```

## Acceptance criteria

- 回答可指向检索片段 ID
- 无命中时行为可测（拒答/澄清，而非幻觉）
- 能报告 retrieval_miss vs grounded_wrong

## Failure modes observed

（练习时填写实际观测；下列为预期高发）

- 检索噪声主导决策
- 引用与正文不一致
- 把 RAG 当 Memory

## What I'd improve next

- （练习后填写）

## References

- 路线：[docs/practice-paths.md](../../../docs/practice-paths.md)
- 清单：[../README.md](../README.md)
