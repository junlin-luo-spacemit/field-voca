---
tags: [nvidia, cuda, performance, atom]
domain: cuda
aliases: [occupancy, 占用率]
status: approved
created: 2026-08-07
updated: 2026-08-07
source: https://docs.nvidia.com/cuda/cuda-programming-guide/
superseded_by:
---

# Occupancy / 占用率

## 定义

**Occupancy** 是 [[SM]] 上**活跃 warp 数**与该 SM 最大可支持 warp 数的比值（0~100%），是衡量 GPU 利用率的核心指标。

## 官方原文

> "Higher occupancy helps hide memory latency by keeping the SM busy."

## 为什么重要

- 高 occupancy → 更多 warp 并发 → 更强的 [[latency hiding]] 能力 → 隐藏内存访问延迟
- 低 occupancy → SM 空闲周期增多 → 性能下降
- **但并非越高越好**：某些 kernel 受限于计算吞吐而非内存，此时牺牲 occupancy 换取更多寄存器/shared memory 反而更快

## 影响因素

| 资源 | 限制方式 |
|------|----------|
| **寄存器用量** | 每线程寄存器数 × block 大小 ≤ SM 寄存器堆总量 |
| **Shared memory 用量** | 每 block 的 shared memory ≤ SM 可用 shared memory |
| **Block 大小** | 线程数必须是 warp 大小（32）的倍数，最大 1024 |
| **SM 最大 block 数** | 每个 SM 最多同时驻留的 block 数（架构相关，如 Hopper = 32） |

## 计算公式

```
Occupancy = 实际活跃 warp 数 / SM 最大 warp 数
```

- Hopper H100：最大 64 warps/SM
- 若一个 block = 256 线程 = 8 warps，且每 SM 驻留 8 个 block，则 occupancy = 64/64 = 100%

## 优化工具

- **Nsight Compute**：报告实际 occupancy 和限制因素
- **Occupancy Calculator**（CUDA Toolkit 内置 Excel/API）：预估给定配置的 occupancy
- **`cudaOccupancyMaxActiveBlocksPerMultiprocessor`**：运行时查询 API

## 优化策略

1. **减少寄存器用量**：`-maxrregcount=N` 或 `__launch_bounds__(maxTPB, minBPSM)`
2. **减少 shared memory**：动态分配改静态，或减少 per-block 用量
3. **调整 block 大小**：通常 128/256 是平衡点
4. **权衡取舍**：计算密集型 kernel 可接受低 occupancy

## 相关词条

- [[SM]] — occupancy 的计算单元
- [[warp]] — occupancy 的计量单位
- [[latency hiding]] — 高 occupancy 的核心收益
- [[shared memory]] — 限制 occupancy 的资源之一
- [[register file]] — 限制 occupancy 的另一资源
