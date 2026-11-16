---
tags: [ai, quantization, xslim, spacemit]
domain: ai
aliases: [XSlim, xslim]
status: approved
created: 2026-11-16
updated: 2026-11-16
source: C:\Users\luojunlin\docs-ai\en\compute_stack\ai_compute_stack\xslim.md
superseded_by:
---

# XSlim

## 定义

**XSlim** 是 **SpacemiT 自研**的训练后量化（Post-Training Quantization, PTQ）工具。它内置了针对 SpacemiT 芯片调优的量化策略，并通过一份 JSON 配置文件驱动统一 API，简化量化流程。

> 源码：<https://github.com/spacemit-com/xslim>

## 为什么重要

- **芯片专项调优**：量化策略针对 SpacemiT AI 核心（A60/A100）预先优化，开箱可用
- **统一 API**：JSON 配置驱动，无需修改代码即可切换量化方案
- **压缩收益显著**：量化后模型体积大幅减少，推理速度提升，适合边缘部署

## 核心功能

- **快速上手**：JSON 配置一键启动量化
- **量化配置**：支持多种量化格式（INT4、INT8 等）
- **精度调优**：内置精度损失补偿策略

## 在 SpacemiT 工作流中的位置

```
原始模型（FP32）
       ↓ XSlim（PTQ 量化）
量化模型（INT4/INT8）
       ↓
运行推理（ONNXRuntime / Llama.cpp）
```

## 相关词条

- [[ONNXRuntime]] — 量化后模型的推理引擎
- [[Llama.cpp]] — 支持 GGUF 量化格式的推理引擎
- [[../ai/AI-Architecture|AI 架构]] — 整个 AI 软件栈

## 来源文档

- `docs-ai/en/compute_stack/ai_compute_stack/xslim.md`
- 源码：<https://github.com/spacemit-com/xslim>
