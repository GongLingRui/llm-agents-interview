# VisualWebArena: Evaluating Multimodal Agents on Realistic Visually Grounded Web Tasks

## Metadata

- Authors / Year / Venue: Koh et al. · 2024 · arXiv:2401.13649
- Links: https://arxiv.org/abs/2401.13649
- Tags: browser-computer-use, evaluation
- Reference: [paper-visualwebarena-2024](../../../references/papers.md#paper-visualwebarena-2024)
- Last reviewed: 2026-10-07

## TL;DR

在可复现 Web 环境上强调 **视觉 grounding** 任务；说明纯文本/DOM 通道不足以覆盖大量真实 GUI 任务。

## Problem setup

许多网页任务依赖图像、图标、布局，而非可访问的纯文本节点。

## Method / Mechanism

扩展 WebArena 类环境：任务需要截图/多模态感知才能正确定位与操作；评测仍强调功能正确性。

## How it changes the Agent Loop

- 感知层必须支持 **hybrid：DOM + 截图/a11y**。
- Eval 集需包含视觉依赖切片，否则「文本 Agent」虚高。
- 动作空间仍归 Tool Use；本文贡献在评测与感知需求。

## Experiments that matter

- 视觉 grounding 任务上，纯文本/DOM Agent 明显弱于需要截图的设定（相对 WebArena 文本通道虚高）。
- 多模态基线有增益，但端到端成功率仍远低于人类（见原文；自托管环境重跑后再定 SLO）。
- 失败常来自 UI 元素定位错误与视觉幻觉，而非「少写一步推理」。

## Engineering takeaways

1. Browser Agent 默认规划视觉通道与 token 预算。
2. 截图全量进窗会爆 Context——用裁剪/标注/工具描述。
3. 成功标准看业务状态，不看「点击了看起来对的像素」。

## Limitations & failure modes

- 视觉模型幻觉 UI 元素。
- 分辨率/主题变化导致脆弱。

## When you should / shouldn't use it

| 应该 | 不应该 |
|------|--------|
| UI 强视觉依赖站点 | 纯 API/可访问树已足够的内部工具 |

## Related handbook modules

- [browser-computer-use](../../../agent-types/browser-computer-use/browser-computer-use.md)
- [evaluation](../../../production/evaluation/evaluation.md)

## References

- [paper-visualwebarena-2024](../../../references/papers.md#paper-visualwebarena-2024)
- [paper-webarena-2024](../../../references/papers.md#paper-webarena-2024)
