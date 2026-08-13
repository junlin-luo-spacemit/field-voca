---
tags: [nvidia, cuda, hardware, atom]
domain: cuda
aliases: [register file, 寄存器堆, RF]
status: approved
created: 2026-08-07
updated: 2026-08-07
source: https://docs.nvidia.com/cuda/cuda-programming-guide/
superseded_by:
---

# Register File / 寄存器堆

## 定义

**Register File** 是每个 [[SM]] 上的**寄存器总资源池**，由该 SM 上所有活跃线程共享分配。每个线程从中分配私有寄存器，寄存器不足时会触发 **register spilling**（溢出到 local memory，性能骤降）。

## 官方原文

> "The register file stores thread local variables which are usually allocated by the compiler."

## 硬件规格

| 架构 | 寄存器堆大小/SM | 每线程最大寄存器 |
|------|----------------|-----------------|
| Volta (V100) | 65,536 × 32-bit | 255 |
| Ampere (A100) | 65,536 × 32-bit | 255 |
| Hopper (H100) | 65,536 × 32-bit | 255 |
| Blackwell (B200) | 65,536 × 32-bit | 255 |

## 工作原理

```
一个 SM 的寄存器堆 = 65,536 个 32-bit 寄存器

假设启动配置：
- Block size = 256 线程
- 每线程使用 32 个寄存器
- 每 block 需要 256 × 32 = 8,192 个寄存器

该 SM 最多驻留：65,536 / 8,192 = 8 个 blocks
→ 如果该 SM 支持更多 blocks，但寄存器不足，occupancy 受限
```

## Register Spilling（寄存器溢出）

- 当每线程寄存器需求超出可分配量时，编译器将部分变量**溢出到 local memory**
- Local memory 物理上是 [[HBM]]，延迟数百 cycles，严重拖累性能
- **Nsight Compute** 报告指标：`lmem_load/store` transactions

### 检测溢出

```bash
# 编译时查看寄存器用量
nvcc -Xptxas -v kernel.cu

# 输出示例：
# ptxas info : Used 64 registers, 0+48 bytes lmem
#                    ^^               ^^^^ 有溢出！
```

## 优化策略

### 1. 限制寄存器用量（可能降低性能）

```cuda
// 方法 A：编译时全局限制
nvcc -maxrregcount=32 kernel.cu

// 方法 B：per-kernel 限制（推荐）
__global__ void __launch_bounds__(256, 4) kernel() {
    // 256 = 最大线程数/block
    // 4 = 期望最小 blocks/SM（暗示寄存器预算）
}
```

### 2. 减少局部变量

- 合并临时变量
- 用 `const` 让编译器内联常量
- 避免大型局部数组（改用 [[shared memory]]）

### 3. 权衡取舍

- 某些算法需要大量寄存器才能高效（如复杂数学运算）
- 此时牺牲 [[occupancy]] 换取更多寄存器可能反而更快

## 与 Occupancy 的关系

```
寄存器限制的 occupancy = 
    (SM 寄存器总数) / (每线程寄存器数 × block 大小)
```

- 每线程 32 寄存器、block 256 线程 → 最多 8 blocks/SM（100% occupancy）
- 每线程 64 寄存器、block 256 线程 → 最多 4 blocks/SM（50% occupancy）

## 相关词条

- [[SM]] — register file 所在的硬件单元
- [[occupancy]] — 寄存器用量是 occupancy 的主要限制因素
- [[shared memory]] — 与寄存器堆并列的片上资源
- [[warp]] — 寄存器分配以 warp 粒度管理
- Local Memory — 寄存器溢出的目标（实际是 global memory）
