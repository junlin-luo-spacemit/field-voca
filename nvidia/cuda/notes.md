---
tags: [nvidia, cuda, notes]
domain: cuda
last_updated: 2026-06-29
source: https://docs.nvidia.com/cuda/cuda-programming-guide/
---

# CUDA 学习笔记

> 文档入口：https://docs.nvidia.com/cuda/cuda-programming-guide/index.html
> 最后更新：2026-06-29（基于官方文档 May 27, 2026 版本）

---

## 文档结构概览

NVIDIA CUDA 文档体系分为以下几类：

| 文档 | 说明 | 链接 |
|------|------|------|
| **CUDA C++ Programming Guide** | 核心参考，覆盖编程模型、内存、执行配置（5部分） | [链接](https://docs.nvidia.com/cuda/cuda-programming-guide/index.html) |
| **CUDA C++ Best Practices Guide** | 性能优化实践指南 | [链接](https://docs.nvidia.com/cuda/cuda-c-best-practices-guide/index.html) |
| **CUDA Runtime API** | C Runtime API 参考手册 | [链接](https://docs.nvidia.com/cuda/cuda-runtime-api/index.html) |
| **CUDA Driver API** | 更底层的驱动层 API | [链接](https://docs.nvidia.com/cuda/cuda-driver-api/index.html) |
| **PTX ISA** | 并行线程执行指令集架构（汇编级） | [链接](https://docs.nvidia.com/cuda/parallel-thread-execution/index.html) |
| **CUDA Math API** | 数学函数库参考 | — |
| **Toolkit Release Notes** | 版本更新说明 | — |

### CUDA Programming Guide 五大部分

| 部分 | 内容 | 适合人群 |
|------|------|----------|
| **Part 1** | Introduction & Programming Model（语言无关概念） | 所有人，包括非开发者 |
| **Part 2** | Programming GPUs in CUDA（C++ & Python 基础） | 入门开发者 |
| **Part 3** | Advanced CUDA（多 GPU、精细控制） | 有经验的开发者 |
| **Part 4** | CUDA Features（CUDA Graphs、Dynamic Parallelism、Unified Memory等） | 需要特定功能时查阅 |
| **Part 5** | Technical Appendices（规格参考） | 技术参考 |

---

## 关键概念速记（来自 Part 1 Programming Model）

### 异构系统（Heterogeneous System）
- **host**：CPU 及其直连内存（system memory / host memory）
- **device**：GPU 及其直连内存（device memory / global memory）
- 应用始终从 CPU 启动，host code 通过 CUDA API 控制 GPU
- CPU 和 GPU **可以同时执行**，最优性能需最大化两者利用率

### 执行层次（从大到小）
```
Grid（网格）
  └── Thread Block Cluster（线程块簇，Compute Capability 9.0+，即 Hopper 起）
          └── Thread Block（线程块，单 SM 内执行，最多 1024 线程）
                  └── Warp（线程束，32 threads，SIMT 执行单元）
                          └── Thread（线程，最小执行单元）
```

**关键约束**（官方原文）：
- 同一 block 内所有线程运行在**同一个 SM** 上，可共享 shared memory 和同步
- 不同 block 之间**无数据依赖保证**，可以任意顺序调度
- Thread Block Cluster（Hopper+）中的 blocks 在同一个 **GPC**（Graphics Processing Cluster）内并发执行，可访问 **distributed shared memory**

### GPU 硬件模型
```
GPU
└── GPC (Graphics Processing Cluster)  ← 多个
        └── SM (Streaming Multiprocessor)  ← 多个
                ├── Register File（寄存器堆，线程私有）
                ├── Unified Data Cache
                │     ├── L1 Cache（可配置比例）
                │     └── Shared Memory（可配置比例，block 内共享）
                ├── Constant Cache（缓存 constant memory）
                └── Functional Units（CUDA Core, Tensor Core, RT Core 等）
L2 Cache（所有 SM 共享）
Global Memory / HBM（最大，延迟最高）
```

### 内存速度排序（从快到慢）
```
Register（线程私有）
  > Shared Memory / L1 Cache（block 内共享，片上）
  > Constant Cache（只读广播，片上缓存）
  > L2 Cache（全 GPU 共享，片上）
  > Global Memory / HBM（片外 DRAM，容量最大，延迟最高）
```

### 两种编程模型对比（CUDA 2026 新特性）

| 维度 | SIMT 编程模型 | Tile 编程模型（新） |
|------|--------------|--------------------|
| 代码粒度 | 每个**线程**写代码 | 每个**block**写代码 |
| 线程控制 | 开发者显式控制 | 编译器自动映射到线程 |
| warp 发散 | 存在（需注意） | 无概念（block 统一控制流） |
| 跨架构迁移 | 可能需调整 | 编译器自动适配不同架构 |
| 数据单元 | 标量/数组 | **tile**（多维数据块，不可变） |
| 适用场景 | 精细控制、特定优化 | 矩阵运算、简化开发 |

### 常用修饰符
| 修饰符 | 说明 |
|--------|------|
| `__global__` | kernel 函数，从 host 调用，在 device 执行 |
| `__device__` | 只能在 device 上调用和执行 |
| `__host__` | 只在 host 上执行（默认） |
| `__shared__` | 声明 shared memory 变量（block 内共享，片上） |
| `__constant__` | 声明 constant memory 变量（只读，64KB，带广播缓存） |
| `__managed__` | 声明 unified memory 变量（CPU/GPU 均可访问） |

---

## 重要数字

| 参数 | 值 |
|------|----|
| Warp 大小 | 32 threads |
| Block 最大线程数 | 1024 |
| SM shared memory（Hopper） | 最大 228 KB |
| H100 SXM SM 数量 | 132 |
| H100 SXM 内存带宽 | 3.35 TB/s |
| H100 FP16 Tensor Core 算力 | 989 TFLOPS |

---

## 重要数字（官方规格）

| 参数 | 值 | 备注 |
|------|----|------|
| Warp 大小 | **32 threads** | 所有架构固定 |
| Block 最大线程数 | **1024** | 所有架构 |
| Block 线程数建议 | **32 的倍数** | 非倍数时最后一个 warp 有空闲 lane |
| Cluster 适用架构 | Compute Capability **9.0+**（Hopper） | |
| SM shared memory（Hopper H100） | 最大 **228 KB** | unified data cache 可配置 |
| H100 SXM SM 数量 | **132** | |
| H100 SXM 内存带宽 | **3.35 TB/s** | HBM3 |
| H100 FP16 Tensor Core 算力 | **989 TFLOPS** | |

---

## 常用修饰符

| 修饰符 | 说明 |
|--------|------|
| `__global__` | kernel 函数，从 host 调用，在 device 执行 |
| `__device__` | 只能在 device 上调用和执行 |
| `__host__` | 只在 host 上执行（默认） |
| `__shared__` | 声明 shared memory 变量（block 内共享） |
| `__constant__` | 声明 constant memory 变量（只读，带广播缓存） |

---

## CUDA 生态（GPU 编程的多种路径）

| 方式 | 示例 | 抽象层次 |
|------|------|----------|
| 专用库 | cuBLAS, cuFFT, cuDNN, CUTLASS | 最高（无需写 kernel）|
| AI 框架 | PyTorch, TensorFlow（底层调库） | 高 |
| DSL | NVIDIA Warp, OpenAI Triton | 中（编译到 CUDA）|
| CUDA C++ / Python | 本指南内容 | 中低 |
| PTX / SASS | GPU 汇编 | 最低 |

---

## 待深入阅读

- [x] Part 1: Introduction & Programming Model（2026-06-29 完成）
- [x] Part 2: Writing CUDA Kernels + Understanding Memory（2026-06-29 完成）
- [ ] Part 3: Advanced CUDA（多 GPU、CUDA Graphs、Dynamic Parallelism）
- [ ] Part 4: CUDA Features（完整功能参考）
- [ ] CUDA C++ Best Practices Guide - Memory Optimizations
- [ ] Hopper Architecture Whitepaper（H100 架构白皮书）
- [ ] Compute Capabilities Appendix（各架构详细规格表）

---

## Part 2 速查：内存类型完整对比

> 来源：官方 Table 1（2.3.3 GPU Device Memory Spaces）

| 内存类型 | 作用域 | 生命周期 | 物理位置 | 延迟 | 容量 |
|----------|--------|---------|---------|------|------|
| **Register** | Thread | Kernel | SM 片上 | ~1 cycle | 极小（每线程有限） |
| **Shared** | Block | Kernel | SM 片上 | ~5–30 cycle | 较小（Hopper 最大 228 KB/SM） |
| **Local** | Thread | Kernel | Device DRAM | ~400–800 cycle | 逻辑无限（实际受显存限制） |
| **Constant** | Grid（只读） | Application | Device DRAM + 专用缓存 | 命中缓存 ~1 cycle，未命中同 global | 64 KB |
| **Global** | Grid | Application | Device DRAM | ~400–800 cycle | 最大（GB 级，如 H100 80 GB） |

**Local memory 注意**：名字叫 "local"（逻辑局部），实际存在 **global memory（DRAM）** 上，延迟与 global memory 相同。触发条件：register spilling、大数组、无法静态确定索引的数组。

---

## Part 2 速查：Memory Coalescing 规则

> 来源：2.3.4.1 Coalesced Global Memory Access

Global memory 以 **32-byte 事务**为单位传输。一次 warp load 指令（32 线程 × 4 bytes = 128 bytes）最优需 4 次事务：

```
最优（100% 利用率）：
  Thread 0 → addr N+0
  Thread 1 → addr N+4    → 4 次 32-byte 事务，传输 128B，使用 128B
  Thread 2 → addr N+8
  ...（连续地址）

最差（12.5% 利用率）：
  Thread 0 → addr 0
  Thread 1 → addr 32     → 32 次事务，传输 1024B，使用 128B
  Thread 2 → addr 64
  ...（每线程间隔 ≥ 32 bytes）
```

**实践规则**：
- ✅ `threadIdx.x` 对应内存地址最低维（行主序，最后索引移动最快）
- ✅ `vecAdd` 类 kernel（workIndex = threadIdx.x + blockIdx.x\*blockDim.x）天然 coalesced
- ❌ 矩阵转置的**写操作**（按列写）天然不 coalesced → 用 shared memory 暂存后改为行主序写

**最优 coalescing 本质**：最大化 `bytes_used / bytes_transferred` 比值。

---

## Part 2 速查：Shared Memory Bank Conflict 规则

> 来源：2.3.4.2 Shared Memory Access Patterns

Shared memory 分 **32 个 bank**，连续 32-bit 字依次映射（bank = (word_index) % 32）：

| 访问模式 | 结果 |
|----------|------|
| 同 warp 各线程访问不同 bank | ✅ 无冲突，并发执行 |
| 同 warp 多线程访问同 bank **相同地址** | ✅ 广播，无冲突 |
| 同 warp 多线程访问同 bank **不同地址** | ❌ Bank conflict，串行化 |

**典型 bank conflict 场景**：`__shared__ float arr[32][32]`，按列访问（stride = 32 elements = 32 banks）→ 32-way bank conflict。

**修复方法**：加一列 padding：
```cpp
__shared__ float arr[32][33];  // padding 使每行错开，消除 bank conflict
```

**矩阵转置最优写法（两个性能点同时解决）**：
```cpp
// 读：coalesced（按行读 global memory）
smemArray[threadIdx.x][threadIdx.y] = a[row * m + col];
__syncthreads();
// 写：coalesced（按行写 global memory）+ padding 消除 bank conflict
c[col2 * m + row2] = smemArray[threadIdx.y][threadIdx.x];
```

---

## Part 2 速查：CUDA Streams 使用模式

> 来源：2.5 Asynchronous Execution

### 基本生命周期
```
cudaStreamCreate(&stream)
  → kernel<<<g,b,0,stream>>>() 或 cudaMemcpyAsync(..., stream)
  → cudaStreamSynchronize(stream)   // 等待完成
  → cudaStreamDestroy(stream)
```

### 多 Stream 并发模板（计算与传输重叠）
```
Stream 1:  [H→D copy] [kernel A] [D→H copy]
Stream 2:         [H→D copy] [kernel B] [D→H copy]
                        ↑ 时间轴：重叠执行
```
要求：H→D/D→H 的 host 内存必须是 **pinned memory**（`cudaMallocHost`），否则退化为同步拷贝。

### 同步粒度选择
| 函数 | 粒度 | 说明 |
|------|------|------|
| `cudaDeviceSynchronize()` | 全 GPU | 等待所有 stream 完成，开销大 |
| `cudaStreamSynchronize(s)` | 单 stream | 等待指定 stream 完成 |
| `cudaEventSynchronize(e)` | 单 event | 等待特定操作点完成，粒度最细 |
| `cudaStreamQuery(s)` | 单 stream | 非阻塞轮询 |
| `cudaEventQuery(e)` | 单 event | 非阻塞轮询 |

### 跨 Stream 依赖（Event 桥接）
```cpp
// Stream 1 完成 kernel1 后，stream 2 才能执行 kernel2
cudaEventRecord(event, stream1);          // 在 stream1 中记录 event
cudaStreamWaitEvent(stream2, event, 0);  // stream2 等待该 event
kernel2<<<g, b, 0, stream2>>>();         // stream2 继续执行
```

---

## Part 2 速查：CUDA Graphs 使用场景与流程

> 来源：2.5.9.2 Introduction to CUDA Graphs with Stream Capture

**适用场景**：相同 kernel 序列需重复执行数百/数千次（如 inference loop、训练 step）。  
**收益**：消除每次 CPU launch overhead（每个 API 调用约数微秒），使 GPU 持续满负荷。

**三步流程**：
```
① Capture（一次）
   cudaStreamBeginCapture(stream, cudaStreamCaptureModeGlobal)
   [提交 kernel / memcpy 到 stream]
   cudaStreamEndCapture(stream, &graph)

② Instantiate（一次）
   cudaGraphInstantiate(&instance, graph, NULL, NULL, 0)

③ Launch（重复多次）
   cudaGraphLaunch(instance, stream)
   cudaStreamSynchronize(stream)
```

---

## Part 2 速查：Error Handling 最佳实践

> 来源：2.1.7 Error Checking in CUDA

```cpp
// 1. kernel launch 后立即检查启动参数错误
kernel<<<blocks, threads>>>(args...);
CUDA_CHECK(cudaGetLastError());       // 捕获 launch config 错误

// 2. 同步后检查异步执行错误
CUDA_CHECK(cudaDeviceSynchronize());  // 捕获 kernel 运行时错误

// 3. 诊断技巧：设置环境变量
//    CUDA_LAUNCH_BLOCKING=1  → 每次 launch 后同步（定位出错 kernel）
//    CUDA_LOG_FILE=cuda.log  → 驱动错误写入文件（需 r570+ 驱动）
```

**注意**：`cudaGetLastError` 会**清除**错误状态；`cudaPeekAtLastError` **不清除**。异步错误（kernel 崩溃）只会在下一次 CUDA API 调用时才被报告。

---

## Part 2 速查：Occupancy 计算示例

> 来源：2.3.7 Kernel Launch and Occupancy（以 Compute Capability 10.0 为例）

**定义**：`occupancy = active warps per SM / max warps per SM`

SM 资源限制（CC 10.0 示例）：
| 参数 | 值 |
|------|----|
| maxBlocksPerMultiProcessor | 32 |
| maxThreadsPerMultiProcessor | 2048 |
| sharedMemPerMultiprocessor | 228 KB |
| regsPerMultiprocessor | 65536 |

**示例 1**：`kernel<<<512, 768>>>()`
- 每 SM 最多 2 个 block（768×2 = 1536 ≤ 2048）
- Occupancy = (768×2) / 2048 = **75%**

**示例 2**：`kernel<<<512, 32>>>()`  
- block 限制：最多 32 blocks/SM（maxBlocksPerMultiProcessor）
- 32 blocks × 32 threads = 1024 threads → Occupancy = 1024 / 2048 = **50%**

**调优思路**：threads/block 和 sharedMem/block 是程序员可控的主要旋钮；register 由编译器控制（可用 `--maxrregcount` 限制上限，代价是可能 spill）。

---

## Part 3 速查：Thread Block Cluster 使用规则

> 来源：3.1.2 Launching Clusters（CC 9.0+ Hopper / CC 10.0+ Blackwell）

**核心概念**：Cluster = 一组保证同时运行在同一 GPC 上的 thread blocks，支持跨 block 的 distributed shared memory 访问和 cluster 级同步。

**三种声明方式对比**：

| 方式 | 灵活性 | 语法 | <<<>>> 语义 |
|------|--------|------|------------|
| `__cluster_dims__(2,1,1)` | 编译期固定 | kernel 函数注解 | 第一维 = block 数 |
| `cudaLaunchKernelEx` + `cudaLaunchAttributeClusterDimension` | 运行时可变 | `cudaLaunchConfig_t` | 第一维 = block 数 |
| `__block_size__((1024,1,1),(2,1,1))` | 编译期固定 | kernel 函数注解 | 第一维 = **cluster 数** |

**规则**：
- grid 各维度必须是 cluster 对应维度的整数倍
- CC 10.0+ 可用 `cudaLaunchAttributePreferredClusterDimension` 指定"首选"更大 cluster（所有 blocks 以 minimum 或 preferred 尺寸执行，需 kernel 兼容两种情况）

---

## Part 3 速查：Programmatic Dependent Launch (PDL)

> 来源：3.1.4（CC 9.0+，需 `cudaLaunchKernelEx`）

**目的**：让 primary/secondary kernel 在同一 stream 中部分重叠执行，减少 launch overhead 并提高 SM 利用率。

```
时间轴（同一 stream）：
  [primary kernel ]─────────────────────────────►
             ↓ cudaTriggerProgrammaticLaunchCompletion()
                     [secondary kernel ──► cudaGridDependencySynchronize() ─► 依赖工作]
                      ↑ 独立工作在此先执行
```

**三步模板**：
```cpp
// 1. primary kernel 内：依赖数据写完后触发
__global__ void primary() {
    // ... 写入 secondary 依赖的数据 ...
    cudaTriggerProgrammaticLaunchCompletion();
    // ... 与 secondary 重叠的其余工作 ...
}

// 2. secondary kernel 内：独立工作后等待
__global__ void secondary() {
    // ... 独立工作 ...
    cudaGridDependencySynchronize();  // 等待 primary 完成依赖数据
    // ... 依赖工作 ...
}

// 3. 启动：secondary 需用 cudaLaunchKernelEx + 特殊属性
cudaLaunchAttribute attr[1];
attr[0].id = cudaLaunchAttributeProgrammaticStreamSerialization;
attr[0].val.programmaticStreamSerializationAllowed = 1;
cudaLaunchConfig_t cfg = {gridDim, blockDim, 0, stream, attr, 1};
primary<<<gridDim, blockDim, 0, stream>>>();
cudaLaunchKernelEx(&cfg, secondary);
```

---

## Part 3 速查：Stream 进阶规则

> 来源：3.1.3 More on Streams and Events

**隐式同步触发条件**（会导致不同 stream 意外串行，需避免）：
```
cudaMallocHost / cudaMalloc / cudaMemset（同设备内）
任何提交到 NULL stream 的操作（非 cudaStreamNonBlocking 流受影响）
L1/Shared memory 配置切换（cudaFuncSetCacheConfig 旧 API）
```

**非阻塞流**（推荐在多流并发时使用）：
```cpp
cudaStreamCreateWithFlags(&stream, cudaStreamNonBlocking);
// 不受 NULL stream 隐式同步影响
```

**Stream 优先级**：
```cpp
int lo, hi;
cudaDeviceGetStreamPriorityRange(&lo, &hi);  // hi <= lo，hi 为最高优先级
cudaStreamCreateWithPriority(&s_high, cudaStreamNonBlocking, hi);
cudaStreamCreateWithPriority(&s_low,  cudaStreamNonBlocking, lo);
// 优先级影响调度顺序，不抢占已运行任务
```

**无计时 Event（跨 stream 依赖首选）**：
```cpp
cudaEventCreateWithFlags(&event, cudaEventDisableTiming);
// 性能优于普通 event，仅用于依赖表达时应使用
```

---

## Part 3 速查：Thread Scope 与 Scoped Atomics

> 来源：3.2.3–3.2.4

**Thread Scope 选择原则**：使用最窄的作用域（性能从高到低）：
```
thread_scope_block  >  thread_scope_device  >  thread_scope_system
（block 最快，system 最慢，跨 GPU 时才用 system）
```

**Scoped Atomic 使用模式**：
```cpp
// 计数器（只需原子性，无排序）
cuda::atomic<int, cuda::thread_scope_block> counter;
counter.fetch_add(1, cuda::memory_order_relaxed);

// 生产者-消费者（需 happens-before 保证）
cuda::atomic<bool, cuda::thread_scope_block> ready;
// 生产者：
data = 42;
ready.store(true, cuda::memory_order_release);  // release：data 写入对消费者可见
// 消费者：
while (!ready.load(cuda::memory_order_acquire)) {}  // acquire：看到 true 后 data 已可见
int val = data;
```

**独立线程调度（CC 7.0+）注意事项**：
```
旧代码（CC < 7.0）假设 warp lockstep 执行的地方，需改用 __syncwarp() 显式同步
典型场景：warp 内 reduction、intra-warp mutex、warp vote 函数
```

---

## Part 3 速查：Async Barrier 与 Pipeline 模式

> 来源：3.2.4.2–3.2.4.3

**Async Barrier（arrive-wait 分离）**：
```cpp
#include <cuda/barrier>
using barrier_t = cuda::barrier<cuda::thread_scope_block>;
__shared__ barrier_t bar;

// 初始化（只需一次，thread 0 执行）
if (threadIdx.x == 0) init(&bar, blockDim.x);
__syncthreads();

// 循环中：arrive 不阻塞，wait 阻塞
barrier_t::arrival_token token = bar.arrive();  // 非阻塞到达
compute_independent_work();                      // 独立工作
bar.wait(std::move(token));                      // 等待所有线程 arrive
```

**Pipeline（double-buffer 模式）**：
```
Stage 0: [load tile 0 async] ─────────────────────────────────────►
                               [compute tile 0] [load tile 1 async]
                                                                    [compute tile 1]
↑ 计算与数据加载重叠
```

**异步拷贝硬件机制对比**：

| 机制 | 引入版本 | 方向 | 规模 | 用途 |
|------|---------|------|------|------|
| LDGSTS | CC 8.0 | global → shared | 小块（绕过寄存器） | 通用异步加载 |
| TMA | CC 9.0 | global ↔ shared（含跨 cluster）| 大块多维 tensor | Transformer tile 加载 |
| STAS | CC 9.0 | 寄存器 → distributed shared | 小块 | cluster 内数据交换 |

---

## Part 3 速查：L1/Shared Memory 配置

> 来源：3.2.6 Configuring L1/Shared Memory Balance

```cpp
// 推荐：hint 语义，不会触发隐式同步
cudaFuncSetAttribute(kernel, cudaFuncAttributePreferredSharedMemoryCarveout,
                     cudaSharedmemCarveoutMaxShared);  // 最大化 shared memory
// 或指定百分比（向上取整到支持的档位）：
cudaFuncSetAttribute(kernel, cudaFuncAttributePreferredSharedMemoryCarveout, 50);

// 启用超过 48 KB 的 dynamic shared memory（必须显式 opt-in）
cudaFuncSetAttribute(kernel, cudaFuncAttributeMaxDynamicSharedMemorySize, 98304); // 96KB
kernel<<<grid, block, 98304>>>(args);
```

**各架构 shared memory 最大容量**（per SM）：

| 架构 | CC | 最大 shared memory |
|------|----|--------------------|
| Volta | 7.0 | 96 KB |
| Ampere | 8.0 | 164 KB |
| Hopper | 9.0 | 228 KB |
| Blackwell | 10.0 | 100 KB |

> **注意**：`cudaFuncSetCacheConfig`（旧 API）是 hard requirement，不同配置 kernel 交替时触发隐式同步 → 用 `cudaFuncSetAttribute` 替代。

---

## Part 3 速查：CUDA Driver API vs Runtime API

> 来源：3.3 The CUDA Driver API

| 对比维度 | Runtime API | Driver API |
|---------|------------|-----------|
| 前缀 | `cuda` | `cu` |
| 上下文管理 | 自动（primary context） | 手动（`cuCtxCreate`） |
| Module 加载 | 自动 | 手动（`cuModuleLoad`） |
| 可互用 | ✅ | ✅ |
| VMM / 高级特性 | 部分不支持 | ✅ 完整支持 |

**何时需要 Driver API**：
- Virtual Memory Management（细粒度内存映射 / IPC）
- 需要 floating context（在多线程间传递 context）
- 加载第三方 PTX/CUBIN、动态 JIT 编译

**混用模板**（Runtime + Driver）：
```cpp
// Runtime API 初始化后，从 Driver API 获取 primary context
CUcontext ctx;
cuCtxGetCurrent(&ctx);   // 拿到 Runtime 创建的 primary context

// CUdeviceptr 与 void* 互转
CUdeviceptr dptr = (CUdeviceptr)cuda_ptr;
float* ptr = (float*)dptr;
```

---

## Part 3 速查：Multi-GPU 编程模式

> 来源：3.4 Programming Systems with Multiple GPUs

**设备管理基本流程**：
```cpp
// 1. 枚举设备
int n;
cudaGetDeviceCount(&n);
for (int i = 0; i < n; i++) {
    cudaDeviceProp prop;
    cudaGetDeviceProperties(&prop, i);
    // 选设备：查 prop.major/minor、prop.totalGlobalMem、CPU affinity
}

// 2. 切换设备（stream/memory 绑定到当前设备）
cudaSetDevice(0);
cudaStream_t s0; cudaStreamCreate(&s0);
float *p0; cudaMalloc(&p0, size);

cudaSetDevice(1);
cudaStream_t s1; cudaStreamCreate(&s1);
float *p1; cudaMalloc(&p1, size);
```

**P2P 访问开启与拷贝**：
```cpp
// 检查并开启
int canAccess;
cudaDeviceCanAccessPeer(&canAccess, 0, 1);
if (canAccess) {
    cudaSetDevice(0);
    cudaDeviceEnablePeerAccess(1, 0);  // device 0 可访问 device 1 的显存
}

// 方式 1：P2P 拷贝（不经过 host）
cudaMemcpyPeer(p1, 1, p0, 0, size);           // 同步
cudaMemcpyPeerAsync(p1, 1, p0, 0, size, s1);  // 异步

// 方式 2：开启 P2P 后直接用指针跨 GPU 访问（UVA 下同一地址空间）
cudaSetDevice(1);
MyKernel<<<g, b>>>(p0);  // kernel 在 device 1 执行，读取 device 0 的 p0
```

**跨 GPU Stream 同步（Event 桥接）**：
```cpp
// cudaStreamWaitEvent 跨设备有效（即使 stream 和 event 属于不同设备）
cudaSetDevice(0);
cudaEventRecord(event, s0);      // device 0 的 s0 记录 event

cudaSetDevice(1);
cudaStreamWaitEvent(s1, event);  // device 1 的 s1 等待 device 0 的 event
MyKernel<<<g, b, 0, s1>>>();
```

**IOMMU 注意（Linux 裸机）**：
```
启用 IOMMU → PCIe P2P 传输 silent corruption → 必须禁用 IOMMU
VM pass-through → 应启用 IOMMU + VFIO 驱动
Windows → 无此限制
```


---

## Part 4: CUDA Features 速查

### 4.1 Unified Memory 速查
| API | 作用 |
|-----|------|
| cudaMallocManaged(&ptr, size) | 分配统一内存 |
| cudaMemPrefetchAsync(ptr, size, dev, stream) | 预取到指定设备 |
| cudaMemAdvise(ptr, size, advice, dev) | 访问提示 |

Advise 类型: SetReadMostly=多设备只读副本 / SetPreferredLocation=首选位置 / SetAccessedBy=映射不迁移

---

### 4.2 CUDA Graphs 速查
`cpp
cudaStreamBeginCapture(stream, cudaStreamCaptureModeGlobal);
kernel<<<g,b,0,stream>>>(...);
cudaGraph_t graph;
cudaStreamEndCapture(stream, &graph);
cudaGraphExec_t exec;
cudaGraphInstantiate(&exec, graph, 0);
cudaGraphLaunch(exec, stream);          // 可多次重放
// 更新节点参数（不重新实例化）
cudaGraphExecKernelNodeSetParams(exec, node, &params);
cudaGraphExecUpdate(exec, graph, &result);
`

---

### 4.3 Stream-Ordered Allocator 速查
`cpp
cudaMallocAsync(&ptr, size, stream);
kernel<<<g,b,0,stream>>>(ptr);
cudaFreeAsync(ptr, stream);
// 防止内存池自动缩减
uint64_t threshold = UINT64_MAX;
cudaMemPoolSetAttribute(pool, cudaMemPoolAttrReleaseThreshold, &threshold);
`

---

### 4.4 Cooperative Groups 速查
`cpp
#include <cooperative_groups.h>
namespace cg = cooperative_groups;
auto block = cg::this_thread_block();
auto warp  = cg::tiled_partition<32>(block);
auto grid  = cg::this_grid();  // 需 cudaLaunchCooperativeKernel
warp.sync();
int val = cg::reduce(warp, threadVal, cg::plus<int>());
grid.sync();  // 全网格同步
`

---

### 4.5 PDL 速查
`cpp
// Primary kernel
cudaTriggerProgrammaticLaunchCompletion();
// Secondary kernel
cudaGridDependencySynchronize();
// 启动属性
attr.id = cudaLaunchAttributeProgrammaticStreamSerialization;
attr.val.programmaticStreamSerializationAllowed = 1;
`

---

### 4.6 Green Contexts 速查
```cpp
CUdevResource devRes;
cudaDevGetDevResource(dev, &devRes, cudaDevResourceTypeSM);
CUdevSmResource groups[2];
unsigned int count = 2;
cudaDevSmResourceSplitByCount(groups, &count, &devRes, 0, numSMs/2);
CUdevResourceDesc desc;
cudaDevResourceGenerateDesc(&desc, groups, count);
CUgreenCtx greenCtx;
cudaGreenCtxCreate(&greenCtx, desc, dev, cudaGreenCtxDefaultFlags);
cudaStream_t stream;
cudaExecutionCtxStreamCreate(&stream, greenCtx, 0);
```

---

### 4.7 Lazy Loading 速查
```bash
CUDA_MODULE_LOADING=LAZY    # 默认（12.3+）
CUDA_MODULE_LOADING=EAGER   # 强制全量加载
```
```cpp
CUmoduleLoadingMode mode;
cuModuleGetLoadingMode(&mode);
// 强制加载特定函数
CUfunction fn;
cuModuleGetFunction(&fn, module, "kernelName");
```

---

### 4.8 Error Log Management 速查
```bash
export CUDA_LOG_FILE=stdout   # 或 stderr 或文件路径
```
```cpp
void myCallback(void* data, CUlogLevel level, char* msg, size_t len) {
    printf("[CUDA] %s\n", msg);
}
CUlogsCallbackHandle handle;
cuLogsRegisterCallback(myCallback, nullptr, &handle);
// 手动导出
CUlogIterator iter;
cuLogsCurrent(&iter, 0);
char buf[25600]; size_t sz = sizeof(buf);
cuLogsDumpToMemory(&iter, buf, &sz, 0);
```

---

### 4.9 Async Barriers 速查
```cpp
#include <cuda/barrier>
__shared__ cuda::barrier<cuda::thread_scope_block> bar;

__global__ void kernel() {
    if (threadIdx.x == 0) init(&bar, blockDim.x);
    __syncthreads();
    // 异步拷贝 + barrier
    auto token = cuda::device::barrier_arrive_tx(bar, 1, sizeof(data));
    cuda::memcpy_async(smem, gmem, sizeof(data), bar);
    bar.wait(std::move(token));  // 等待数据就绪
    // 简单用法
    bar.arrive_and_wait();
}
```

---

### 4.10 Pipelines 速查
```cpp
#include <cuda/pipeline>
__shared__ cuda::pipeline_shared_state<cuda::thread_scope_block, 2> state;

__global__ void kernel(float* gmem) {
    __shared__ float smem[2][TILE];
    auto pipe = cuda::make_pipeline(cg::this_thread_block(), &state);
    for (int i = 0; i < N+2; i++) {
        pipe.producer_acquire();
        if (i < N) cuda::memcpy_async(smem[i%2], gmem+i*TILE, sizeof(float)*TILE, pipe);
        pipe.producer_commit();
        pipe.consumer_wait();
        if (i >= 2) compute(smem[(i-2)%2]);
        pipe.consumer_release();
    }
}
```

---

### 4.11 Async Copies (TMA) 速查
```cpp
// LDGSTS (CC 8.0+): 16字节对齐触发最优 cp.async
cuda::memcpy_async(smem, gmem, cuda::aligned_size_t<16>(size), barrier);

// TMA 1D bulk (CC 9.0+)
cuda::device::memcpy_async_tx(dst_smem, src_gmem, size, bar);

// TMA 多维: Host 端创建 tensorMap
PFN_cuTensorMapEncodeTiled_v12000 fn;
cudaGetDriverEntryPointByVersion("cuTensorMapEncodeTiled",
    (void**)&fn, 12000, cudaEnableDefault, &status);
fn(&tensorMap, ...);
// Device 端
__global__ void k(const __grid_constant__ CUtensorMap tmap) {
    cuda::ptx::cp_async_bulk_tensor(&smem, &tmap, coords, &bar);
}
```

---

### 4.13 L2 Cache Control 速查
```cpp
// 设置持久化区域大小（最多75% L2）
cudaDeviceSetLimit(cudaLimitPersistingL2CacheSize, 6*1024*1024);
// 设置 stream 访问策略
cudaStreamAttrValue attr = {};
attr.accessPolicyWindow.base_ptr  = ptr;
attr.accessPolicyWindow.num_bytes = size;
attr.accessPolicyWindow.hitRatio  = 1.0f;
attr.accessPolicyWindow.hitProp   = cudaAccessPropertyPersisting;
attr.accessPolicyWindow.missProp  = cudaAccessPropertyStreaming;
cudaStreamSetAttribute(stream, cudaStreamAttributeAccessPolicyWindow, &attr);
// 重置持久化缓存
cudaCtxResetPersistingL2Cache();
```

---

### 4.16 Virtual Memory Management 速查
```cpp
// Driver API VMM 完整流程
CUmemAllocationProp prop = {};
prop.type = CU_MEM_ALLOCATION_TYPE_PINNED;
prop.location.type = CU_MEM_LOCATION_TYPE_DEVICE;
prop.location.id = device;

CUmemGenericAllocationHandle handle;
cuMemCreate(&handle, size, &prop, 0);      // 分配物理内存

CUdeviceptr ptr;
cuMemAddressReserve(&ptr, size, 0, 0, 0); // 预留虚地址
cuMemMap(ptr, size, 0, handle, 0);        // 建立映射

CUmemAccessDesc access = {};
access.location = prop.location;
access.flags = CU_MEM_ACCESS_FLAGS_PROT_READWRITE;
cuMemSetAccess(ptr, size, &access, 1);    // 开放访问权限

// 释放（顺序重要）
cuMemUnmap(ptr, size);
cuMemRelease(handle);
cuMemAddressFree(ptr, size);
```

---

### 4.17 Extended GPU Memory (EGM) 速查
```cpp
// Grace Hopper NVLink-C2C
int numaId;
cudaDeviceGetAttribute(&numaId, cudaDevAttrHostNumaId, device);
CUmemAllocationProp prop = {};
prop.type = CU_MEM_ALLOCATION_TYPE_PINNED;
prop.location.type = CU_MEM_LOCATION_TYPE_HOST_NUMA;
prop.location.id = numaId;
CUmemGenericAllocationHandle handle;
cuMemCreate(&handle, size, &prop, 0);
```

---

### 4.18 Dynamic Parallelism (CDP2) 速查
```cpp
__global__ void parent_kernel(int N) {
    if (threadIdx.x == 0) {
        // Fire-and-forget: 父不等待子
        child_kernel<<<grid, block, 0, cudaStreamFireAndForget>>>(N);
        // Tail launch: 父退出后子接替执行，节省资源
        // child_kernel<<<grid, block, 0, cudaStreamTailLaunch>>>(N);
    }
}
// 编译: nvcc --generate-code arch=compute_90,code=sm_90
```

---

### 4.19 Graphics Interop 速查
```cpp
// OpenGL 互操作
cudaGraphicsResource_t res;
cudaGraphicsGLRegisterBuffer(&res, vbo, cudaGraphicsMapFlagsWriteDiscard);
cudaGraphicsMapResources(1, &res, stream);
float4* dptr; size_t bytes;
cudaGraphicsResourceGetMappedPointer((void**)&dptr, &bytes, res);
myKernel<<<g,b,0,stream>>>(dptr);
cudaGraphicsUnmapResources(1, &res, stream);
cudaGraphicsUnregisterResource(res);

// Vulkan 外部资源导入
cudaExternalMemoryHandleDesc memDesc = {};
memDesc.type = cudaExternalMemoryHandleTypeOpaqueFd;  // Linux fd
memDesc.handle.fd = vulkanFd;
memDesc.size = bufferSize;
cudaExternalMemory_t extMem;
cudaImportExternalMemory(&extMem, &memDesc);
void* devPtr;
cudaExternalMemoryBufferDesc bufDesc = {0, bufferSize, 0};
cudaExternalMemoryGetMappedBuffer(&devPtr, extMem, &bufDesc);

// Vulkan Semaphore
cudaExternalSemaphore_t extSem;
cudaImportExternalSemaphore(&extSem, &semDesc);
cudaSignalExternalSemaphoresAsync(&extSem, &sigParams, 1, stream);
cudaWaitExternalSemaphoresAsync(&extSem, &waitParams, 1, stream);
```

---

### 4.20 Driver Entry Point Access 速查
```cpp
#include <cudaTypedefs.h>

PFN_cuMemAllocAsync_v11020 pfn_cuMemAllocAsync;
CUdriverProcAddressQueryResult status;
cudaGetDriverEntryPointByVersion(
    "cuMemAllocAsync",
    (void**)&pfn_cuMemAllocAsync,
    11020,                 // CUDA 11.2
    cudaEnableDefault,
    &status);

if (status == cudaDriverEntryPointSuccess) {
    pfn_cuMemAllocAsync(...);
}

// 获取 per-thread default stream 变体 (_ptsz 后缀)
cudaGetDriverEntryPointByVersion(
    "cuLaunchKernel",
    (void**)&pfn,
    version,
    cudaEnablePerThreadDefaultStream,
    &status);
```

---

## Best Practices Guide (BPG) 速查笔记

---

### BPG-1 开发方法论 APOD

```
Profile -> Find Hotspot -> Parallelize -> Optimize -> Deploy -> Repeat
```

- 用 gprof/Nsight 找 CPU 热点，集中资源
- 串行部分比例 P_serial = 10% → 最大加速 10x（Amdahl 定律）
- 先跑通，再优化；避免过早优化

---

### BPG-2 性能计时

**CPU 计时（粗粒度，需同步）**
```cpp
cudaDeviceSynchronize();
clock_t start = clock();
kernel<<<grid, block>>>(args);
cudaDeviceSynchronize();
double ms = 1000.0 * (clock() - start) / CLOCKS_PER_SEC;
```

**CUDA Event 计时（推荐，精度 ~0.5 μs）**
```cpp
cudaEvent_t start, stop;
cudaEventCreate(&start);
cudaEventCreate(&stop);

cudaEventRecord(start);
kernel<<<grid, block>>>(args);
cudaEventRecord(stop);

cudaEventSynchronize(stop);
float ms = 0;
cudaEventElapsedTime(&ms, start, stop);  // 毫秒

cudaEventDestroy(start);
cudaEventDestroy(stop);
```

---

### BPG-3 带宽计算

**理论带宽（GB/s）**
```
BW_theory = mem_clock_Hz * bus_width_bytes * 2 (DDR) / 1e9
# 例：A100 SXM 2.0GHz * 5120bit/8 * 2 = 2000 GB/s
```

**有效带宽测量**
```cpp
// 假设 N 个 float 的读 + 写
float bytes = 2.0f * N * sizeof(float);
float bw_gb = bytes / (ms * 1e6f);  // GB/s
printf("Effective BW: %.1f GB/s\n", bw_gb);
```

---

### BPG-4 内存层次优化速查

**Pinned Memory（锁页内存）**
```cpp
float *h_data;
cudaHostAlloc(&h_data, bytes, cudaHostAllocDefault);
// H2D 可达 ~12 GB/s vs 普通内存 ~6 GB/s
// 释放
cudaFreeHost(h_data);
```

**Zero-Copy（仅适合集成 GPU 或单次读写）**
```cpp
float *h_mapped, *d_mapped;
cudaHostAlloc(&h_mapped, bytes, cudaHostAllocMapped);
cudaHostGetDevicePointer(&d_mapped, h_mapped, 0);
kernel<<<grid, block>>>(d_mapped);  // GPU 直接访问主机内存
```

**异步传输与 Kernel 重叠（双 buffer ping-pong）**
```cpp
cudaStream_t s0, s1;
cudaStreamCreate(&s0);
cudaStreamCreate(&s1);

// 分段 staged 传输
int chunk = N / 2;
for (int i = 0; i < 2; i++) {
    int offset = i * chunk;
    cudaStream_t s = (i == 0) ? s0 : s1;
    cudaMemcpyAsync(d_in + offset, h_in + offset,
                    chunk * sizeof(float), cudaMemcpyHostToDevice, s);
    kernel<<<grid, block, 0, s>>>(d_in + offset, d_out + offset, chunk);
    cudaMemcpyAsync(h_out + offset, d_out + offset,
                    chunk * sizeof(float), cudaMemcpyDeviceToHost, s);
}
cudaStreamSynchronize(s0);
cudaStreamSynchronize(s1);
```

---

### BPG-5 Shared Memory Bank Conflict 消除

```cpp
// 矩阵转置：有 bank conflict（stride=32）
__shared__ float s[32][32];

// 消除方法：padding +1 列
__shared__ float s[32][33];  // 每行多1个float，错开bank对齐
// 写入（合并访问）
s[threadIdx.y][threadIdx.x] = d_in[...];
__syncthreads();
// 读出（转置后合并访问，无 bank conflict）
d_out[...] = s[threadIdx.x][threadIdx.y];
```

---

### BPG-6 Occupancy 优化

**查询 Occupancy**
```cpp
int blockSize = 256;
int minGridSize, gridSize;
// 自动选择最优 blockSize
cudaOccupancyMaxPotentialBlockSize(
    &minGridSize, &blockSize, myKernel, 0, 0);
gridSize = (N + blockSize - 1) / blockSize;
myKernel<<<gridSize, blockSize>>>(args);

// 查询实际 occupancy
int activeBlocks;
cudaOccupancyMaxActiveBlocksPerMultiprocessor(
    &activeBlocks, myKernel, blockSize, 0);
int maxBlocks;  // 查询 SM 最大 block 数
cudaDeviceProp prop;
cudaGetDeviceProperties(&prop, 0);
float occupancy = (float)(activeBlocks * blockSize)
                / (float)(prop.maxThreadsPerMultiProcessor);
printf("Occupancy: %.1f%%\n", occupancy * 100);
```

**`__launch_bounds__` 控制 register**
```cpp
// 保证每 block 最多 256 线程，每 SM 至少 2 block
__global__ __launch_bounds__(256, 2)
void myKernel(float* d) { ... }
```

**nvcc 控制寄存器上限**
```
nvcc -maxrregcount=32 kernel.cu -o kernel
```

---

### BPG-7 指令优化速查

| 优化 | 代替 | 加速 |
|------|------|------|
| `__sinf(x)` | `sinf(x)` | ~2x |
| `__expf(x)` | `expf(x)` | ~2x |
| `__fdividef(a,b)` | `a/b` | ~2x |
| `rsqrtf(x)` | `1.0f/sqrtf(x)` | ~2x |
| `sincosf(x,&s,&c)` | `sinf+cosf` | ~2x |
| `exp2f(x)` | `powf(2,x)` | ~10x |
| `exp10f(x)` | `powf(10,x)` | ~10x |
| `cbrtf(x)` | `powf(x,1.f/3.f)` | ~3x |
| `i>>k` | `i/(2^k)` | 有效 |
| `i&(2^k-1)` | `i%(2^k)` | 有效 |

**全局快速数学标志（精度降低约 1-2 ULP）**
```
nvcc -use_fast_math kernel.cu
# 等价于：-ftz=true -prec-div=false -prec-sqrt=false
```

**单独标志含义**
```
-ftz=true          # 非规格化数（denormals）刷新为零
-prec-div=false    # 低精度除法（~2 ULP）
-prec-sqrt=false   # 低精度 sqrt（~2 ULP）
```

---

### BPG-8 Warp Divergence 控制

```cpp
// 差：if 条件不按 warp 对齐 -> 严重 divergence
if (threadIdx.x % 2 == 0) { ... }

// 好：条件按 warp 对齐（32 线程一组）
if (threadIdx.x / 32 % 2 == 0) { ... }
// 或使用 warpSize（移植性更好）
if ((threadIdx.x / warpSize) % 2 == 0) { ... }

// 短分支让编译器自动 predication（无需手动）
// #pragma unroll 展开小循环消除分支
#pragma unroll 4
for (int i = 0; i < 4; i++) {
    sum += data[i * stride];
}
```

---

### BPG-9 多架构编译

```makefile
# 同时生成多个架构 cubin + PTX（最后一组保留 PTX 供未来 JIT）
NVCC_FLAGS = -gencode arch=compute_80,code=sm_80 \
             -gencode arch=compute_86,code=sm_86 \
             -gencode arch=compute_90,code=sm_90 \
             -gencode arch=compute_90,code=compute_90
```

---

### BPG-10 设备探测与错误处理

**设备探测模板**
```cpp
int devCount;
cudaGetDeviceCount(&devCount);
for (int i = 0; i < devCount; i++) {
    cudaDeviceProp prop;
    cudaGetDeviceProperties(&prop, i);
    printf("Device %d: %s (SM %d.%d)\n",
           i, prop.name, prop.major, prop.minor);
    printf("  Global Mem: %.0f MB, SMs: %d\n",
           prop.totalGlobalMem / 1e6, prop.multiProcessorCount);
    printf("  Max threads/SM: %d, Shared mem/SM: %zu KB\n",
           prop.maxThreadsPerMultiProcessor, prop.sharedMemPerMultiprocessor/1024);
    printf("  L2 Cache: %d KB, Concurrent kernels: %d\n",
           prop.l2CacheSize/1024, prop.concurrentKernels);
}
```

**错误处理宏**
```cpp
#define CUDA_CHECK(call)                                             \
    do {                                                             \
        cudaError_t err = (call);                                    \
        if (err != cudaSuccess) {                                    \
            fprintf(stderr, "CUDA error %s:%d: %s\n",              \
                    __FILE__, __LINE__, cudaGetErrorString(err));    \
            exit(EXIT_FAILURE);                                      \
        }                                                            \
    } while (0)

// kernel launch 后检查
CUDA_CHECK(cudaMemcpy(d_out, h_out, bytes, cudaMemcpyDeviceToHost));
kernel<<<grid, block>>>(args);
CUDA_CHECK(cudaGetLastError());      // 捕获 launch 错误
CUDA_CHECK(cudaDeviceSynchronize()); // 捕获异步执行错误
```

---

### BPG-11 nvidia-smi 常用命令

```bash
nvidia-smi                          # 概览（所有 GPU 状态）
nvidia-smi -q -d MEMORY             # 内存详情
nvidia-smi -q -d ECC                # ECC 错误统计
nvidia-smi --query-gpu=name,utilization.gpu,memory.used \
           --format=csv             # CSV 格式查询
nvidia-smi -L                       # 列出所有 GPU UUID
CUDA_VISIBLE_DEVICES=0,2 ./app      # 只使用 GPU 0 和 2
```

---

### BPG-12 nvcc 诊断 Flags

```bash
# 查看每个 kernel 的 register、smem、lmem 用量
nvcc --ptxas-options=-v kernel.cu

# 关闭 FTZ 保留 denormal（debug 精度问题用）
nvcc -ftz=false kernel.cu

# 查看 PTX
nvcc -ptx kernel.cu -o kernel.ptx

# 查看 SASS（需 CUDA toolkit cuobjdump）
cuobjdump --dump-sass kernel.cubin
```


---

## CUDA Runtime API 速查笔记

### RT-1: 设备初始化与查询模板

```cpp
#include <cuda_runtime.h>
#include <cstdio>

int main() {
    int deviceCount = 0;
    cudaGetDeviceCount(&deviceCount);
    printf("Found %d CUDA device(s)\n", deviceCount);

    for (int i = 0; i < deviceCount; i++) {
        cudaDeviceProp prop;
        cudaGetDeviceProperties(&prop, i);
        printf("Device %d: %s\n", i, prop.name);
        printf("  Compute Capability: %d.%d\n", prop.major, prop.minor);
        printf("  SMs: %d\n", prop.multiProcessorCount);
        printf("  Global Memory: %.2f GB\n", prop.totalGlobalMem / 1e9);
        printf("  Shared Mem/Block: %zu KB\n", prop.sharedMemPerBlock / 1024);
        printf("  Max Threads/Block: %d\n", prop.maxThreadsPerBlock);
        printf("  Warp Size: %d\n", prop.warpSize);
    }

    cudaSetDevice(0);  // 使用第一个设备
    return 0;
}
```

### RT-2: 错误处理宏（CUDA_CHECK）

```cpp
#include <cstdio>
#include <cstdlib>

#define CUDA_CHECK(call)                                                   \
    do {                                                                    \
        cudaError_t err = (call);                                          \
        if (err != cudaSuccess) {                                          \
            fprintf(stderr, "CUDA Error at %s:%d - %s: %s\n",             \
                    __FILE__, __LINE__,                                     \
                    cudaGetErrorName(err), cudaGetErrorString(err));        \
            exit(EXIT_FAILURE);                                            \
        }                                                                  \
    } while (0)

// 用于检查 kernel 启动后的异步错误
#define CUDA_CHECK_LAST()                                                  \
    do {                                                                    \
        cudaError_t err = cudaGetLastError();                              \
        if (err != cudaSuccess) {                                          \
            fprintf(stderr, "CUDA Kernel Error at %s:%d - %s: %s\n",      \
                    __FILE__, __LINE__,                                     \
                    cudaGetErrorName(err), cudaGetErrorString(err));        \
            exit(EXIT_FAILURE);                                            \
        }                                                                  \
    } while (0)

// 使用示例
// CUDA_CHECK(cudaMalloc(&d_ptr, size));
// myKernel<<<grid, block>>>(d_ptr);
// CUDA_CHECK_LAST();
// CUDA_CHECK(cudaDeviceSynchronize());
```

### RT-3: Stream 与 Graph Capture 流程

```cpp
// === 基础 Stream 使用 ===
cudaStream_t stream;
CUDA_CHECK(cudaStreamCreateWithFlags(&stream, cudaStreamNonBlocking));

// 异步操作排入流
CUDA_CHECK(cudaMemcpyAsync(d_in, h_in, size, cudaMemcpyHostToDevice, stream));
myKernel<<<grid, block, 0, stream>>>(d_in, d_out);
CUDA_CHECK(cudaMemcpyAsync(h_out, d_out, size, cudaMemcpyDeviceToHost, stream));

CUDA_CHECK(cudaStreamSynchronize(stream));
CUDA_CHECK(cudaStreamDestroy(stream));

// === Stream 间依赖（Event） ===
cudaStream_t s1, s2;
cudaEvent_t event;
cudaStreamCreate(&s1); cudaStreamCreate(&s2);
cudaEventCreate(&event);

kernelA<<<g, b, 0, s1>>>(data);
cudaEventRecord(event, s1);       // s1 中标记
cudaStreamWaitEvent(s2, event, 0); // s2 等待 s1 到达 event
kernelB<<<g, b, 0, s2>>>(data);   // 保证在 kernelA 之后执行

// === Graph Capture ===
cudaGraph_t graph;
cudaGraphExec_t graphExec;

cudaStreamBeginCapture(stream, cudaStreamCaptureModeGlobal);
// 排入要捕获的操作（不会执行）
kernelA<<<g, b, 0, stream>>>(args);
kernelB<<<g, b, 0, stream>>>(args);
cudaStreamEndCapture(stream, &graph);

cudaGraphInstantiate(&graphExec, graph, NULL, NULL, 0);

// 多次重放
for (int i = 0; i < iterations; i++) {
    cudaGraphLaunch(graphExec, stream);
}
cudaStreamSynchronize(stream);

cudaGraphExecDestroy(graphExec);
cudaGraphDestroy(graph);
```

### RT-4: Event 计时模板

```cpp
cudaEvent_t start, stop;
cudaEventCreate(&start);
cudaEventCreate(&stop);

cudaEventRecord(start, stream);
myKernel<<<grid, block, 0, stream>>>(args);
cudaEventRecord(stop, stream);

cudaEventSynchronize(stop);

float ms = 0.0f;
cudaEventElapsedTime(&ms, start, stop);
printf("Kernel time: %.3f ms\n", ms);

cudaEventDestroy(start);
cudaEventDestroy(stop);
```

**注意**: 使用 `cudaEventDisableTiming` 创建的事件不能用于计时但同步开销更低。

### RT-5: Memory 分配速查表

| 分配方式 | 函数 | 位置 | 特点 |
|---------|------|------|------|
| 设备内存 | `cudaMalloc` | GPU | 最常用，需手动 H↔D 拷贝 |
| 2D 对齐 | `cudaMallocPitch` | GPU | 行对齐优化 2D 访问 |
| 3D 对齐 | `cudaMalloc3D` | GPU | 3D pitched 分配 |
| Unified Memory | `cudaMallocManaged` | CPU+GPU | 自动迁移，开发简单 |
| 页锁定 | `cudaMallocHost` | CPU(pinned) | DMA 直传，异步拷贝必需 |
| 带标志页锁定 | `cudaHostAlloc` | CPU(pinned) | Portable/Mapped/WC |
| 流顺序 | `cudaMallocAsync` | GPU | 池化分配，低开销 |
| 从指定池 | `cudaMallocFromPoolAsync` | GPU | 可跨设备指定池 |

**内存拷贝对比**:
| 函数 | 同步/异步 | 适用 |
|------|----------|------|
| `cudaMemcpy` | 同步（阻塞） | 简单场景 |
| `cudaMemcpyAsync` | 异步 | 需 pinned 内存 + stream |
| `cudaMemcpy2D` | 同步 | pitched 2D 内存 |
| `cudaMemcpy3D` | 同步 | 3D 内存 |

### RT-6: Occupancy API 用法

```cpp
// 方法1: 手动指定 blockSize，查询每 SM 活跃 blocks
int numBlocks;
int blockSize = 256;
size_t dynamicSmem = 0;
cudaOccupancyMaxActiveBlocksPerMultiprocessor(
    &numBlocks, myKernel, blockSize, dynamicSmem);
printf("Max active blocks/SM: %d\n", numBlocks);

// 方法2: 自动搜索最优 blockSize（C++ 模板）
int minGridSize, optBlockSize;
cudaOccupancyMaxPotentialBlockSize(
    &minGridSize, &optBlockSize, myKernel, 0, 0);
printf("Optimal block size: %d, min grid: %d\n", optBlockSize, minGridSize);

// 方法3: 查询给定配置下可用的动态 shared memory
size_t availSmem;
cudaOccupancyAvailableDynamicSMemPerBlock(
    &availSmem, myKernel, numBlocks, blockSize);
printf("Available dynamic smem: %zu bytes\n", availSmem);
```

### RT-7: Peer Access (P2P) 模板

```cpp
// 多 GPU P2P 直接内存访问
int canAccess;
cudaDeviceCanAccessPeer(&canAccess, 0, 1);  // GPU0 能否访问 GPU1?

if (canAccess) {
    cudaSetDevice(0);
    cudaDeviceEnablePeerAccess(1, 0);  // GPU0 启用对 GPU1 的访问

    cudaSetDevice(1);
    cudaDeviceEnablePeerAccess(0, 0);  // GPU1 启用对 GPU0 的访问（对称）

    // 现在 GPU0 的 kernel 可直接读写 GPU1 上 cudaMalloc 的内存
    // 也可用 cudaMemcpyPeer 显式 P2P 拷贝:
    // cudaMemcpyPeer(d_dst_gpu0, 0, d_src_gpu1, 1, size);

    // 清理
    cudaSetDevice(0); cudaDeviceDisablePeerAccess(1);
    cudaSetDevice(1); cudaDeviceDisablePeerAccess(0);
}
```

### RT-8: cudaLaunchKernelExC + Cluster Launch 模板

```cpp
// === 使用 cudaLaunchKernelExC 设置 launch-time 属性 ===
cudaLaunchConfig_t config = {};
config.gridDim = dim3(numBlocks);
config.blockDim = dim3(blockSize);
config.dynamicSmemBytes = sharedMemSize;
config.stream = stream;

// 设置属性数组
cudaLaunchAttribute attrs[2];
attrs[0].id = cudaLaunchAttributeClusterDimension;
attrs[0].val.clusterDim = {2, 1, 1};  // 2-block cluster

attrs[1].id = cudaLaunchAttributeClusterSchedulingPolicyPreference;
attrs[1].val.clusterSchedulingPolicyPreference = cudaClusterSchedulingPolicySpread;

config.attrs = attrs;
config.numAttrs = 2;

cudaLaunchKernelExC(&config, (void*)myClusterKernel, kernelArgs);

// === cudaLaunchHostFunc 在流中排队 host 回调 ===
void CUDART_CB myCallback(void* userData) {
    printf("Stream reached this point! data=%p\n", userData);
    // 可以调用 CUDA API（与旧的 cudaStreamAddCallback 不同）
}

cudaLaunchHostFunc(stream, myCallback, (void*)myData);
```

### RT-9: Stream Ordered Memory Allocator 使用模板

```cpp
// === 基础流顺序分配 ===
void* d_buf;
cudaStream_t stream;
cudaStreamCreate(&stream);

cudaMallocAsync(&d_buf, size, stream);        // 流中分配
myKernel<<<grid, block, 0, stream>>>(d_buf);  // 使用
cudaFreeAsync(d_buf, stream);                 // 流中释放
cudaStreamSynchronize(stream);

// === 自定义 Memory Pool ===
cudaMemPoolProps poolProps = {};
poolProps.allocType = cudaMemAllocationTypePinned;
poolProps.location.type = cudaMemLocationTypeDevice;
poolProps.location.id = 0;  // GPU 0

cudaMemPool_t myPool;
cudaMemPoolCreate(&myPool, &poolProps);

// 设置释放阈值（保留 64MB 不还给 OS）
cuuint64_t threshold = 64ULL * 1024 * 1024;
cudaMemPoolSetAttribute(myPool, cudaMemPoolAttrReleaseThreshold, &threshold);

// 从自定义池分配
void* d_ptr;
cudaMallocFromPoolAsync(&d_ptr, size, myPool, stream);

// 修剪 pool（释放超出 minBytesToKeep 的 OS 内存）
cudaMemPoolTrimTo(myPool, 0);

// P2P pool 访问
cudaMemAccessDesc accessDesc = {};
accessDesc.location.type = cudaMemLocationTypeDevice;
accessDesc.location.id = 1;  // 允许 GPU1 访问
accessDesc.flags = cudaMemAccessFlagsProtReadWrite;
cudaMemPoolSetAccess(myPool, &accessDesc, 1);

cudaMemPoolDestroy(myPool);
```

### RT-10: Unified Memory 最佳实践

```cpp
float* data;
cudaMallocManaged(&data, N * sizeof(float));

// 设置建议：数据偏好驻留在 GPU 0
cudaMemAdvise(data, N * sizeof(float),
              cudaMemAdviseSetPreferredLocation, 0);

// 建议 GPU 1 也会访问（建立映射，避免 page fault）
cudaMemAdvise(data, N * sizeof(float),
              cudaMemAdviseSetAccessedBy, 1);

// 显式预取到 GPU 0（避免首次访问 page fault）
cudaMemPrefetchAsync(data, N * sizeof(float), 0, stream);

// 对只读数据启用 read-duplication
cudaMemAdvise(readOnlyData, size,
              cudaMemAdviseSetReadMostly, 0);

// 预取回 CPU
cudaMemPrefetchAsync(data, N * sizeof(float), cudaCpuDeviceId, stream);
```

**UM 性能要点**:
- 首次访问无预取 → page fault（几十 μs 延迟）
- `cudaMemPrefetchAsync` 批量迁移，摊薄开销
- `SetPreferredLocation` 减少不必要的来回迁移
- `SetReadMostly` 允许多设备各持有副本
- `SetAccessedBy` 建立直接映射（无迁移但可能走 PCIe）


---

## Compute Capabilities 速查笔记

> 来源：CUDA Programming Guide - 5.1 Compute Capabilities

### CC-1 架构 → 计算能力 → 代表 GPU 速查

```
┌─────────────────────────────────────────────────────────────────────────┐
│ 架构          │ CC     │ 代表 GPU                                      │
├─────────────────────────────────────────────────────────────────────────┤
│ Turing        │ 7.5    │ T4, RTX 2080 Ti, Titan RTX                   │
│ Ampere        │ 8.0    │ A100, A30                                     │
│               │ 8.6    │ A40, RTX 3090/3080/3070/3060, RTX A6000       │
│               │ 8.7    │ Jetson AGX Orin                               │
│ Ada Lovelace  │ 8.9    │ L4, L40S, RTX 4090/4080/4070/4060            │
│ Hopper        │ 9.0    │ H100, H200, GH200                            │
│ Blackwell     │ 10.0   │ B200, GB200                                   │
│               │ 10.3   │ GB300, B300                                   │
│               │ 12.0   │ RTX 5090/5080/5070/5060, RTX PRO 6000        │
│               │ 12.1   │ GB10 (DGX Spark)                              │
│ Thor          │ 11.0   │ Jetson T5000/T4000                            │
└─────────────────────────────────────────────────────────────────────────┘
```

**记忆技巧**：CC major 数字 ≠ 架构代数，看产品线定位：
- 数据中心旗舰：A100(8.0) → H100(9.0) → B200(10.0)
- 消费级旗舰：RTX 3090(8.6) → RTX 4090(8.9) → RTX 5090(12.0)

---

### CC-2 SM 核心参数对比表

```
┌───────────────────────────────────────────────────────────────────────────────┐
│ 参数                          │ 7.5  │ 8.0  │ 8.6  │ 8.9  │ 9.0  │ 10.x │ 12.x│
├───────────────────────────────────────────────────────────────────────────────┤
│ Max Blocks/SM                 │ 16   │ 32   │ 16   │ 16   │ 32   │ 24   │ 24  │
│ Max Warps/SM                  │ 32   │ 64   │ 48   │ 48   │ 64   │ 48   │ 48  │
│ Max Threads/SM                │ 1024 │ 2048 │ 1536 │ 1536 │ 2048 │ 1536 │ 1536│
│ Shared Mem/SM (KB)            │ 64   │ 164  │ 100  │ 100  │ 228  │ 100  │ 100 │
│ Max Shared Mem/Block (KB)     │ 64   │ 163  │ 99   │ 99   │ 227  │ 99   │ 99  │
│ L1/SMEM Configurable (KB tot) │ 96   │ 192  │ 128  │ 128  │ 256  │ 256  │ 128 │
│ Registers/SM                  │ 64K  │ 64K  │ 64K  │ 64K  │ 64K  │ 64K  │ 64K │
│ Max Registers/Thread          │ 255  │ 255  │ 255  │ 255  │ 255  │ 255  │ 255 │
└───────────────────────────────────────────────────────────────────────────────┘
```

**所有 CC 不变量**：Warp=32, MaxThreads/Block=1024, Grid x=2³¹-1, Regs/SM=64K

---

### CC-3 Tensor Core 支持矩阵速查

```
精度     │ 7.5 │ 8.0 │ 8.6 │ 8.9 │ 9.0 │10.0 │10.3 │12.x │
─────────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┼─────┤
FP64     │  -  │  ✓  │  -  │  -  │  ✓  │  ✓  │  -  │  -  │
TF32     │  -  │  ✓  │  ✓  │  ✓  │  ✓  │  ✓  │  ✓  │  ✓  │
BF16     │  -  │  ✓  │  ✓  │  ✓  │  ✓  │  ✓  │  ✓  │  ✓  │
FP16     │  ✓  │  ✓  │  ✓  │  ✓  │  ✓  │  ✓  │  ✓  │  ✓  │
FP8      │  -  │  -  │  -  │  ✓  │  ✓  │  ✓  │  ✓  │  ✓  │
FP6      │  -  │  -  │  -  │  -  │  -  │  ✓  │  ✓  │  ✓  │
FP4      │  -  │  -  │  -  │  -  │  -  │  ✓  │  ✓  │  ✓  │
INT8     │  ✓  │  ✓  │  ✓  │  ✓  │  ✓  │  ✓  │  ✓  │  ✓  │
INT4     │  ✓  │  ✓  │  ✓  │  ✓  │  -  │  -  │  -  │  -  │
```

**趋势**：精度逐代下探（FP8→FP6→FP4），INT4 被 FP4 取代

---

### CC-4 编译目标选择指南

```python
# 最常见编译策略（覆盖主流 GPU）
# 方式 1：nvcc 多目标 (fat binary)
nvcc -gencode arch=compute_80,code=sm_80 \  # A100
     -gencode arch=compute_86,code=sm_86 \  # RTX 3090
     -gencode arch=compute_89,code=sm_89 \  # RTX 4090
     -gencode arch=compute_90,code=sm_90 \  # H100
     -gencode arch=compute_90,code=compute_90  # PTX for future GPUs

# 方式 2：cmake（更常见）
set(CMAKE_CUDA_ARCHITECTURES "80;86;89;90")

# Blackwell 编译（需要 CUDA 12.8+）
nvcc -arch=compute_100 -code=sm_100  # baseline 特性
nvcc -arch=compute_100a             # 全部架构专属特性
nvcc -arch=compute_100f             # 家族共享特性
```

**选择原则**：
- 生产部署：指定具体 `sm_XX` 获得最优性能
- 通用分发：附带 PTX (`code=compute_XX`) 确保未来兼容
- 架构专属特性（TMA/DPX/DSMEM）：必须用 `compute_XXa`

---

### CC-5 Occupancy 计算核心参数

```python
# 影响 Occupancy 的三个限制因素
# 1. Registers per thread → 总 warp 受 64K regs/SM 限制
# 2. Shared memory per block → 总 block 受 SMEM/SM 限制  
# 3. Threads per block → 总 block 受 Max Blocks/SM 限制

# 快速计算示例 (CC 9.0, H100)
regs_per_thread = 32
threads_per_block = 256
smem_per_block = 8192  # 8KB

# 寄存器限制：64K regs / (32 regs × 256 threads) = 8 blocks
reg_limited_blocks = 65536 // (regs_per_thread * threads_per_block)  # = 8

# SMEM 限制：228KB / 8KB = 28 blocks → capped at 32 (max blocks/SM)
smem_limited_blocks = (228 * 1024) // smem_per_block  # = 29 → cap 32

# Block 数量限制：直接取 min(reg, smem, max_blocks_per_sm)
active_blocks = min(reg_limited_blocks, smem_limited_blocks, 32)  # = 8

# Occupancy = active_warps / max_warps_per_SM
warps_per_block = threads_per_block // 32  # = 8
occupancy = (active_blocks * warps_per_block) / 64  # = 8×8/64 = 100%
```

**工具**：
- `cudaOccupancyMaxActiveBlocksPerMultiprocessor()` — 运行时查询
- CUDA Occupancy Calculator（Excel）— 离线规划
- Nsight Compute → Occupancy 分析面板

---

### CC-6 架构特性引入时间线

```
CC 7.5 (Turing, 2018):
  ├── RT Core (光追), Tensor Core Gen2 (FP16/INT8/INT4)
  └── 独立整数执行管道

CC 8.0 (Ampere, 2020):
  ├── TF32 Tensor Core, FP64 Tensor Core (A100)
  ├── Hardware memcpy_async, Async Barrier
  ├── L2 Cache Residency Control
  └── Reduced-precision atomics (BF16)

CC 8.9 (Ada Lovelace, 2022):
  ├── FP8 Tensor Core, 4th-gen RT Core
  └── Shader Execution Reordering (SER)

CC 9.0 (Hopper, 2022):
  ├── Transformer Engine (FP8 自动混合精度)
  ├── DPX 原生指令, DSMEM, Thread Block Cluster
  ├── TMA (Tensor Memory Accelerator)
  └── 228KB SMEM/SM, 2048 threads/SM

CC 10.0 (Blackwell, 2024):
  ├── FP6/FP4 Tensor Core, 5th-gen Tensor Core
  ├── 2nd-gen Transformer Engine
  └── Secure MIG, Decompression Engine

CC 12.0 (Blackwell Consumer, 2025):
  └── RTX 50 系列（桌面/笔记本）
```

---

### CC-7 FP32:FP64 吞吐比速查

```
CC 7.5  → 32:1  (Turing: FP64 极弱)
CC 8.0  →  2:1  (A100: FP64 性能王者)
CC 8.6  → 64:1  (RTX 3090: 消费卡无 FP64)
CC 8.9  → 64:1  (RTX 4090: 同上)
CC 9.0  →  2:1  (H100: FP64 回归)
CC 10.0 → 64:1  (B200: 推理定位，但有 FP64 TC)
CC 12.x → 64:1  (RTX 5090: 消费卡)
```

**规律**：数据中心旗舰(A100/H100) FP64=2:1，其余 GPU FP64≥32:1
如需 FP64 科学计算 → 只选 A100/H100

---

### CC-8 共享内存配置选项 (L1/SMEM split)

```
CC 7.5:  total 96KB  → options: 32, 64 KB for SMEM
CC 8.0:  total 192KB → options: 0, 8, 16, 32, 64, 100, 132, 164 KB
CC 8.6:  total 128KB → options: 0, 8, 16, 32, 64, 100 KB
CC 8.9:  total 128KB → options: 0, 8, 16, 32, 64, 100 KB
CC 9.0:  total 256KB → options: 0, 8, 16, 32, 64, 100, 132, 164, 196, 228 KB
CC 10.x: total 256KB → options: 0, 8, 16, 32, 64, 100, 132, 164, 196, 228 KB
CC 12.x: total 128KB → options: 0, 8, 16, 32, 64, 100 KB
```

**设置方式**：
```cpp
// 为 kernel 设置最大共享内存偏好
cudaFuncSetAttribute(kernel,
    cudaFuncAttributeMaxDynamicSharedMemorySize,
    requiredSmemBytes);
// 注意：超过 48KB 必须显式 opt-in
```
