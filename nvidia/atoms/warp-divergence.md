---
tags: [nvidia, cuda, performance, atom]
domain: cuda
aliases: [warp divergence, warp 发散, 分支发散, branch divergence]
status: approved
created: 2026-08-07
updated: 2026-08-07
source: https://docs.nvidia.com/cuda/cuda-programming-guide/
superseded_by:
---

# Warp Divergence / Warp 发散

## 定义

**Warp Divergence** 是指同一 [[warp]] 内的 32 个线程因条件分支（if/else/switch）走不同执行路径，导致 GPU 必须**串行执行**各条路径，未进入当前路径的线程被 **masked off**（掩蔽），功能单元空闲。

## 官方原文

> "When different threads in a warp follow different code paths, this is sometimes called warp divergence. GPU utilization is maximized when threads within a warp follow the same control flow path."

## 性能影响

| 场景 | 活跃线程数 | 有效吞吐 |
|------|-----------|---------|
| 无分支 | 32/32 | 100% |
| 50% 线程走 if 分支 | 16/32（两次串行执行） | 50% |
| 最坏情况（每线程不同路径） | 1/32 | ~3% |

## 示例

```cuda
__global__ void divergent_kernel(int *data) {
    int idx = threadIdx.x;
    if (idx % 2 == 0) {
        // 偶数线程走这里（分支 A）
        data[idx] *= 2;
    } else {
        // 奇数线程走这里（分支 B）
        data[idx] += 1;
    }
    // GPU 执行流程：
    // 1. 掩蔽奇数线程，执行分支 A（16 线程活跃）
    // 2. 掩蔽偶数线程，执行分支 B（16 线程活跃）
    // 实际耗时 = 分支 A 时间 + 分支 B 时间
}
```

## 优化策略

### 1. 按 warp 边界对齐条件

```cuda
// ❌ 差：每个 warp 内都有分支
if (threadIdx.x % 2 == 0) { ... }

// ✅ 好：整个 warp 统一路径
if (threadIdx.x < 32) { ... }
```

### 2. Branch Predication（编译器自动）

- 对于**极短分支**（<= 几条指令），编译器将 if-else 转为 predicated 指令
- 所有路径的指令都调度，但 false predicate 的指令不写回结果
- 避免实际分支跳转开销，但**两条路径指令都会执行**

### 3. 循环展开 `#pragma unroll`

```cuda
#pragma unroll
for (int i = 0; i < 4; i++) {
    if (i == target) { ... }  // 展开后可能被优化掉
}
```

## 检测工具

- **Nsight Compute**：`Branch Efficiency` 指标（100% = 无发散）
- **nvprof**：`branch_efficiency` metric

## 相关词条

- [[warp]] — 发散的基本单位
- [[SM]] — 执行分支的硬件
- [[occupancy]] — 发散会降低有效 occupancy
- SIMT — Single Instruction, Multiple Threads 执行模型
