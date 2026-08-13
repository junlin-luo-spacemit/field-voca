---
tags: [nvidia, cuda, hardware, atom]
domain: cuda
aliases: [Streaming Multiprocessor, 流多处理器]
status: approved
created: 2026-06-29
updated: 2026-06-29
source: https://docs.nvidia.com/cuda/cuda-programming-guide/
superseded_by:
---

# SM — Streaming Multiprocessor / 流多处理器

## 定义

**SM**（Streaming Multiprocessor）是 GPU 的基本计算单元。每个 SM 包含多个 [[CUDA-Core]]、[[Tensor-Core]]、[[warp]] 调度器、[[register-file]] 和 [[shared-memory]]。

## 官方原文

> "The NVIDIA GPU architecture is built around a scalable array of multithreaded Streaming Multiprocessors (SMs)."

## 关键参数（H100 SXM5）

| 参数 | 数值 |
|------|------|
| SM 总数 | 132 |
| 每 SM CUDA Core | 128（FP32） |
| 每 SM Tensor Core | 4（第4代） |
| 每 SM 共享内存 | 最大 228 KB |
| 每 SM 寄存器 | 65536 × 32-bit |
| 每 SM 最大线程 | 2048 |

## 工作原理

- SM 以 [[warp]]（32线程）为调度单位执行指令
- 多个 warp 并发驻留在同一 SM，通过 [[latency-hiding]] 隐藏延迟
- [[occupancy]] 描述 SM 上实际活跃 warp 数与最大值的比率

## 相关词条

- [[warp]] — SM 的调度单位
- [[CUDA-Core]] — SM 内的标量计算单元
- [[Tensor-Core]] — SM 内的矩阵计算单元
- [[shared-memory]] — SM 内部的快速暂存内存
- [[occupancy]] — SM 利用率指标
