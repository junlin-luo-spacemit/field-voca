---
tags: [nvidia, cuda, hardware, atom]
domain: cuda
aliases: [warp, 线程束]
date: 2026-06-29
source: https://docs.nvidia.com/cuda/cuda-programming-guide/
---

# warp / 线程束

## 定义

**warp** 是 GPU 的基本执行单位，由 **32 个线程**组成。[[SM]] 上的 warp 调度器每个时钟周期选择一个就绪 warp 发射指令，所有32个线程执行相同指令（SIMT 模型）。

## 官方原文

> "The multiprocessor creates, manages, schedules, and executes threads in groups of 32 parallel threads called warps."

## 关键数字

| 参数 | 数值 |
|------|------|
| warp 大小 | **32 线程**（固定不变） |
| warp lane | 线程在 warp 内的编号，0–31 |
| 每 SM 最大 warp 数 | 64（Hopper） |

## SIMT 执行模型

- warp 内所有线程在同一时刻执行同一条指令
- 分支（if/else）导致 [[warp divergence]]，部分线程被 masked off，串行执行两条路径
- 同一 warp 内线程可通过 shuffle 指令直接交换寄存器数据（无需共享内存）

## warp 与线程层次关系

```
Grid
└── Block（最大1024线程）
    └── warp（32线程）× N
        └── thread（1个）× 32
```

## 相关词条

- [[SM]] — warp 运行的硬件单元
- [[warp divergence]] — 分支导致的性能问题
- [[occupancy]] — SM 上活跃 warp 数量比率
- [[latency hiding]] — 多 warp 并发隐藏访存延迟
