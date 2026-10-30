---
tags: [atom, ai, core, A100, spacemit]
domain: hardware
aliases: [A100-AI-Core, A100 AI Core, A100 AI核心, A100核心]
status: approved
created: 2026-10-29
updated: 2026-10-29
source: C:\Users\luojunlin\docs-chip\zh\key_stone\
superseded_by:
---

# A100 AI Core

**中文译名**：A100 AI 核心

## 定义

A100 AI Core 是 SpacemiT K3 芯片中的专用 AI 加速核心，采用超宽并行计算架构，用于加速深度学习推理中的矩阵运算。

## 关键参数

- **核心数量**：K3 配备 8×A100 AI 核心
- **整机算力**：约 60 TOPS
- **支持数据类型**：BF16 / FP16 / FP8 / INT8 / INT4
- **功能单元**：
  - 矩阵运算单元（Matrix Compute Unit）
  - 向量运算单元（Vector Compute Unit）
- **架构特点**：超宽并行，专为矩阵乘加（GEMM）等 AI 运算优化

## 为什么重要

AI 推理的核心计算是大量矩阵乘法（GEMM）。A100 专用硬件相比通用 CPU 核心：
- **算力提升**：同功耗下提供数十倍算力
- **能效提升**：每 TOPS 功耗更低
- **数据类型优化**：原生支持低精度量化（INT8/INT4）

## 与其他厂商对比

| 厂商 | AI 加速方案 | 算力 | 特点 |
|------|------------|------|------|
| SpacemiT K3 | 8×A100 AI Core | ~60 TOPS | 独立 AI 核心，与 CPU 解耦 |
| NVIDIA Jetson Orin | Ampere GPU | 275 TOPS | GPU 通用架构 |
| ARM Ethos-U65 | NPU | 1-2 TOPS | 边缘低功耗方案 |

## 应用场景

- 本地大语言模型推理（7B-13B 参数）
- 边缘 AI 服务器
- 实时视觉推理（目标检测、分割）
- 边缘智能计算

## 与 Matrix Extension 的关系

- **Matrix Extension** 是指令集级别的扩展（K1 Cluster0 采用）
- **A100 AI Core** 是独立的硬件加速核心（K3 专属）
- 两者都用于加速矩阵运算，但实现层次不同

## 相关词条

- [[X100-Core]] — K3 的 CPU 核心
- [[Matrix-Extension]] — SpacemiT 矩阵指令集扩展
- [[../hardware/products/K3]] — 使用 A100 AI 核心的产品
- [[../ai/AI-SDK]] — 利用 A100 算力的软件开发套件

## 来源文档

- K3 产品简介：https://cdn-resource.spacemit.com/file/chip/K3/K3_brief_zh.pdf
- K3 Datasheet（待发布）
- K3 User Manual（Chapter 8: AI Core Architecture）
