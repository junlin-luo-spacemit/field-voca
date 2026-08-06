---
tags: [nvidia, cuda, hardware, atom]
domain: cuda
aliases: [CUDA Core, CUDA核心, FP32 Core]
date: 2026-06-29
source: https://docs.nvidia.com/cuda/cuda-programming-guide/
---

# CUDA Core / CUDA 核心

## 定义

**CUDA Core** 是 NVIDIA GPU 中执行**标量浮点/整数运算**的基本计算单元，每个 CUDA Core 每个时钟周期可完成一次 FP32 或 INT32 运算。

## 官方原文

> "Each SM contains a set of CUDA Cores (also called streaming processors) that execute floating-point and integer instructions."

## 与 Tensor Core 的区别

| | CUDA Core | [[Tensor-Core]] |
|---|---|---|
| **运算类型** | 标量（scalar） | 矩阵（matrix） |
| **适用场景** | 通用计算、向量运算 | GEMM、深度学习 |
| **引入版本** | Tesla（2006） | Volta（2017） |
| **吞吐** | 1 op/cycle | 矩阵块/cycle |

## 每代 SM 的 CUDA Core 数量

| 架构 | CUDA Cores / SM |
|------|----------------|
| Volta | 64 |
| Turing | 64 |
| Ampere | 128 |
| Hopper | 128 |
| Blackwell | 128 |

## 相关词条

- [[SM]] — CUDA Core 所在的硬件单元
- [[warp]] — CUDA Core 以 warp 为单位执行指令
- [[Tensor-Core]] — 专用矩阵运算单元，与 CUDA Core 并列
