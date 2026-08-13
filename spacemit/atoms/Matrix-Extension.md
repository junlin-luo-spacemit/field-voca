---
tags: [atom, matrix, ai, spacemit]
domain: hardware
aliases: [Matrix Extension, Matrix Engine, 矩阵扩展, 矩阵引擎]
status: approved
created: 2026-08-06
updated: 2026-08-06
superseded_by:
---

# Matrix Extension / Matrix Engine

## 定义

矩阵扩展（Matrix Extension）指的是在处理器中加入专门加速矩阵运算的指令集或硬件单元，用于高效执行深度学习中的矩阵乘加（GEMM）等计算。

## 为什么重要

深度学习的核心计算是大量矩阵乘法，Matrix Engine 可以显著提高吞吐量并降低能耗，是衡量 AI SoC 能力的关键指标之一。

## 关注点

- 支持的数据类型（FP16 / BF16 / INT8 / INT4 等）
- 每周期矩阵吞吐（MACs）
- 是否支持稀疏、混合精度与向量化指令

## 在 SpacemiT 中

- 文档中提到 `SpacemiT AI Matrix Extension Instruction Set`，为其 AI 平台提供矩阵加速能力。

**官方参考文档：** SpacemiT 文档中心中有专门章节“Matrix Extension Instruction Set”（文档中心搜索或浏览 AI / Compute Architecture 小节可见），入口：https://www.spacemit.com/community/document?lang=en
