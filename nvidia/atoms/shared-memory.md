---
tags: [nvidia, cuda, memory, atom]
domain: cuda
aliases: [shared memory, 共享内存, SMEM, __shared__]
status: approved
created: 2026-08-07
updated: 2026-08-07
source: https://docs.nvidia.com/cuda/cuda-programming-guide/
superseded_by:
---

# Shared Memory / 共享内存

## 定义

**Shared Memory** 是每个 [[SM]] 上的**片上可编程内存**（on-chip SRAM），同一 thread block 内的所有线程共享，延迟极低（~20-30 cycles，远快于 [[HBM]] 的数百 cycles），是优化 CUDA kernel 的核心手段。

## 官方原文

> "Shared memory can be used for exchanging data between threads of a thread block or cluster."

## 硬件特性

| 参数 | 数值（Hopper H100） |
|------|---------------------|
| 最大容量/SM | 228 KB |
| 最大容量/Block | 227 KB（需 opt-in） |
| Bank 数 | 32（每 bank 4 字节/周期） |
| 延迟 | ~20-30 cycles |
| 与 L1 cache 关系 | 共享 **unified data cache** 物理资源 |

## 使用方式

### 静态分配

```cuda
__global__ void kernel() {
    __shared__ float tile[256];  // 声明在 block 内共享的数组
    tile[threadIdx.x] = ...;
    __syncthreads();              // 同步后再读取其他线程写入的数据
}
```

### 动态分配

```cuda
extern __shared__ float dynamic_smem[];  // 声明外部共享内存

// 启动时指定大小
kernel<<<grid, block, 48*1024>>>();  // 第3个参数 = 动态 SMEM 字节数
```

## 典型应用场景

1. **矩阵乘法分块**：将 global memory 数据加载到 shared memory tile，复用减少带宽
2. **规约（reduction）**：warp 内先用 shuffle，block 内用 shared memory 聚合
3. **转置**：读取连续，写入跨步 → 用 shared memory 缓存后按连续模式写回
4. **卷积/stencil**：邻居数据复用，halo 区域共享

## Bank Conflict

- Shared memory 分为 **32 个 bank**，连续 32-bit 地址依次映射到 bank 0~31
- 同一 warp 内多线程访问**同一 bank 的不同地址**时，访问串行化（降至 1/N 带宽）
- **广播例外**：所有线程读同一地址时无冲突

### 消除 conflict 的技巧

```cuda
// ❌ 有冲突：转置时列访问
__shared__ float tile[32][32];
tile[threadIdx.y][threadIdx.x] = ...;  // 写入无冲突
value = tile[threadIdx.x][threadIdx.y];  // 读取有冲突（同列 = 同 bank）

// ✅ 无冲突：padding 列维度
__shared__ float tile[32][33];  // 多 1 列，bank 映射错开
```

## Unified Data Cache 配置

Shared memory 与 L1 cache 共享同一物理 SRAM，比例可调：

```cuda
// 最大化 shared memory（适合 SMEM 密集 kernel）
cudaFuncSetAttribute(kernel, 
    cudaFuncAttributePreferredSharedMemoryCarveout,
    cudaSharedmemCarveoutMaxShared);

// 最大化 L1 cache（适合 global memory 密集 kernel）
cudaFuncSetAttribute(kernel, 
    cudaFuncAttributePreferredSharedMemoryCarveout,
    cudaSharedmemCarveoutMaxL1);
```

## 相关词条

- [[SM]] — shared memory 所在的硬件单元
- [[warp]] — shared memory 访问的 bank conflict 以 warp 为单位
- [[HBM]] — shared memory 是其片上缓存层
- [[occupancy]] — shared memory 用量影响 occupancy
- Distributed Shared Memory — Hopper+ cluster 内跨 block 的扩展
