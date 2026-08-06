---
tags: [nvidia, cuda, memory, atom]
domain: cuda
aliases: [Unified Memory, 统一内存, UM]
date: 2026-06-29
source: https://docs.nvidia.com/cuda/cuda-programming-guide/
---

# Unified Memory / 统一内存

## 定义

**Unified Memory** 是 CUDA 提供的编程模型特性，在 CPU 和 GPU 之间创建一个**统一的内存地址空间**，数据由 CUDA runtime 自动在 host 和 device 间迁移，无需手动 `cudaMemcpy`。

## 官方原文

> "Unified Memory creates a pool of managed memory that is shared between the CPU and GPU, bridging the CPU-GPU divide."

## 工作原理

```
CPU (Host)                  GPU (Device)
+------------+              +------------+
|  System    |  ←自动迁移→  |   HBM      |
|  DRAM      |    page      |  (VRAM)    |
+------------+   fault      +------------+
       ↑                          ↑
       └──── 统一虚拟地址空间 ─────┘
```

## 使用方式

```c
// 传统方式
float *d_arr;
cudaMalloc(&d_arr, size);
cudaMemcpy(d_arr, h_arr, size, cudaMemcpyHostToDevice);

// Unified Memory 方式
float *arr;
cudaMallocManaged(&arr, size);   // CPU/GPU 均可直接访问
```

## 关键特性

| 特性 | 说明 |
|------|------|
| **cudaMallocManaged** | 分配托管内存的 API |
| **Page migration** | 按需将内存页迁移到访问方 |
| **Prefetching** | `cudaMemPrefetchAsync` 主动预迁移 |
| **Oversubscription** | Pascal+ 支持 GPU 内存超额分配 |

## 优缺点

**优点**：代码简化，无需手动管理数据移动  
**缺点**：page fault 迁移有延迟，性能不及显式管理

## 相关词条

- [[HBM]] — GPU 侧物理显存
- [[SM]] — 执行访问统一内存的 kernel
- cudaMalloc — 传统显存分配 API（对比）
