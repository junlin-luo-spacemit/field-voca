---
tags: [nvidia, cuda, hardware, atom]
domain: cuda
aliases: [Tensor Core, 张量核心]
date: 2026-06-29
source: https://docs.nvidia.com/cuda/cuda-programming-guide/
---

# Tensor Core / 张量核心

## 定义

**Tensor Core** 是 [[SM]] 内专用于**矩阵乘累加（MMA, Matrix Multiply-Accumulate）** 的硬件单元，是 AI 训练/推理的核心算力来源。

## 官方原文

> "Starting with the Volta architecture, Tensor Cores provide matrix multiply and accumulate (MMA) operations."

## 各代规格对比

| 代数 | 架构 | 精度支持 | 峰值算力（H100） |
|------|------|----------|-----------------|
| 第1代 | Volta (V100) | FP16 | — |
| 第2代 | Turing (T4) | FP16, INT8, INT4 | — |
| 第3代 | Ampere (A100) | FP16, BF16, TF32, FP64, INT8 | 312 TFLOPS (FP16) |
| 第4代 | Hopper (H100) | FP8, FP16, BF16, TF32, FP64 | 989 TFLOPS (FP16) |
| 第5代 | Blackwell (B200) | FP4, FP6, FP8, FP16, BF16 | ~2.25 PFLOPS (FP8) |

## 工作原理

- 执行 $D = A \times B + C$ 矩阵运算，一条指令完成整块矩阵乘累加
- 配合 [[warp]] 内所有32个线程协同完成一次 MMA tile 计算
- 由 WMMA API 或 cuBLAS / [[TensorRT]] 自动调用，不需手动编程

## 相关词条

- [[SM]] — Tensor Core 所在的硬件单元
- [[CUDA-Core]] — 标量计算对比
- [[warp]] — Tensor Core 的调度单位
- [[TensorRT]] — 自动利用 Tensor Core 的推理框架
- [[HBM]] — 为 Tensor Core 喂数据的高带宽内存
