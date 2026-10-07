# OSWorld: Benchmarking Multimodal Agents for Open-Ended Tasks in Real Computer Environments

## Metadata

- Authors / Year / Venue: Xie et al. · 2024 · arXiv:2404.07972
- Links: https://arxiv.org/abs/2404.07972
- Tags: browser-computer-use, evaluation, sandbox
- Reference: [paper-osworld-2024](../../../references/papers.md#paper-osworld-2024)
- Last reviewed: 2026-10-07

## TL;DR

真实计算机环境中的开放 **Computer Use** 基准：跨应用桌面交互，评测全环境 GUI Agent。

## Problem setup

浏览器基准无法覆盖多应用桌面工作流（文件管理器、办公套件、设置等）。

## Method / Mechanism

在真实 OS 环境布置开放任务；Agent 通过截图/a11y 等感知并产生点击/键入等动作；以任务完成态评测。

## How it changes the Agent Loop

- 将「浏览器 Agent」推广为 **桌面 Computer Use**：沙箱与权限面显著扩大。
- Loop 不变；工具=OS 动作；Security/Sandbox 成为硬前置。

## Experiments that matter

- 开放桌面任务上，强多模态 Agent 端到端成功率远低于人类（见原文基线表；部署时在自托管快照环境重跑）。
- 跨应用工作流（文件/办公/设置）比单页 Web 更暴露探索与恢复不足。
- 工程信号：以任务完成态/环境断言评测，不靠动作序列匹配。

## Engineering takeaways

1. 默认可还原快照环境；禁止裸金属跑不可信 Agent。
2. 高危动作（安装、外发、删文件）HITL。
3. Eval 用环境断言，不靠动作序列匹配。

## Limitations & failure modes

- 逃逸与供应链风险高于纯 Web。
- 任务开放导致评测贵、噪声大。

## When you should / shouldn't use it

| 应该 | 不应该 |
|------|--------|
| 真桌面自动化研发与评测 | 只需浏览器或 API 集成时 |

## Related handbook modules

- [browser-computer-use](../../../agent-types/browser-computer-use/browser-computer-use.md)
- [sandbox](../../../runtime/sandbox/sandbox.md)
- [security](../../../production/security/security.md)
- [evaluation](../../../production/evaluation/evaluation.md)

## References

- [paper-osworld-2024](../../../references/papers.md#paper-osworld-2024)
