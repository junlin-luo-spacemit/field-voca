---
tags: [nvidia, cuda, performance, atom]
domain: cuda
aliases: [latency hiding, 延迟隐藏]
status: approved
created: 2026-08-07
updated: 2026-08-07
source: https://docs.nvidia.com/cuda/cuda-programming-guide/
superseded_by:
---

# Latency Hiding / 延迟隐藏

## 定义

**Latency Hiding** 是 GPU 通过高**线程级并行（TLP）**掩盖内存或指令延迟的核心机制。当一个 [[warp]] 因访问内存而停顿时，[[SM]] 的 warp 调度器立即切换到另一个就绪 warp 继续执行，从而保持计算单元持续忙碌。

## 官方原文

> "GPU通过高TLP（大量活跃warp）掩盖内存或指令延迟的机制，与CPU的深缓存/乱序执行策略不同。"

## 工作原理

```
时间轴：
Cycle 1: Warp 0 发起 global memory 读取（需要 ~400 cycles）
Cycle 2: Warp 0 停顿，调度器切换到 Warp 1 执行
Cycle 3: Warp 1 执行计算指令
...
Cycle 50: Warp 1 停顿，切换到 Warp 2
...
Cycle 401: Warp 0 的数据到达，继续执行

核心思想：只要有足够多的就绪 warp，SM 永远不会空闲等待
```

## 与 CPU 的对比

| | GPU（Latency Hiding） | CPU（Latency Reduction） |
|---|---|---|
| **策略** | 通过大量线程切换隐藏延迟 | 通过缓存和乱序执行减少延迟 |
| **并发单位** | 数千线程/SM | 少量线程/核心 |
| **缓存** | 较小（L1/L2 相对算力） | 巨大的 L3 缓存 |
| **切换开销** | 零开销（硬件多上下文） | 有开销（需保存/恢复寄存器） |
| **适用场景** | 高吞吐、高延迟容忍 | 低延迟、串行任务 |

## 延迟来源

| 延迟类型 | 典型周期数 | 隐藏所需 warp 数 |
|---------|-----------|-----------------|
| 寄存器依赖 | ~4 cycles | 少量 |
| [[shared memory]] 访问 | ~20-30 cycles | 中等 |
| L2 cache 命中 | ~200 cycles | 较多 |
| [[HBM]] (global memory) | ~400-800 cycles | 大量 |

## 隐藏延迟的条件

### 1. 高 Occupancy

- [[occupancy]] 越高 → 活跃 warp 越多 → 调度器选择余地越大
- 但并非越高越好：计算密集型 kernel 受益小

### 2. 充足的指令级并行（ILP）

```cuda
// ❌ 低 ILP：每条指令依赖前一条
float a = data[i];
float b = a * 2;
float c = b + 1;
result[i] = c;

// ✅ 高 ILP：多个独立指令流
float a = data[i];
float b = data[i+1];
float c = data[i+2];
// a, b, c 三条加载可流水线并发
```

### 3. 访存模式优化

- **Coalesced access**：warp 内线程访问连续地址，合并为少量事务
- **Bank conflict 消除**：避免 [[shared memory]] 访问串行化
- **Prefetching**：`cudaMemPrefetchAsync` 提前搬运数据

## Little's Law 推导

```
所需 warp 数 ≈ (延迟 × 吞吐) / (每 warp 指令数)

示例：
- Global memory 延迟 = 400 cycles
- SM 吞吐 = 每 cycle 发射 1 条指令
- 每 warp 平均 10 条独立指令

所需 warp 数 ≈ 400 / 10 = 40 warps
```

## 检测工具

- **Nsight Compute**：
  - `Warp Stall Reasons`：显示 warp 停顿原因分布（memory、sync、exec dependency）
  - `SM Busy` vs `Memory Busy`：判断是否受内存延迟限制
- **nvprof**：`stall_*` metrics

## 相关词条

- [[warp]] — 延迟隐藏的调度单位
- [[SM]] — 执行 warp 切换的硬件
- [[occupancy]] — 高 occupancy 提供更多 warp 供调度
- [[HBM]] — 主要的延迟来源
- [[shared memory]] — 低延迟数据复用手段
- TLP（Thread-Level Parallelism）— 延迟隐藏的基础
