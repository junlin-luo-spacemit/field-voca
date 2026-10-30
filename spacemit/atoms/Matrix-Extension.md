---
tags: [atom, matrix, ai, spacemit]
domain: hardware
aliases: [Matrix Extension, Matrix Engine, 矩阵扩展, 矩阵引擎]
status: approved
created: 2026-08-06
updated: 2026-10-29
source: C:\Users\luojunlin\docs-ai\zh\architecture\ime_extension.md
superseded_by:
---

# Matrix Extension / Matrix Engine

**中文译名**：矩阵扩展 / 矩阵引擎

## 定义

矩阵扩展（Matrix Extension）指的是在处理器中加入专门加速矩阵运算的指令集或硬件单元，用于高效执行深度学习中的矩阵乘加（GEMM）等计算。

## 为什么重要

深度学习的核心计算是大量矩阵乘法，Matrix Extension 可以显著提高吞吐量并降低能耗，是衡量 AI SoC（系统级芯片）能力的关键指标之一。

## 关注点

- 支持的数据类型（FP16 / BF16 / INT8 / INT4 等）
- 每周期矩阵吞吐（MACs）
- 是否支持稀疏、混合精度与向量化指令

## 在 SpacemiT 中

SpacemiT 提供两种矩阵加速方案：

1. **K1 Cluster0 AI Extension** — 指令集级扩展
   - 在 X60 CPU 核心中添加 AI 自定义指令（如 `smt.vmadot`）
   - 约 2 TOPS 算力
   
2. **K3 A100 AI Core** — 独立硬件加速核心
   - 8×A100 专用 AI 核心
   - 约 60 TOPS 算力

**官方参考文档：** SpacemiT 文档中心有专门章节"Matrix Extension Instruction Set"（文档中心搜索或浏览 AI / Compute Architecture 小节可见），入口：https://www.spacemit.com/community/document?lang=en

## 相关词条

- [[A100-AI-Core]] — K3 的专用 AI 加速核心
- [[X60-Core]] — K1 的 CPU 核心
- [[../hardware/products/K1]] — 使用 AI Extension 的产品
- [[../hardware/products/K3]] — 使用 A100 AI Core 的产品
