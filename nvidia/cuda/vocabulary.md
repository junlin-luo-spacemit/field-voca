---
tags: [nvidia, cuda, vocabulary]
domain: cuda
last_updated: 2026-06-29
part: Part 1 ✅, Part 2 ✅
source: https://docs.nvidia.com/cuda/cuda-programming-guide/
---

# CUDA 词汇表

> 来源：https://docs.nvidia.com/cuda/cuda-programming-guide/index.html
> 版本：May 27, 2026
> 格式：英文术语 → 中文译名 + 官方定义 + 例句

---

## 编程模型（Programming Model）

### host / 主机端
CPU 及其直连连接的内存（system memory / host memory）的统称。CUDA 应用总是从 host 开始执行。
> *The host code uses CUDA APIs to copy data between host memory and device memory.*

### device / 设备端
GPU 及其直连连接的内存（device memory）的统称。
> *Device code runs on the GPU and is launched by the host.*

### host code / 主机端代码
在 CPU 上运行的代码，通过 CUDA API 控制 GPU 负载、内存拷贝和 kernel 启动。

### device code / 设备端代码
在 GPU 上执行的代码，即 kernel 函数中的内容。

### kernel launch / kernel 启动
host 代码启动一个 kernel 在 GPU 上并行执行的行为。需指定 **execution configuration**（执行配置）。
> *`myKernel<<<gridDim, blockDim, sharedMem, stream>>>(args)`*

### execution configuration / 执行配置
kernel 启动时用 `<<<...>>>` 语法指定的参数，包括 grid/block 维度、shared memory 大小、stream 等。

### thread / 线程
GPU 上执行的最小执行单元。每个线程有唯一 ID，可用内建变量 `threadIdx`、`blockIdx` 确定并行工作范围。
> *Each thread executes the kernel function and uses its unique ID to determine what data it is responsible for.*
> 每个线程执行核函数，并通过唯一 ID 确定其负责的数据。

### thread block / 线程块
一组协作线程的集合。同一 block 内的线程运行在同一个 SM 上，可共享 shared memory 并相互同步。最多 1024 个线程（**应为 32 的倍数**）。
> *All threads of a thread block are executed by a single SM and share on-chip shared memory.*

### grid / 网格
一次 kernel 启动的所有 thread block 的集合。可是 1、2、3 维。Grid 可包含数百万个 blocks。
> *A grid may consist of millions of thread blocks, while the GPU may have only tens or hundreds of SMs.*

### thread block cluster / 线程块簇
**Compute Capability 9.0+（Hopper）新引入**的可选分组层次，介于 block 和 grid 之间。同一 cluster 内的 blocks 在**同一个 GPC** 中并发执行，可访问 **distributed shared memory**。
> *Thread blocks in a cluster can communicate and synchronize using Cooperative Groups.*

### GPC (Graphics Processing Cluster) / 图形处理簇
多个 SM 的分组，是 thread block cluster 的调度单元。

### warp / 线程束
GPU 调度和执行的基本单位，**固定 32 个线程**。以 SIMT 方式执行。线程从 block 中按可预测方式分配到 warp lanes（0-31）。
> *Within a thread block, threads are organized into groups of 32 threads called warps.*
> *One implication of warp execution is that thread blocks are best specified to have a total number of threads which is a multiple of 32.*

### warp lane / warp 通道
一个 warp 内每个线程的编号（0-31）。

### kernel / 核函数
在 GPU 上并行执行的函数，用 `__global__` 修饰。启动后大量线程并行运行该函数代码。
> *A function invoked for execution on the GPU is called a kernel. Launching a kernel starts many threads executing the kernel code in parallel.*

### SIMT (Single Instruction, Multiple Threads) / 单指令多线程
NVIDIA GPU 的执行范式。warp 中所有线程执行相同指令，但每个线程可以走不同的分支路径（即不同于 SIMD 的固定控制流）。
> *In SIMT, all threads in the warp are executing the same kernel code, but each thread may follow different branches through the code.*

---

## Tile 编程模型（Tile Programming Model）

> **CUDA 新特性**（2026 版 Programming Guide 新增章节）

### tile programming model / Tile 编程模型
CUDA 支持的第二种编程范式。开发者以**整个 block** 为粒度写代码，操作称为 **tile** 的多维数据块；编译器自动将操作映射到 block 内的各线程。与 SIMT 并存，适合矩阵运算等规则计算。
> *In tile programming, the programmer writes code at the level of an entire thread block. The compiler maps these operations to the individual threads of the block.*

### tile / 数据块（tile）
Tile 编程模型中的数据单元，是一个**多维的值集合**，存在于 block 内部，对线程是局部的。
- **不可变（immutable）**：对 tile 的操作产生新 tile，而非修改原有 tile
- 每维尺寸必须是 2 的幂次，且**编译时确定**
- 编译器决定 tile 数据存储方式（寄存器/shared memory）
- 不可作为 kernel 参数传递
> *A tile is a multidimensional collection of values that is local to a single block and immutable.*

### array（tile 编程中）/ 全局数组
Tile 编程中存储在 device memory 的多维容器，**可变**，tile 通过 load/store 与 array 交换数据。

### tile space / tile 空间
Array 按照指定 tile 形状进行等分切割的概念空间。load 操作用 tile-space 索引（如 `(i,j)`）定位并取回对应的 tile；越界元素用零填充（或按指定规则处理）。

### distributed shared memory / 分布式共享内存
Thread block cluster（Hopper+）特性：cluster 内所有 block 的 shared memory 对彼此可见，形成更大范围的片上可编程内存。
> *Threads in clusters can access the shared memory of all blocks in the cluster.*

---

## 内存层次（Memory Hierarchy）

### global memory / 全局内存
GPU 上最大的内存空间（即 device memory / VRAM），**所有 SM 上的所有线程均可访问**，延迟最高（~400-800 cycles）。对应官方术语 *device memory* 或 *DRAM*。
> *From the perspective of device code, the DRAM attached to the GPU is called global memory, because it is accessible to all SMs in the GPU.*

### shared memory / 共享内存
每个 SM 上的片上可编程内存（on-chip），同一 block（或同一 cluster，见 distributed shared memory）内的线程共享。延迟极低，容量有限。与 L1 cache 共用物理资源（unified data cache），比例可在运行时配置。
> *Shared memory can be used for exchanging data between threads of a thread block or cluster.*

### unified data cache / 统一数据缓存
每个 SM 内 shared memory 与 L1 cache 共享的物理资源。可在运行时配置两者的分配比例。
> *The unified data cache provides the physical resources for shared memory and L1 cache.*

### register file / 寄存器堆
每个 SM 上的寄存器总资源，由 block 内所有线程共享（各线程私占部分寄存器）。寄存器不足时会触发 register spilling。
> *The register file stores thread local variables which are usually allocated by the compiler.*

### register / 寄存器
每个线程私有，速度最快，数量有限（每个 SM 约 65536 个 32-bit 寄存器）。调度 block 到 SM 时，总寄存器需求（每线程寄存器数 × 线程数）不得超过 SM 寄存器堆大小。

### constant memory / 常量内存
只读，带专用 constant cache（per-SM），适合广播到所有线程的不变数据（如 kernel 参数）。
> *The compiler may place kernel parameters into constant memory to allow them to be cached in the SM separately from the L1 data cache.*

### constant cache / 常量缓存
每个 SM 上专门缓存 constant memory 内容的缓存，与 L1 data cache 独立。

### texture memory / 纹理内存
带空间局部性缓存的只读内存，适合 2D 空间访问模式（图像处理、体素数据等）。

### L1 cache / 一级缓存
每个 SM 内的缓存，物理上与 shared memory 共用 unified data cache，硬件自动管理。

### L2 cache / 二级缓存
所有 SM 共享的片上缓存层，位于 SM 与 global memory（HBM）之间。

### Unified Memory / 统一内存
CUDA 特性，允许分配 CPU 和 GPU 均可访问的内存。CUDA runtime 或底层硬件在需要时自动将数据迁移到正确的位置。简化编程，但最优性能仍需尽量减少数据迁移。
> *A CUDA feature called unified memory allows applications to make memory allocations which can be accessed from CPU or GPU.*

### mapped memory / 映射内存
CPU 内存通过 PCIe/NVLink 直接映射供 GPU 访问，GPU 无法隐藏 PCIe 的高延迟和低带宽，**性能差**，不推荐替代 Unified Memory。

### HBM (High Bandwidth Memory) / 高带宽内存
通过 TSV（硅通孔）堆叠的 DRAM，与 GPU 封装在同一基板，极低延迟、极高带宽。
| 型号 | HBM 版本 | 显存 | 带宽 |
|------|----------|------|------|
| A100 SXM | HBM2e | 80 GB | 2.0 TB/s |
| H100 SXM | HBM3 | 80 GB | 3.35 TB/s |
| B200 | HBM3e | 192 GB | 8.0 TB/s |

---

## 硬件概念（Hardware Concepts）

### SM (Streaming Multiprocessor) / 流式多处理器
GPU 的基本计算单元，包含：register file、unified data cache（L1 + shared memory）、constant cache、CUDA Core、Tensor Core 等功能单元。一个 SM 同时执行一个或多个 thread block（至寄存器/shared memory 资源耗尽为止）。
> *All threads of a thread block are executed in a single SM.*

### GPC (Graphics Processing Cluster) / 图形处理簇
多个 SM 的分组单元，也是 thread block cluster（Compute Capability 9.0+）的调度范围。

### functional units / 功能单元
SM 内执行实际计算的硬件单元，包括 CUDA Core、Tensor Core、RT Core 等，数量因架构不同而异。

### CUDA Core / CUDA 核心
执行标量浮点（FP32/FP64）和整数（INT32）运算的基本处理单元。

### Tensor Core / 张量核心
专为矩阵乘加（MMA / WMMA）运算设计的硬件单元，是深度学习加速的核心来源。
| 代数 | 架构 | 支持精度 |
|------|------|----------|
| 第1代 | Volta | FP16 |
| 第2代 | Turing | FP16, INT8, INT4 |
| 第3代 | Ampere | FP16, BF16, TF32, FP64, INT8 |
| 第4代 | Hopper | FP16, BF16, TF32, FP8, INT8 |
| 第5代 | Blackwell | FP4, FP6, FP8 及以上 |

### occupancy / 占用率
SM 上**活跃 warp 数**与 SM 最大可支持 warp 数之比。高占用率有助于隐藏内存延迟（通过切换 warp 填满等待周期）。
> *Higher occupancy helps hide memory latency by keeping the SM busy.*

### compute capability / 计算能力
NVIDIA GPU 的版本号，标识该 GPU 支持的 CUDA 特性集和硬件规格。major.minor 格式，相同 major 版本架构一致。
| 版本 | 架构 | 代表型号 |
|------|------|----------|
| 7.0 | Volta | V100 |
| 7.5 | Turing | RTX 20 系列, T4 |
| 8.0 | Ampere | A100 |
| 8.6 | Ampere | RTX 30 系列, A10 |
| 9.0 | Hopper | H100, H200 |
| 10.0 | Blackwell | B200, GB200 |

---

## 性能优化（Performance Optimization）

### warp divergence / warp 发散（分支发散）
同一 warp 内的线程走不同的控制流分支（if/else）时，未进入当前分支的线程被**掩蔽（masked off）**，只有活跃线程执行当前指令，导致 GPU 利用率下降。
> *When different threads in a warp follow different code paths, this is sometimes called warp divergence. GPU utilization is maximized when threads within a warp follow the same control flow path.*

### masked off / 掩蔽
warp 内因分支发散而暂时不执行当前指令的线程状态，这些 lane 的功能单元在此期间空闲。

### memory coalescing / 内存合并访问
同一 warp 中的线程访问**连续内存地址**时，硬件将多次访问合并为一次（或少数几次）内存事务，最大化 global memory 带宽利用率。反之则产生多次事务，带宽效率下降。
> *Understanding warp execution is helpful for understanding global memory coalescing.*

### shared memory access pattern / 共享内存访问模式
Shared memory 分为若干 bank，同一 warp 中多个线程访问**同一 bank 的不同地址**会产生 bank conflict，导致串行化，降低性能。

### bank conflict / bank 冲突
多个线程同时访问 shared memory 的同一 bank（但不同地址），导致访问串行化，降低吞吐量。

### register spilling / 寄存器溢出
每个线程的寄存器需求超出 SM 寄存器堆可分配量时，数据溢出到 local memory（物理上是 global memory），延迟极高，严重影响性能。

### ILP (Instruction-Level Parallelism) / 指令级并行
通过流水线重叠执行多条互不依赖的指令，隐藏功能单元延迟。

### TLP (Thread-Level Parallelism) / 线程级并行
通过在 SM 上同时调度大量活跃 warp，在一个 warp 等待内存时切换到另一个 warp，从而隐藏内存延迟。

### latency hiding / 延迟隐藏
GPU 通过高 TLP（大量活跃 warp）掩盖内存或指令延迟的机制，与 CPU 的深缓存/乱序执行策略不同。

### async copy / 异步拷贝
CUDA 11+ 引入（`cuda::memcpy_async`），允许线程在等待 global→shared memory 数据搬运时继续执行计算，实现计算与数据加载的重叠。

---

## 重要 API 概念

### cudaMalloc / cudaFree
在 GPU global memory 上分配/释放内存。
```cpp
cudaMalloc((void**)&d_ptr, size);  // 分配
cudaFree(d_ptr);                   // 释放
```

### cudaMemcpy
在 Host 与 Device 之间同步传输数据，调用时 CPU 会阻塞等待完成。
```cpp
// Host → Device
cudaMemcpy(d_ptr, h_ptr, size, cudaMemcpyHostToDevice);
// Device → Host
cudaMemcpy(h_ptr, d_ptr, size, cudaMemcpyDeviceToHost);
```

### cudaMemcpyAsync
cudaMemcpy 的异步版本，需指定 stream，可与其他操作重叠执行。

### stream / 流
CUDA 操作的有序序列。**同一 stream 内的操作顺序执行**，**不同 stream 之间可以并发**，常用于重叠计算与数据传输。默认 stream（NULL stream）与所有操作同步。
> *Using multiple streams enables overlapping computation with data transfer.*

### default stream / 默认流
也称 NULL stream 或 stream 0，默认情况下所有 kernel 和内存操作均在默认流中执行。

### event / 事件
用于精确 GPU 计时和 stream 间同步的对象。
```cpp
cudaEvent_t start, stop;
cudaEventCreate(&start);
cudaEventRecord(start, stream);   // 在 stream 中插入记录点
cudaEventSynchronize(stop);       // CPU 等待事件完成
cudaEventElapsedTime(&ms, start, stop);  // 获取耗时（毫秒）
```

### cudaDeviceSynchronize
阻塞 CPU 直到 GPU 上所有流的所有操作全部完成。常用于调试和计时。

### cooperative groups / 协作组
CUDA 9+ 引入，提供更灵活的线程协作原语。支持在 block、cluster、grid、多 GPU 等不同粒度定义协作范围，替代固定的 `__syncthreads()`。

### `__syncthreads()` / block 内屏障
block 内所有线程的同步屏障函数，确保调用点之前的共享内存写入对块内所有线程可见。

### CUDA Graphs / CUDA 图
将一系列 CUDA 操作（kernel 启动、内存拷贝等）定义为一个可重复执行的图结构，大幅减少 CPU 启动 kernel 的 overhead。

### Dynamic Parallelism / 动态并行层次
CUDA 特性，允许 GPU 上的 kernel 直接启动新的 kernel，无需 CPU 介入，适合自适应、递归算法。

---

## CUDA 生态工具

### NVCC (NVIDIA CUDA Compiler) / NVIDIA CUDA 编译器
CUDA C++ 代码的编译器，将 `.cu` 文件分离为 host 代码（转交系统 C++ 编译器）和 device 代码（编译为 PTX/SASS）。

### PTX (Parallel Thread Execution) / 并行线程执行指令集
NVIDIA GPU 的虚拟指令集架构（类似汇编语言）。NVCC 先编译为 PTX，再由驱动编译为特定 GPU 架构的 SASS。

### SASS (Shader ASSembly) / 着色器汇编
GPU 实际执行的本机指令，与具体 GPU 架构绑定（不可跨架构移植）。

### trtexec
TensorRT 毴带的命令行工具，用于模型转换、性能测试，不需写一行代码。

### Nsight Systems / Nsight 系统分析工具
NVIDIA GPU 应用性能分析工具，提供系统级时间线视图（CPU+GPU+内存传输）。

### Nsight Compute / Nsight 计算分析工具
NVIDIA GPU kernel 级性能分析工具，提供硬件计数器、内存带宽分析、warp 状态等详细数据。

### NVIDIA Warp / Warp（编程语言）
NVIDIA 开发的 Python 框架，编译到 CUDA 上运行，提供比 CUDA C++ 更高层次的 GPU 编程抗抽象。注意与 CUDA 执行单元 warp 区分。

### OpenAI Triton / Triton
OpenAI 开发的 GPU 编程语言，编译到 CUDA PTX 执行，PyTorch 2.0 的 `torch.compile` 底层使用。

---

## C++ 编程基础（Part 2：函数与变量限定符）

> 来源：2.1 Intro to CUDA C++，2.3 Writing SIMT Kernels（May 2026）

### `__global__` / kernel 入口限定符
声明一个 **kernel 函数**：由 host 用 `<<<...>>>` 启动，在 device（GPU）上执行。返回类型必须是 `void`。
```cpp
__global__ void vecAdd(float* A, float* B, float* C, int n) { ... }
```

### `__device__` / 设备函数限定符
声明只能在 GPU 上**编译和调用**的函数或变量。只能从 `__global__` 或其他 `__device__` 函数调用。
```cpp
__device__ float square(float x) { return x * x; }
```

### `__host__` / 主机函数限定符
声明只在 CPU 上编译和执行的函数（未标注时也是默认行为）。可与 `__device__` 组合，让编译器同时生成 CPU 和 GPU 版本。
```cpp
__host__ __device__ float relu(float x) { return x > 0 ? x : 0; }
```

### `__shared__` / 共享内存变量限定符
将变量分配在 **shared memory** 中。有两种分配方式：
- **静态**：大小在编译时确定 → `__shared__ float arr[1024];`
- **动态**：运行时通过 launch 第三个参数传入大小 → `extern __shared__ float arr[];`，kernel launch 时 `myKernel<<<g, b, bytes>>>()`

### `__constant__` / 常量内存变量限定符
将变量存储在 **constant memory** 中（64 KB，只读，grid 生命周期，带专用缓存广播）。
```cpp
__constant__ float coeffs[4];
// host 端写入：
cudaMemcpyToSymbol(coeffs, h_coeffs, sizeof(h_coeffs));
```

### `__managed__` / 统一内存变量限定符
将全局变量声明为 **managed memory**（unified memory），CPU 和 GPU 均可直接访问，驱动自动管理迁移。
```cpp
__managed__ int globalCounter;
```

---

## 线程索引内建变量（Built-in Thread Index Variables）

> 这些变量在 kernel 内部可直接使用，无需声明。

### `threadIdx` / 线程在 block 内的索引
3D 向量（`.x/.y/.z`），**zero-indexed**，表示线程在其所在 block 中的坐标。最常用 `.x` 做 1D 线性化。
> *`threadIdx.x` will take on values from 0 up to and including `blockDim.x - 1`.*

### `blockIdx` / block 在 grid 内的索引
3D 向量，标识当前线程所属 block 在 grid 中的坐标，zero-indexed。

### `blockDim` / block 的维度
3D 向量，表示 block 在各维度的线程数，由 launch config 决定，block 内所有线程值相同。

### `gridDim` / grid 的维度
3D 向量，表示 grid 在各维度的 block 数，由 launch config 决定，所有线程值相同。

### `dim3` / 三维整数向量类型
CUDA 定义的辅助类型，用于在 `<<<...>>>` 中指定多维 grid 和 block 大小。未指定的维度默认为 1。
```cpp
dim3 grid(16, 16);    // 16×16 blocks
dim3 block(8, 8);     // 8×8 threads/block
MatAdd<<<grid, block>>>(A, B, C);
```

---

## Kernel 启动（Kernel Launch）

### triple chevron notation / 三角括号启动语法
CUDA C++ 扩展语法，用于从 host 启动 kernel：
```
kernelName<<<gridDim, blockDim, sharedMemBytes, stream>>>(args...);
```
| 参数 | 类型 | 说明 | 默认值 |
|------|------|------|--------|
| `gridDim` | `int` 或 `dim3` | grid 中 block 数量/维度 | 必填 |
| `blockDim` | `int` 或 `dim3` | block 中线程数量/维度 | 必填 |
| `sharedMemBytes` | `size_t` | 动态 shared memory 大小（字节） | 0 |
| `stream` | `cudaStream_t` | 执行该 kernel 的 stream | NULL stream |

> 启动是**异步**的，函数返回不代表 kernel 已执行完毕。

### bounds checking / 边界检查
当问题规模不是 block 大小整数倍时，启动多余线程，在 kernel 内用 `if` 判断越界：
```cpp
int idx = threadIdx.x + blockIdx.x * blockDim.x;
if (idx < vectorLength) { ... }
```
越界线程不执行任何工作，overhead 极小；但应避免启动全部线程均不工作的多余 block。

### `cudaLaunchKernelEx` / 扩展 kernel 启动 API
`<<<>>>` 语法的替代接口，支持更多配置（如 cluster 大小），适合需要 thread block cluster 等高级特性时使用。

---

## 内存类型详解（GPU Device Memory Spaces）

> 官方 Table 1 完整对照（来自 2.3.3 GPU Device Memory Spaces）

| 内存类型 | 作用域（Scope） | 生命周期（Lifetime） | 物理位置 |
|----------|---------------|-------------------|---------|
| Global | Grid（所有线程） | Application | Device DRAM |
| Constant | Grid（所有线程，只读） | Application | Device DRAM + 专用缓存 |
| Shared | Block（同一 block 内） | Kernel | SM 片上 |
| Local | Thread（线程私有） | Kernel | Device DRAM |
| Register | Thread（线程私有） | Kernel | SM 片上 |

### local memory / 局部内存
线程私有的存储空间，**逻辑上线程独享，物理上位于 global memory（DRAM）**。访问延迟与 global memory 相同。编译器在以下情况下会将变量分配到 local memory：
- 无法以常量索引访问的数组
- 占用过多寄存器空间的大结构体/数组
- 寄存器溢出（register spilling）时的溢出数据

> *The 'local' label refers to its logical scope, not its physical location.*

---

## 内存性能（Memory Performance）

> 来源：2.3.4 Memory Performance

### coalesced global memory access / 合并全局内存访问
Global memory 以 **32-byte 事务（transaction）** 为粒度。warp 内 32 线程的访问会被硬件合并（coalesce）：
- **最优情况**：32 线程访问连续 4-byte 地址 → 需 4 次事务 → 利用率 100%
- **最差情况**：32 线程访问地址间距 ≥ 32 字节 → 需 32 次事务 → 利用率 12.5%（128B有效 / 1024B传输）
- 实践原则：**让 `threadIdx.x` 相邻的线程访问相邻内存地址**
> *Ensuring proper coalescing of global memory accesses is one of the most important performance considerations.*

### shared memory bank / 共享内存 bank
Shared memory 分为 **32 个 bank**，连续 32-bit 字依次映射到各 bank（每 bank 每周期 32-bit 带宽）。
- **无冲突（推荐）**：warp 内各线程访问**不同 bank** → 并发执行，最高吞吐
- **bank conflict（性能问题）**：warp 内多线程访问**同一 bank 不同地址** → 串行化访问，性能下降
- **广播（例外）**：多线程访问同一 bank 的**相同地址** → 广播，无冲突

### bank conflict padding / Bank 冲突填充
消除 shared memory bank conflict 的常用技巧：对二维数组的列维度加 1：
```cpp
// 有 bank conflict：
__shared__ float arr[32][32];
// 消除 bank conflict：
__shared__ float arr[32][33];  // 加一列 padding
```
> 原因：padding 使各行起始地址错开，打破连续线程访问同一 bank 的规律。

### atomics / 原子函数
对 global memory 或 shared memory 地址执行**原子读-改-写**操作，保证多线程无锁并发安全。
- C++ 推荐用法：`cuda::atomic_ref<T, Scope>` + `fetch_add()` 等
- 标量：`atomicAdd`, `atomicMax`, `atomicCAS` 等（老式 API）
- 性能代价：原子操作强制线程序列化，应尽量减少使用频率
```cpp
cuda::atomic_ref<float, cuda::thread_scope_device> ref(*result);
ref.fetch_add(value);
```

---

## 异步执行（Asynchronous Execution）

> 来源：2.5 Asynchronous Execution

### CUDA stream / CUDA 流
操作的**有序工作队列**，同一 stream 内的操作**顺序执行**，不同 stream 之间**可以并发**。
- 创建/销毁：`cudaStreamCreate(&stream)` / `cudaStreamDestroy(stream)`
- 使用：`kernel<<<g, b, 0, stream>>>()` 或 `cudaMemcpyAsync(dst, src, size, kind, stream)`
> *A stream operates like a work-queue into which programs can add operations to be executed in order.*

### default stream / NULL stream
也称 stream 0。不指定 stream 的 kernel launch 和 `cudaMemcpy` 均进入该流。
- **blocking stream（默认）**：default stream 与所有 blocking stream 同步；default stream 中的操作会等待所有 blocking stream 完成后才执行，反之亦然
- 用 `cudaStreamCreateWithFlags(&s, cudaStreamNonBlocking)` 创建非阻塞流，解除与 default stream 的隐式同步

### blocking stream vs non-blocking stream / 阻塞流与非阻塞流
| 类型 | 创建方式 | 与 default stream 的关系 |
|------|---------|------------------------|
| blocking（默认） | `cudaStreamCreate()` | 双向同步：互相等待 |
| non-blocking | `cudaStreamCreateWithFlags(cudaStreamNonBlocking)` | 独立，不与 default stream 同步 |

### per-thread default stream / 每线程独立默认流
编译选项 `--default-stream per-thread`（或宏 `CUDA_API_PER_THREAD_DEFAULT_STREAM`）使每个 host 线程拥有独立的 default stream，避免多线程程序中 default stream 的隐式全局同步。

### CUDA event / CUDA 事件
插入 stream 中的**时间戳标记**，用于精确计时和 stream 间依赖控制。
```cpp
cudaEvent_t start, stop;
cudaEventCreate(&start);   cudaEventCreate(&stop);
cudaEventRecord(start, stream);
kernel<<<g, b, 0, stream>>>(...);
cudaEventRecord(stop, stream);
cudaStreamSynchronize(stream);          // 等待 stream 完成
cudaEventElapsedTime(&ms, start, stop); // 获取耗时（毫秒）
cudaEventDestroy(start);  cudaEventDestroy(stop);
```

### `cudaStreamSynchronize` / 流同步
阻塞 CPU 直到**指定 stream** 中所有操作完成。比 `cudaDeviceSynchronize` 粒度更细，推荐在多流程序中使用。

### `cudaStreamQuery` / 流状态轮询
非阻塞地检查 stream 是否已完成所有操作，返回 `cudaSuccess`（完成）或 `cudaErrorNotReady`（未完成）。

### `cudaStreamWaitEvent` / 跨流事件依赖
让某个 stream 等待另一个 event 完成后，再继续执行后续操作，实现**跨 stream 依赖关系**。
```cpp
cudaStreamWaitEvent(stream2, event, 0); // stream2 等待 event 完成后再执行后续操作
```

### `cudaLaunchHostFunc` / 流内 host 回调
在 stream 中插入 **host 端回调函数**，回调在 stream 到达该位置时由 CUDA 执行。回调期间 stream 暂停推进，回调内**不能调用任何 CUDA API**。
> 注：`cudaStreamAddCallback()` 已废弃，应改用 `cudaLaunchHostFunc`。

### CUDA Graphs / CUDA 图
将一系列 CUDA 操作（kernel、memcpy 等）构成的 **DAG（有向无环图）** 先定义（capture）、后反复高效执行，大幅减少每次启动的 CPU overhead。
- **三个阶段**：① capture（`cudaStreamBeginCapture/EndCapture`）→ ② instantiate（`cudaGraphInstantiate`）→ ③ launch（`cudaGraphLaunch`，可重复）
- **适用场景**：相同操作序列需重复执行数百/数千次的场景（如 inference loop）

### `cudaGetLastError` / 获取并清除错误
返回当前 host 线程的最后一个 CUDA 错误，**同时清除**错误状态。应在 kernel launch 后立即调用以捕获启动参数错误。

### `cudaPeekAtLastError` / 查看但不清除错误
返回当前错误状态，**不清除**，用于在不影响后续错误检测的前提下检查错误。

### `CUDA_LOG_FILE` / CUDA 错误日志环境变量
设置此环境变量后，CUDA 驱动将错误信息写入指定文件（或 stdout/stderr）。无需修改代码即可捕获驱动级错误。需 **NVIDIA Driver r570+**。

### `CUDA_LAUNCH_BLOCKING` / 同步启动调试变量
设置 `CUDA_LAUNCH_BLOCKING=1` 后，每次 kernel launch 后自动同步，便于定位异步错误的来源 kernel。**仅用于调试**，性能影响极大。

---

## 统一内存与系统内存（Unified and System Memory）

> 来源：2.6 Unified and System Memory

### Unified Virtual Address Space (UVA) / 统一虚拟地址空间
同一进程中，所有 host 内存和所有 GPU 的 global memory 共享**单一虚拟地址空间**，每个处理器占据不同地址范围。
- `cudaPointerGetAttributes()` 可查询指针属于哪个设备
- `cudaMemcpy` 传 `cudaMemcpyDefault` 可自动判断拷贝方向

### HMM (Heterogeneous Memory Management) / 异构内存管理
Linux 内核特性（kernel ≥ 6.1.24 / 6.2.11 / 6.3+），提供**软件一致性**的完整 unified memory 支持：**所有系统内存（含 malloc/mmap 分配的）自动成为 managed memory**，无需 `cudaMallocManaged`。
- 检测方式：`nvidia-smi -q | grep Addressing`，显示 `Addressing Mode : HMM` 则可用

### ATS (Address Translation Services) / 地址翻译服务
**硬件一致性**的完整 unified memory，要求 GPU 通过 **NVLink C2C**（Chip-to-Chip）连接到 NVIDIA CPU（Grace Hopper、Grace Blackwell 等）。
- 性能优于 HMM（硬件缓存一致性，无需软件页迁移）
- ATS 可用时自动禁用 HMM
- `cudaDevAttrPageableMemoryAccessUsesHostPageTables = 1` 表示 ATS 可用

### unified memory paradigm / 统一内存范式
四种范式取决于硬件和 OS：
| 范式 | 条件 | 特点 |
|------|------|------|
| Limited support | Windows / WSL / Tegra | 只有显式 `cudaMallocManaged` 内存受支持，GPU 运行时 CPU 不可访问 |
| Full explicit managed | Linux，无 HMM/ATS | 仅 `cudaMallocManaged` 等显式分配支持全功能 |
| Full software coherence | Linux + HMM | 所有系统内存为 managed memory，软件一致性 |
| Full hardware coherence | Grace Hopper/Blackwell + ATS | 所有系统内存为 managed memory，硬件一致性 |

### `cudaMallocManaged` / 分配托管内存
在支持 unified memory 的系统上分配 **managed memory**，CPU 和 GPU 均可直接使用该指针。驱动按需迁移数据页。
```cpp
float* data;
cudaMallocManaged(&data, N * sizeof(float));
// CPU 写入 → kernel 访问 → 数据自动迁移
cudaDeviceSynchronize();
// CPU 再次访问 → 数据自动迁移回
```

### `cudaMemAdvise` / 内存访问提示
向驱动提供 managed memory 的访问模式提示，帮助优化数据驻留位置和迁移策略（如 `cudaMemAdviseSetReadMostly`、`cudaMemAdviseSetPreferredLocation` 等）。

### `cudaMemPrefetchAsync` / 异步预取
提前将 managed memory 异步迁移到指定设备（CPU 或 GPU），在 kernel 执行前完成数据搬运，**与其他 GPU 工作重叠**。
```cpp
cudaMemPrefetchAsync(ptr, size, cudaCpuDeviceId, stream); // 预取到 CPU
cudaMemPrefetchAsync(ptr, size, deviceId, stream);         // 预取到 GPU
```

### page-locked host memory / 页锁定内存（pinned memory）
通过 CUDA API 分配的**不可被 OS 换出（swap）** 的 host 内存，是**异步内存拷贝的必要条件**，同时可提升同步拷贝速度。
- `cudaMallocHost` / `cudaHostAlloc`：分配 pinned memory
- `cudaFreeHost`：释放
- `cudaHostRegister`：将 `malloc` 等方式分配的普通内存注册为 page-locked

### `cudaHostRegister` / 注册现有内存为页锁定
将非 CUDA API 分配的内存（`malloc`、第三方库等）注册为 page-locked，使其可用于异步拷贝。在无 HMM/ATS 的系统上，还需用 `cudaHostGetDevicePointer` 获取可在 kernel 中使用的 device 侧指针。

### mapped memory / 映射内存（zero-copy memory）
通过 `cudaMallocHost` / `cudaHostRegister` 分配的 pinned memory 可直接映射给 GPU kernel 访问，kernel 访问时数据**不迁移，每次访问都经过 PCIe/NVLink**（高延迟低带宽）。
- 与 unified memory 的区别：**mapped memory 数据不迁移**，始终在 host；unified memory 会迁移到访问它的处理器旁
- 历史别名：zero-copy memory
- 适用场景：偶发访问、数据量极小；**不适合**作为 kernel 主要内存来源

---

## Part 3 — Advanced CUDA

> 来源：CUDA Programming Guide Part 3（chapters 3.1–3.4）

---

### 3.1 Advanced CUDA APIs

---

### `cudaLaunchConfig_t` / Kernel 启动配置结构体
`cudaLaunchKernelEx` 的参数结构体，扩展了 triple chevron `<<<>>>` 的四个参数（gridDim、blockDim、dynamicSmem、stream），允许附加零个或多个 `cudaLaunchAttribute`。

```cpp
cudaLaunchConfig_t config = {0};
config.gridDim   = numBlocks;
config.blockDim  = threadsPerBlock;
config.attrs     = attribute;
config.numAttrs  = 1;
cudaLaunchKernelEx(&config, my_kernel, arg1, arg2);
```

### `cudaLaunchAttribute` / Kernel 启动属性
附加在 `cudaLaunchConfig_t` 上的键值对，控制 kernel 启动的高级行为：

| 属性 ID | 用途 |
|---------|------|
| `cudaLaunchAttributeClusterDimension` | 指定 thread block cluster 尺寸（必须满足，≥ CC 9.0） |
| `cudaLaunchAttributePreferredClusterDimension` | 指定首选 cluster 尺寸（尽力满足，≥ CC 10.0） |
| `cudaLaunchAttributePreferredSharedMemoryCarveout` | 指定 shared memory 占比 |
| `cudaLaunchAttributeProgrammaticStreamSerialization` | 启用 PDL（程序化依赖 kernel 启动）|

### thread block cluster / 线程块簇
CC 9.0（Hopper）引入的可选线程层级，介于 thread block 与 grid 之间。同一 cluster 内的 blocks 被保证**同时调度在同一 GPC（Graphics Processing Cluster）上**，可通过 distributed shared memory 互相访问彼此的 shared memory，并在 cluster 范围内同步。

- 声明方式 1（编译期固定）：`__cluster_dims__(2, 1, 1) __global__ void kernel()`
- 声明方式 2（运行时配置）：用 `cudaLaunchKernelEx` + `cudaLaunchAttributeClusterDimension`
- `__block_size__((1024,1,1), (2,1,1))` 同时指定 block 尺寸和 cluster 尺寸，此时 `<<<>>>` 第一维变为 cluster 数而非 block 数

### `__cluster_dims__` / 编译期 cluster 尺寸注解
附加在 `__global__` kernel 函数上，固定每次启动必须使用的 cluster 尺寸。与 `cudaLaunchAttributeClusterDimension` 的区别：前者编译期固定，后者运行时可变。

### `__block_size__` / block+cluster 尺寸注解
同时指定 block 尺寸和 cluster 尺寸的 kernel 注解（CC 9.0+）。启用后 `<<<>>>` 语法中第一个参数表示 cluster 数量而非 block 数量。

### GPC (Graphics Processing Cluster) / 图形处理器簇
GPU 的中间层硬件结构，包含多个 SM。Thread block cluster 保证其成员 blocks 同时运行在同一 GPC 上，从而支持 distributed shared memory 低延迟访问。

### distributed shared memory / 分布式共享内存
CC 9.0+ 特性。同一 cluster 内的线程可以直接读写其他 block 的 shared memory（通过 `cluster.map_shared_rank()` 获取远端指针），延迟远低于 global memory，是 cluster 间数据共享的核心机制。

### stream priority / 流优先级
通过 `cudaStreamCreateWithPriority(stream, flags, priority)` 创建带优先级的 CUDA stream。GPU 调度器优先选择高优先级 stream 的任务，但**不抢占**已运行的低优先级任务。
- `cudaDeviceGetStreamPriorityRange(&leastPriority, &greatestPriority)` 获取设备支持的优先级范围
- 优先级值越小表示优先级越高（`greatestPriority ≤ leastPriority`，通常 greatest=-1, least=0）

### `cudaStreamNonBlocking` / 非阻塞流标志
用 `cudaStreamCreateWithFlags(stream, cudaStreamNonBlocking)` 创建的流不受 NULL stream（default stream）的隐式同步约束，可在 NULL stream 提交操作期间继续并发执行。推荐在多流并发场景下使用。

### implicit synchronization / 隐式同步
以下操作会强制**不同 stream 的命令串行化**（即使它们无显式依赖）：
- page-locked host memory 分配（`cudaMallocHost`）
- device memory 分配（`cudaMalloc`）
- device memory 清零（`cudaMemset`）
- 同设备内的 memory copy
- 任何提交到 NULL stream 的命令（非阻塞流除外）
- L1/shared memory 配置切换

### `cudaEventDisableTiming` / 禁用计时事件标志
`cudaEventCreateWithFlags(event, cudaEventDisableTiming)` 创建不携带时间戳的 event，性能优于普通 event。仅用于跨 stream 依赖表达时应使用此标志。

### PDL (Programmatic Dependent Launch) / 程序化依赖 kernel 启动
CC 9.0+ 特性。允许 primary kernel 和 secondary kernel 在同一 stream 中部分重叠执行：primary 可在完成依赖数据写入后提前触发 secondary 启动，secondary 可在执行独立工作后再等待 primary 完成。

三个组件：
1. **primary kernel** 调用 `cudaTriggerProgrammaticLaunchCompletion()` — 声明依赖数据已写入 global memory
2. **secondary kernel** 调用 `cudaGridDependencySynchronize()` — 等待所有 primary kernel 完成
3. secondary kernel 用 `cudaLaunchAttributeProgrammaticStreamSerialization` 属性启动

### `cudaTriggerProgrammaticLaunchCompletion()` / 触发程序化启动完成
PDL 中由 primary kernel 调用的设备端函数，通知依赖它的 secondary kernel 可以开始（不会阻塞 primary kernel 自身的后续工作）。

### `cudaGridDependencySynchronize()` / Grid 依赖同步
PDL 中由 secondary kernel 调用的设备端函数，阻塞直到所有依赖的 primary kernel 已调用 `cudaTriggerProgrammaticLaunchCompletion` 并将结果刷新到 global memory。

### `cudaMemcpyBatchAsync` / 批量异步内存拷贝
将多个（通常较小的）内存拷贝操作打包为一个批次提交，摊薄每次拷贝的 CPU/驱动 overhead，并允许运行时优化整批传输。

关键参数：
- `cudaMemcpyAttributes.srcAccessOrder`：`cudaMemcpySrcAccessOrderStream`（流序）/ `cudaMemcpySrcAccessOrderDuringApiCall`（ephemeral 指针）/ `cudaMemcpyAccessOrderAny`
- `srcLocHint` / `dstLocHint`：位置提示（device ID 或 NUMA node ID），帮助运行时选择最优传输路径
- `cudaMemcpyFlagPreferOverlapWithCompute`：提示使用 Copy Engine 与计算重叠（仅 Tegra）

### ephemeral pointer / 临时指针
`cudaMemcpyBatchAsync` 术语：生命周期仅在 API 调用期间有效的 host 指针（如栈上数组）。对此类指针必须设置 `srcAccessOrder = cudaMemcpySrcAccessOrderDuringApiCall`，令拷贝在 API 调用期间立即执行。

### `CUDA_DEVICE_MAX_CONNECTIONS` / 最大并发连接数环境变量
控制 GPU 与主机驱动间的并发连接通道数，增大此值可减少不同 stream 因共用底层资源而产生的**假依赖（false dependency）**导致的意外串行化。MPS 场景下默认值较小。

### `CUDA_MODULE_LOADING` / 模块加载策略环境变量
- `EAGER`：应用启动时一次性加载所有 CUDA 模块，消除首次 kernel 调用的延迟（适合延迟敏感应用）
- 默认（lazy）：按需加载；可通过初始化时的"warm-up" kernel 调用提前触发加载

---

### 3.2 Advanced Kernel Programming

---

### PTX (Parallel Thread Execution) / 并行线程执行 ISA
CUDA 的虚拟机指令集架构，是编译器（nvcc）将 C++/Python 编译到特定 GPU 架构二进制码的**中间层**。PTX 代码在驱动加载时 JIT 编译为设备二进制，从而实现跨代架构兼容。
- `cuda::ptx` 命名空间（libcu++）：提供直接映射到 PTX 指令的 C++ 函数
- Inline PTX：在 C++ kernel 中嵌入 PTX 汇编（类似 CPU inline asm），用于极致性能调优

### SIMT (Single Instruction, Multiple Thread) / 单指令多线程
NVIDIA GPU 的并行执行模型。一条指令同时控制 warp 内 32 个线程，与 CPU SIMD 的关键区别：
- SIMD 暴露向量宽度给软件；SIMT 对程序员隐藏 warp 细节，每线程表现为独立标量
- 线程可分支，但分支发散时 warp 串行执行各分支路径（warp divergence）
- SM 不做分支预测或投机执行

### branch divergence / 分支发散
warp 内线程因条件判断走不同路径（例如 `if (threadIdx.x < 16)`），GPU 串行执行每条路径，未走该路径的线程处于 inactive 状态。严重的 branch divergence 会显著降低 warp 执行效率（最坏情况下降至 1/32）。

### independent thread scheduling / 独立线程调度
CC 7.0（Volta）引入。每个线程拥有独立的**程序计数器（PC）和调用栈**，线程可在 sub-warp 粒度发散和重聚合，允许线程间互相等待而不死锁（CC < 7.0 的 warp-synchronous 代码假设 lockstep 执行，在 CC 7.0+ 上可能行为改变）。
- 需要更新旧代码：warp 内 reduction、mutex 等需改用 `__syncwarp()` 显式同步

### `__syncwarp()` / Warp 内同步
CC 7.0+ 显式 warp 内同步原语。确保 warp 内所有（或指定 mask 内的）线程到达同步点后再继续执行，替代旧代码中对 lockstep 执行的隐式依赖。

### hardware multithreading / 硬件多线程
SM 同时维护多个 warp 的执行上下文（寄存器、PC 等）全部在片上，warp 切换**零开销**。当一个 warp 因内存访问等待时，warp scheduler 立即切换到另一个就绪的 warp，这是 GPU 隐藏内存延迟的核心机制。

### warp scheduler / Warp 调度器
每个 SM 内的硬件调度单元，每个时钟周期从就绪 warp 中选取一个并发射指令。SM 可有多个 warp scheduler（例如 Hopper SM 有 4 个），同一时钟周期可发射多条指令。

### thread scope / 线程作用域
定义内存操作的可见性范围和同步能力范围（libcu++ `cuda::thread_scope_*`）：

| 作用域 | PTX 标记 | 可见性 | 一致性点 |
|--------|---------|--------|---------|
| `thread_scope_thread` | — | 本线程 | — |
| `thread_scope_block` | `.cta` | 同 block 线程 | L1 |
| `thread_scope_device` | `.gpu` | 同 GPU 所有线程 | L2 |
| `thread_scope_system` | `.sys` | 全系统（CPU + 多 GPU）| L2 + 互联缓存 |

cluster 作用域（`.cluster`）：同 cluster 内所有线程，一致性点为 L2。

### scoped atomic / 作用域原子操作
`cuda::atomic<T, Scope>` — libcu++ 提供的 C++ 标准兼容原子类型，结合**线程作用域**（可见性范围）和 **C++ memory_order**（内存排序语义）。
- `memory_order_relaxed`：仅保证原子性，无排序约束（计数器场景）
- `memory_order_acquire` / `release`：生产者-消费者模式的 happens-before 保证
- 性能原则：使用最窄作用域（block < device < system）、最弱 ordering

### `cuda::barrier` / 异步屏障
libcu++ 提供的 arrive-wait 两阶段屏障（CC 7.0+，CC 8.0+ 有硬件加速）：
- `bar.arrive()` — 非阻塞，通知 barrier 已到达，返回 token
- `bar.wait(token)` — 阻塞直到所有参与线程完成 arrive
- 允许线程在 arrive 和 wait 之间执行与 barrier 无关的独立工作（隐藏同步延迟）
- CC 8.0+ 支持 block 内**任意线程子集**的硬件加速同步（不局限于全 warp / 全 block）

### async barrier / arrive-wait 异步屏障 → 见 `cuda::barrier`

### `cuda::pipeline` / 异步流水线
libcu++ 提供的多阶段生产者-消费者同步对象，用于将异步内存拷贝（global → shared）组织成 double/multi-buffer 流水线，使计算与数据加载重叠：

```
producer_acquire → [提交异步拷贝] → producer_commit
consumer_wait   → [使用 shared memory 计算]  → consumer_release
```

Primitives API（`__pipeline_memcpy_async` / `__pipeline_commit` / `__pipeline_wait_prior`）提供等效但更轻量的接口，仅支持 global→shared 拷贝。

### LDGSTS / 全局到共享内存异步拷贝指令
CC 8.0（Ampere）引入的硬件异步拷贝指令，直接从 global memory 加载数据到 shared memory，**绕过寄存器**，减少寄存器压力和 SM 流水线占用。与 `cuda::barrier` 配合使用完成同步。

### TMA (Tensor Memory Accelerator) / 张量内存加速器
CC 9.0（Hopper）引入的专用 DMA 硬件单元，支持**大块多维张量**的异步拷贝（global ↔ shared memory，支持 cluster 间 distributed shared memory）。
- 一条 TMA 指令可传输整个 tile（如 128×128 矩阵），无需每线程发射拷贝指令
- 与 `cuda::barrier` / transaction barrier 协同同步
- 是 Hopper Transformer Engine 和 wgmma 指令的前置依赖

### STAS / 寄存器到分布式共享内存异步写指令
CC 9.0+ 指令，将寄存器数据小规模异步写入 distributed shared memory（cluster 内其他 block 的 shared memory），用于 cluster 内细粒度数据交换。

### `cudaFuncSetAttribute` / 设置 Kernel 函数属性
在 kernel 启动前设置 per-kernel 属性的运行时 API：
- `cudaFuncAttributePreferredSharedMemoryCarveout`：设置 shared memory 占 unified data cache 的比例（整数百分比或枚举），影响 L1/shared memory 平衡
- `cudaFuncAttributeMaxDynamicSharedMemorySize`：显式启用超过 48 KB 的 dynamic shared memory（架构特定，必须配合 `extern __shared__` 使用）

### `cudaSharedmemCarveoutMaxL1` / 最大化 L1 配置
`cudaFuncSetAttribute` 的枚举值，将 unified data cache 尽可能多分配给 L1 cache（适合 global memory 密集访问的 kernel）。对应枚举还有 `cudaSharedmemCarveoutMaxShared`（最大化 shared memory）和 `cudaSharedmemCarveoutDefault`（默认）。

### unified data cache / 统一数据缓存
SM 上 L1 cache 和 shared memory 共用的同一物理 SRAM。两者的容量比例可通过 `cudaFuncSetAttribute` 调整（carveout）。Hopper（CC 9.0）最大 shared memory 容量为 228 KB/SM，Blackwell（CC 10.0）为 100 KB/SM。

### `cudaFuncSetCacheConfig` / 旧版 L1/Shared 配置 API
旧 API，设置 hard requirement，不同配置的 kernel 交替执行时会触发[隐式同步](#implicit-synchronization)导致 launch 串行化。推荐改用 `cudaFuncSetAttribute`（hint 语义，避免不必要串行化）。

---

### 3.3 CUDA Driver API

---

### CUDA Driver API / CUDA 驱动 API
比 CUDA Runtime API 更低层的 C API，所有入口点前缀为 `cu`（如 `cuLaunchKernel`、`cuMemAlloc`）。
- 运行时 API 基于驱动 API 封装
- 提供更细粒度的 context/module 管理
- 某些高级特性（如 Virtual Memory Management）仅在驱动 API 中暴露
- 驱动 API 与运行时 API 可混用（`CUdeviceptr` 可与 `void*` 互转）

| 驱动 API 对象 | 类型 | 对应概念 |
|--------------|------|---------|
| Device | `CUdevice` | CUDA 设备 |
| Context | `CUcontext` | 类似 CPU 进程（独立地址空间） |
| Module | `CUmodule` | 类似动态库（DLL/SO） |
| Function | `CUfunction` | Kernel 函数句柄 |
| Heap memory | `CUdeviceptr` | 设备内存指针（整数类型） |
| Stream | `CUstream` | CUDA 流 |
| Event | `CUevent` | CUDA 事件 |

### CUDA context / CUDA 上下文
驱动 API 中的核心资源容器，类似 CPU 进程：包含独立地址空间、加载的 module、纹理引用等所有 GPU 资源。
- `cuCtxCreate()` — 创建并压入当前线程的 context 栈
- `cuCtxPopCurrent()` — 弹出（"浮动"context，可被其他线程 push）
- `cuCtxAttach()` / `cuCtxDetach()` — 引用计数，用于多库共享同一 context
- `cuDevicePrimaryCtxRetain()` — 从驱动 API 访问 Runtime API 创建的 primary context

### primary context / 主上下文
Runtime API 自动创建和管理的 CUDA context（每设备一个），通过 `cuDevicePrimaryCtxRetain()` 可从驱动 API 侧访问。Runtime API 初始化时绑定到此 context。

### CUDA module / CUDA 模块
驱动 API 中的动态加载代码包（类似 DLL），包含 kernel 函数、全局变量等。
- `cuModuleLoad(module, "file.ptx")` — 从 PTX 文件加载（驱动 JIT 编译）
- `cuModuleLoadDataEx(module, PTXCode, ...)` — 从内存加载 PTX，可解析编译错误
- `cuModuleGetFunction(func, module, "KernelName")` — 获取 kernel 函数句柄
- **为兼容未来架构，应始终加载 PTX 而非 CUBIN**（CUBIN 与架构绑定，PTX 由驱动 JIT 编译）

### `cuLaunchKernel` / 驱动 API Kernel 启动
驱动 API 的 kernel 启动函数，等效于 Runtime API 的 `<<<>>>` 语法，参数通过指针数组传递。

### JIT compilation / 即时编译
驱动在加载 PTX 模块时将其编译为当前 GPU 架构的二进制码（CUBIN）。使应用可在未来 GPU 上运行（无需重新编译应用），代价是首次加载时有编译延迟（可用 `CUDA_MODULE_LOADING=EAGER` 或 warm-up 消除关键路径延迟）。

---

### 3.4 Multi-GPU

---

### multi-GPU programming / 多 GPU 编程
单应用使用多个 GPU 设备以突破单卡算力/显存上限，核心模式包括：
- 单 host 线程驱动多 GPU（`cudaSetDevice` 切换）
- 多 host 线程各驱动一个 GPU
- 多进程各驱动一个 GPU（需 IPC）
- 多节点 NVLink 集群（需 NVSHMEM / MPI）

### `cudaGetDeviceCount` / 获取设备数量
枚举系统中 CUDA 设备总数，配合 `cudaGetDeviceProperties` 读取每个设备的属性（compute capability、内存大小、CPU affinity 等）。

### `cudaSetDevice` / 设置当前设备
将指定 GPU 设为当前 host 线程的活跃设备。之后的 `cudaMalloc`、kernel launch、stream/event 创建均绑定到该设备。默认设备为 device 0。

### peer-to-peer memory access / P2P 内存访问（点对点内存访问）
一个 GPU 的 kernel 直接读写另一个 GPU 的显存（通过 NVLink 或 PCIe），无需经过 host 中转。

- `cudaDeviceCanAccessPeer(result, dev, peerDev)` — 查询是否支持 P2P
- `cudaDeviceEnablePeerAccess(peerDev, 0)` — 开启 P2P（全局影响该 peer 的所有显存分配，有额外开销）
- 更细粒度替代：Virtual Memory Management API（仅在分配时请求 peer 访问权限）

### `cudaMemcpyPeer` / 跨 GPU 内存拷贝
在两个 GPU 的显存间进行数据拷贝，若已启用 P2P 则无需经过 host，直接走 NVLink/PCIe，性能显著优于 GPU→CPU→GPU 路径。异步版本：`cudaMemcpyPeerAsync`。

### false dependency (multi-GPU) / 假依赖（多 GPU）
不同 stream 的独立操作因共用底层硬件资源（传输通道等）而被意外串行化。可通过增大 `CUDA_DEVICE_MAX_CONNECTIONS` 减少发生概率。

### NCCL (NVIDIA Collective Communications Library) / NVIDIA 集合通信库
提供多 GPU 集合操作（AllReduce、Broadcast、Gather 等）的高层库，自动利用 NVLink / GPU-Direct RDMA 优化多 GPU/多节点通信。不属于 CUDA 核心 API，作为独立库使用。

### NVSHMEM / NVIDIA 共享内存库
基于 OpenSHMEM 语义的多 GPU 通信库，支持 GPU 线程直接对远端 GPU 内存进行 RDMA 操作（put/get），适合 fine-grained 多 GPU 通信（NVLink C2C / InfiniBand）。

### IPC (Inter-Process Communication) / 进程间通信（CUDA）
允许不同进程共享 GPU 内存或 CUDA event：
- `cudaIpcGetMemHandle` / `cudaIpcOpenMemHandle` — 导出/导入显存句柄
- Virtual Memory Management API 提供更灵活的跨进程/跨节点 IPC（支持 Linux 和 Windows）

### `cudaDeviceCanAccessPeer` / 查询 P2P 支持
返回两个设备间是否支持 peer-to-peer 内存访问（取决于 PCIe/NVLink 拓扑）。非 NVSwitch 系统上每个设备最多支持 8 个 peer 连接。

### IOMMU（Input-Output Memory Management Unit）/ 输入输出内存管理单元
Linux 裸机上若启用 IOMMU，会阻断 PCIe P2P 内存传输（silent corruption 风险）。解决方案：裸机上禁用 IOMMU；虚拟机（VM pass-through）应启用 IOMMU 并使用 VFIO 驱动。Windows 无此限制。

## 第4部分：CUDA Features（Part 4）

---

### `cudaMallocManaged()` / 统一内存分配
分配统一虚拟地址空间（UVA）内存，CPU 和 GPU 均可直接访问，由驱动自动迁移数据页。等效的声明方式：`__managed__` 关键字修饰全局变量。需 CC 3.0+。

### Unified Memory 硬件范式 / 统一内存硬件模式
4种范式：① Full HW coherent（Grace Hopper NVLink-C2C，零拷贝真正共享）；② HMM software coherent（x86 Linux，CPU 页表驱动迁移）；③ Managed memory only（无 HMM，仅 cudaMallocManaged）；④ Limited（无托管内存支持）。

### `cudaMemPrefetchAsync` / 异步预取
提前将 Unified Memory 页迁移到目标设备，避免 page fault 带来的延迟。参数：ptr、size、dstDevice、stream。

### `cudaMemAdvise` / 内存访问建议
向驱动提供访问提示以优化迁移策略：`cudaMemAdviseSetReadMostly`（多设备只读副本）、`cudaMemAdviseSetPreferredLocation`（首选位置）、`cudaMemAdviseSetAccessedBy`（映射远端内存，避免迁移）。

### `cudaMemDiscardBatchAsync` / 批量丢弃（CUDA 12.4+）
丢弃一批 Unified Memory 范围的物理内存，触发后续访问重新分配，适合释放不再需要的内存页以降低内存压力。

### Access Counter Migration / 访问计数器迁移（CUDA 12.4+）
GPU 硬件统计对远端内存的访问频率，超过阈值后自动迁移数据到本地，减少跨设备访问延迟。需要支持 Access Counters 的硬件（Volta+）。

### GPU Memory Oversubscription / GPU 内存超额订阅
Unified Memory 允许数据集总大小超过单 GPU 物理显存，驱动通过按需换入/换出（demand paging）透明管理。

---

### CUDA Graph / CUDA 图
将一系列 GPU 操作（kernel、memcpy、memset 等）表示为有向无环图（DAG），一次实例化后可多次重放，消除重复 CPU launch 开销。三阶段：definition → instantiation（`cudaGraphInstantiate`）→ execution（`cudaGraphLaunch`）。

### Graph Node Types / 图节点类型
kernel node / CPU function node / memcpy node / memset node / empty node / event record/wait node / conditional node / memory alloc/free node / child graph node。

### Stream Capture / 流捕获
通过 `cudaStreamBeginCapture` + `cudaStreamEndCapture` 将流中的操作自动记录为图，无需手动构建 DAG。

### Edge Data（CUDA 12.3）/ 图边数据
图依赖边携带额外元数据：port（出/入端口）+ type（默认顺序依赖或 Programmatic 依赖），用于支持 PDL 等高级同步模式。

### `cudaGraphExecUpdate` / 图执行对象更新
在不重新实例化的前提下更新已实例化图（cudaGraphExec_t）中的节点参数（如 kernel 参数、memcpy 地址），避免重新实例化的开销。

### Conditional Node / 条件节点
图中的 IF / WHILE / SWITCH 节点，device 端通过 `cudaGraphSetConditional()` 动态控制子图是否执行或循环次数。

### Graph Memory Nodes / 图内存节点
图管理的虚拟地址固定、物理内存可在图执行间复用的 alloc/free 节点，降低显存碎片和分配开销。

### Device Graph Launch / 设备端图启动
从 device 代码中启动图：fire-and-forget（独立执行）、tail launch（替换当前图继续执行）、sibling launch（并行子图），需 CC 9.0+。

### CUDA User Objects / 用户对象
对 CPU 端资源（如 C++ 对象）进行引用计数的机制，绑定到图或图执行对象，确保资源在图执行期间不被释放。

---

### `cudaMallocAsync` / `cudaFreeAsync` / 流有序内存分配
CUDA 11.2+，在 stream 中异步分配/释放内存，语义与 stream 操作顺序一致。来自 stream-ordered memory allocator，底层复用内存池减少系统调用。

### Memory Pool / 内存池（Stream-Ordered）
`cudaMemPoolCreate` 创建显式内存池；default pool 每设备自动存在。`cudaMemPoolAttrReleaseThreshold=UINT64_MAX` 禁止自动归还给 OS。3种复用策略：FollowEventDependencies / AllowOpportunistic / AllowInternalDependencies。

---

### Cooperative Groups / 协作组（CG）
CC 6.0+ 引入的可编程线程协作抽象，支持灵活定义协作粒度。隐式组：`this_thread_block()`、`this_grid()`、`coalesced_threads()`、`this_cluster()`（CC 9.0+）。

### `tiled_partition<N>` / 分块分区
将线程块分割为固定大小（N = 2 的幂次，≤32）的 tile，每个 tile 内可独立 sync/reduce/scan。

### `cudaLaunchCooperativeKernel` / 协作内核启动
启动支持全网格同步的 kernel，所有 block 可通过 `this_grid().sync()` 全局同步。需 CC 6.0+，且 grid 大小不能超过设备并发 block 数。

---

### Programmatic Dependent Launch (PDL) / 程序化依赖启动
CC 9.0+，允许 secondary kernel 在 primary kernel 完成前提前调度到 SM，overlap 通信与计算。Primary 调用 `cudaTriggerProgrammaticLaunchCompletion()`，secondary 调用 `cudaGridDependencySynchronize()`。并发为机会性，不保证。

---

### Green Contexts / 绿色上下文（CUDA 13.1+）
将设备 SM 和 work queue 静态划分，为延迟敏感任务提供资源隔离保证。4步创建：`cudaDeviceGetDevResource` → `cudaDevSmResourceSplitByCount` → `cudaDevResourceGenerateDesc` → `cudaGreenCtxCreate`。比 MIG（进程级静态）更灵活，比 MPS（多进程动态%）更有隔离保证。

---

### Lazy Loading / 懒加载（CUDA 11.7+）
模块中的 kernel 和符号在首次使用时才加载，而非模块加载时全部加载，可大幅减少启动时间和内存占用。默认启用（12.3+ 所有平台）。环境变量：`CUDA_MODULE_LOADING=LAZY/EAGER`。查询：`cuModuleGetLoadingMode()`。含 managed variables 的模块仍 eager 加载。

### Lazy Loading Hazards / 懒加载注意事项
① 并发场景下首次加载可能导致死锁（需预加载）；② 大分配失败（建议预留 buffer）；③ 性能测量偏差（需 warmup 预热）。强制加载：`cuModuleGetFunction()` 或 `cudaFuncGetAttributes()`。

---

### Error Log Management / 错误日志管理（CUDA 12.9+）
将 CUDA API 错误以可读英文描述输出，补充原本仅有错误码的不足。激活：设置 `CUDA_LOG_FILE=stdout/stderr/path`。仅 Driver API 支持，Runtime API 待添加。

### Error Log Format / 错误日志格式
`[Time][TID][Source][Severity][API Entry Point] Message`。日志缓冲最多100条，`cuLogsDumpToMemory` 上限25600字节，仅美式英语。

### `cuLogsRegisterCallback` / 日志回调注册
注册回调函数，每当生成错误日志时触发，签名：`void cb(void *data, CUlogLevel logLevel, char *message, size_t length)`。对应反注册：`cuLogsUnregisterCallback`。

### `cuLogsCurrent` / `cuLogsDumpToFile` / `cuLogsDumpToMemory`
日志迭代器和导出 API：`cuLogsCurrent` 获取当前日志末尾位置；`cuLogsDumpToFile` 导出到文件；`cuLogsDumpToMemory` 导出到内存缓冲。iterator 为 NULL 则导出全部。

---

### `cuda::barrier` / 异步屏障（Async Barrier）
`cuda::barrier<cuda::thread_scope_block>` 实现 block 内生产者-消费者双缓冲同步。`init(&bar, count)`；`arrive()` 返回 token；`wait(move(token))` 阻塞直到所有线程到达。`arrive_and_wait()` = `arrive()` + `wait()`；`arrive_and_drop()` 允许线程提前退出。

### Barrier Phase / 屏障相位
mbarrier 有奇偶两个相位交替切换，`mbarrier_try_wait_parity(bar, parity)` 用相位值等待，避免重用同一 barrier 时的竞争。

### Barrier Transaction Count / 屏障事务计数（CC 9.0+）
`barrier_arrive_tx` / `barrier_expect_tx`：为 TMA 等异步数据传输设置额外的"字节完成"计数，只有数据传输完成且线程全部到达后屏障才开放。

---

### `cuda::pipeline` / CUDA 管线
多级软件流水线抽象，隐藏内存延迟。`cuda::pipeline_shared_state<scope, stages_count>` 声明共享状态。两种模式：Unified（所有线程同时作为生产者和消费者）vs Partitioned（角色固定分工）。

### Pipeline Producer/Consumer API
生产者：`producer_acquire()` → `memcpy_async` → `producer_commit()`；消费者：`consumer_wait()` → 使用数据 → `consumer_release()`。`pipeline_consumer_wait_prior<N>()` 等待除最后 N 个 stage 外的所有 stage 完成。

### Warp Entanglement / Warp 纠缠
发散 warp 中各线程的 commit 会分散到不同 stage，导致 over-wait。解决方案：在 commit 前加 `__syncwarp()` 确保 warp 收敛。`pipeline::quit()` 可提前退出管线。

---

### LDGSTS（CC 8.0+）/ 全局到共享内存异步拷贝
`cp.async` 指令，element-wise 从 global memory 异步加载到 shared memory，不占用 register，不阻塞线程。支持4/8/16字节元素；16字节走 L1 bypass 模式。API：`cuda::memcpy_async`、`cooperative_groups::memcpy_async`、`__pipeline_memcpy_async`。`cuda::aligned_size_t<16>()` 触发最优路径。

### TMA（Tensor Memory Accelerator，CC 9.0+ Hopper）/ 张量内存加速器
SM 内专用硬件单元，执行 bulk 异步数据传输，支持1D到5D张量，主要用途：global↔shared memory 批量搬运，配合 mbarrier 完成同步。

### TMA 1D Bulk-Async / TMA 一维批量异步
无需 tensor map，直接传大块数据：`cuda::device::memcpy_async_tx` + `barrier_arrive_tx/expect_tx`；或 `cuda::ptx::cp_async_bulk`。对齐要求：global 16B，shared 16B，size 为16B倍数。

### CUtensorMap / 张量映射描述符
多维 TMA 传输所需的元数据描述符，通过 Driver API `cuTensorMapEncodeTiled` 创建（需 `cudaGetDriverEntryPointByVersion` 获取函数指针）。推荐以 `const __grid_constant__ CUtensorMap` 参数传递给 kernel。

### TMA Swizzle / TMA 交错模式
`CU_TENSOR_MAP_SWIZZLE_128B/64B/32B/NONE`，对 shared memory 布局进行交错以消除 bank conflict，128B swizzle 提供最优带宽。

### TMA Multicast / TMA 多播（CC 9.0+ sm_90a）
`cp_async_bulk_tensor multicast`：将同一数据块广播到 cluster 内多个 block 的 shared memory，减少重复传输。仅 sm_90a 有最优硬件路径。

### STAS（Store Async，CC 9.0+）/ 异步存储到分布式共享内存
`cuda::ptx::st_async`：将 register 数据异步存储到 cluster 内其他 block 的 shared memory（distributed shared memory），支持4/8/16字节。通过 mbarrier 确认完成。适用 cluster 内 ring pipeline。

---

### Cluster Launch Control（CC 10.0+ Blackwell）/ 集群启动控制
动态负载均衡机制，允许 kernel 取消尚未启动的 block cluster，实现 work stealing。`ptx::clusterlaunchcontrol_try_cancel` 提交取消请求（async proxy operation），通过 mbarrier 等待结果，`clusterlaunchcontrol_query_cancel_is_canceled` 解码是否成功。

---

### L2 Cache Persistence / L2 持久化缓存（CC 8.0+，CUDA 11.0）
`cudaDeviceSetLimit(cudaLimitPersistingL2CacheSize, size)` 预留最多75% L2 作为持久化区域。通过 `cudaStreamSetAttribute` 设置 `accessPolicyWindow`：`hitRatio`（持久化比例0.0-1.0）、`hitProp`（Persisting/Streaming/Normal）。重置：`num_bytes=0` + `cudaCtxResetPersistingL2Cache()`。

---

### Memory Synchronization Domains / 内存同步域（CC 9.0+ Hopper，CUDA 12.0）
将 kernel 分配到不同的物理同步域（最多4个），减少跨域 fence 的范围，提升多租户场景吞吐。逻辑域：Default / Remote；`cudaLaunchAttributeMemSyncDomainMap` 将逻辑域映射到物理域。跨域访问需 system-scope fence；同域内 device-scope 即可。


---

### CUDA IPC / 进程间通信（Legacy，仅 Linux）
cudaIpcGetMemHandle 导出显存句柄，cudaIpcOpenMemHandle 在另一进程导入为设备指针。不支持 cudaMallocManaged。建议2MiB对齐分配防止子分配信息泄露。Tegra 平台仅支持 event sharing，不支持 memory sharing。

### VMM IPC / 虚拟内存管理 IPC（跨平台）
通过 Virtual Memory Management Driver API 创建可跨进程/跨平台共享的内存对象，支持 Linux 和 Windows，比 Legacy IPC 更灵活，支持多节点 fabric handles。

---

### Virtual Memory Management (VMM) / 虚拟内存管理（Driver API）
Driver API 提供细粒度显存控制：cuMemCreate（物理内存）→ cuMemAddressReserve（虚地址）→ cuMemMap（映射）→ cuMemSetAccess（设置访问权限）→ cuMemUnmap → cuMemRelease → cuMemAddressFree。释放必须按顺序。

### Fabric Memory（CUDA 12.4+）/ Fabric 内存
CU_MEM_HANDLE_TYPE_FABRIC：通过 NVLink NVSwitch 实现跨节点 GPU 内存共享。需 IMEX channel（
vidia-caps-imex-channels）实现多用户安全隔离。

### Multicast Object / 多播对象（VMM）
cuMulticastCreate + cuMulticastAddDevice + cuMulticastBindMem：将同一物理内存映射到多个 GPU 的不同虚地址，写入一次多 GPU 同时可见，配合 multimem PTX 指令。

### Compressible Memory / 可压缩内存（VMM）
CU_MEM_ALLOCATION_COMP_GENERIC：分配硬件压缩内存，对带宽敏感的访问模式可提升有效带宽。

### Virtual Aliasing / 虚拟别名（VMM）
同一物理内存映射到多个虚拟地址别名，需在访问前调用 cuda::ptx::fence_proxy_alias() 确保一致性。

---

### Extended GPU Memory (EGM) / 扩展 GPU 内存
NVLink-C2C 系统（如 Grace Hopper）中，GPU 可高带宽访问 CPU NUMA 节点内存。通过 VMM：prop.location.type = CU_MEM_LOCATION_TYPE_HOST_NUMA + numaId；或通过 Memory Pool：cudaMemLocationTypeHostNuma。多节点：CU_MEM_HANDLE_TYPE_FABRIC + TCP/IP 传递句柄。

---

### CUDA Dynamic Parallelism (CDP2) / CUDA 动态并行（第2版）
CUDA 12.0+ 默认，CC 9.0+ 唯一可用版本。设备端直接启动子 kernel：child_kernel<<<g,b,0,cudaStreamFireAndForget>>>() 或 cudaStreamTailLaunch。子 kernel 与父 kernel 共享 L2/全局内存，但有独立的 shared memory 和 register。

### CDP2 Stream Semantics / CDP2 流语义
cudaStreamFireAndForget：子 kernel 独立执行，父不等待；cudaStreamTailLaunch：父 kernel 退出后子 kernel 接替在同一 SM 上执行（tail call 优化），节省资源。

---

### Graphics Interoperability / 图形互操作
CUDA 直接访问 OpenGL/Direct3D 资源的机制。主要流程：Register → Map → 获取设备指针/array → kernel 使用 → Unmap → Unregister。注册代价高，建议每资源只注册一次。

### OpenGL Interop / OpenGL 互操作
cudaGraphicsGLRegisterBuffer（buffer object → 设备指针）；cudaGraphicsGLRegisterImage（texture/renderbuffer → CUDA array）。OpenGL context 必须为当前线程的 current context。Bindless texture 不可注册。

### Direct3D Interop / Direct3D 互操作
支持 D3D9/D3D10/D3D11（不含 D3D12）直接映射；D3D12 通过 External Resource Interop 导入。cudaGraphicsD3D11RegisterResource 注册 D3D11 资源。D3D11 设备必须用 D3D_DRIVER_TYPE_HARDWARE 创建。

### External Resource Interoperability / 外部资源互操作
通过 OS 级句柄（Linux fd / Windows NT handle）在 CUDA 与 Vulkan/D3D11/D3D12/NvSciSync 之间共享内存和同步对象。cudaImportExternalMemory + cudaExternalMemoryGetMappedBuffer/MipmappedArray；cudaImportExternalSemaphore + cudaSignalExternalSemaphoresAsync/cudaWaitExternalSemaphoresAsync。

### Vulkan Interop / Vulkan 互操作
通过 UUID 匹配 CUDA 设备与 Vulkan 物理设备；导出 Vulkan 内存/信号量（VK_KHR_external_memory/semaphore）；在 CUDA 中 import 并映射。Timeline semaphore（64位计数器）比 Binary semaphore 更灵活，可用同一对象表达执行顺序。

### NvSciBuf / NvSciSync Interop / NVIDIA SCI 接口互操作
DRIVE OS 平台专用：NvSciBuf 共享缓冲区，NvSciSync 共享同步对象。cudaDeviceGetNvSciSyncAttributes 获取兼容属性列表；通过 cudaImportExternalMemory/Semaphore 导入到 CUDA。

---

### cuGetProcAddress / Driver 函数地址查询
运行时获取指定版本 CUDA Driver API 函数指针，支持版本化 ABI（_v* 后缀）。用法：cuGetProcAddress("cuFoo", &pfn, CUDA_VERSION, flags, &driverStatus)。失败状态：CU_GET_PROC_ADDRESS_VERSION_NOT_SUFFICIENT（驱动版本不够）或 CU_GET_PROC_ADDRESS_SYMBOL_NOT_FOUND（符号不存在）。

### cudaGetDriverEntryPointByVersion / Runtime API 获取 Driver 函数指针
Runtime API 版本的 cuGetProcAddress，用于在旧 toolkit 上访问新 driver 功能，或请求 per-thread default stream 版本（cudaEnablePerThreadDefaultStream）。

### Per-thread Default Stream Version / 每线程默认流版本
Driver API 中 _ptsz/_ptds 后缀的函数变体，具有 per-thread default stream 语义，避免隐式全局 stream 同步。通过 CU_GET_PROC_ADDRESS_PER_THREAD_DEFAULT_STREAM flag 获取，或编译时定义 CUDA_API_PER_THREAD_DEFAULT_STREAM。
---

## Best Practices Guide (BPG) 词汇表

### APOD / Assess-Parallelize-Optimize-Deploy 循环
CUDA 开发方法论四步循环。Assess（分析热点）→ Parallelize（并行化）→ Optimize（优化）→ Deploy（上线）。每轮迭代逐步提升性能，避免过早优化。

### Amdahl's Law / 阿姆达尔定律
并行加速上限公式：$S = 1 / (1 - P + P/N)$，P 为可并行部分比例，N 为处理器数。强调即使少量串行代码也会严重限制加速比。与之互补的是 Gustafson 定律（弱扩展）。

### Strong Scaling / 强扩展
固定问题规模，增加处理器数量以缩短时间。受 Amdahl 定律约束，串行部分成为瓶颈。

### Weak Scaling / 弱扩展
处理器数量增加时同比扩大问题规模，总时间保持不变。遵循 Gustafson 定律，扩展性更好。

### Hotspot / 热点
程序中占用最多执行时间的函数或代码段。APOD 第一步用 profiler（如 gprof、Nsight）识别热点，集中优化精力。

---

### Theoretical Bandwidth / 理论带宽
硬件规格推算的峰值内存带宽。公式：频率(Hz) × 总线宽度(B) × 传输倍率 / 1e9 = GB/s。例：V100 HBM2 = 0.877GHz × 512B × 2 = 898 GB/s。

### Effective Bandwidth / 有效带宽
实际测量值：(读写字节数) / 执行时间。与理论带宽对比可量化内存优化空间。

### Memory-Bound / 内存瓶颈
kernel 性能受限于内存带宽而非计算吞吐量。此时应优先优化内存访问模式（coalescing、shared memory）。

### Compute-Bound / 计算瓶颈
kernel 性能受限于 ALU 吞吐量。此时应优化指令效率（减少 divergence、使用快速数学函数）。

---

### Pinned Memory / 页锁定内存
`cudaHostAlloc` / `cudaHostRegister` 分配的不可换出主机内存。PCIe 传输峰值可达 ~12 GB/s（x16 Gen3），避免内核分页开销。代价：系统可用内存减少，分配开销大。

### Zero-Copy Memory / 零拷贝内存
`cudaHostAllocMapped` 将主机内存直接映射到 GPU 地址空间，kernel 通过 `cudaHostGetDevicePointer` 获取设备指针直接访问。集成 GPU（integrated=1）上总是有收益；离散 GPU 上仅适用于只读写一次的场景。

### Unified Virtual Addressing (UVA) / 统一虚拟寻址
CUDA 4.0+ 单地址空间，CPU 和所有 GPU 共享虚拟地址范围。`cudaHostAlloc` 的主机/设备指针相同，无需 `cudaHostGetDevicePointer`。P2P 传输的前提条件。

### Async Data Transfer / 异步数据传输
`cudaMemcpyAsync` + pinned memory + 非默认 stream，使 H2D/D2H 传输与 kernel 计算重叠。需要双副本（ping-pong）buffer 或分段（staged）传输策略。

### Staged Concurrent Copy / 分段并发拷贝
将数据分为 N 段，用 N 个 stream 交错执行 memcpy + kernel，总时间从 tT+tE 降至约 max(tT,tE) + min(tT,tE)/N。

---

### Coalesced Access / 合并访问
同一 warp 的 32 线程访问连续对齐的全局内存地址，合并为最少的 32B 事务。CC 6.0+ 标准：warp 访问所需 32B 段数 = 事务数。反例：stride-2 导致 50% 带宽浪费；随机访问最差。

### Misaligned Access / 非对齐访问
warp 访问的地址不按 32B 对齐，产生额外事务（如跨越 5 个 32B 段而非 4 个）。`cudaMalloc` 保证至少 256B 对齐，选 blockDim 为 warpSize 倍数可避免块间非对齐。

### Strided Access / 跨步访问
线程访问步长 > 1 的内存，有效带宽随步长增大线性下降（stride=2 → 50%，stride=32 → ~3%）。常见解决方案：用 shared memory 做转置后再访问。

### Bank Conflict / Bank 冲突
Shared memory 有 32 个 bank（4B/bank），同一 warp 中多线程访问同一 bank 的不同地址时串行化。填充（padding）：声明 `float s[32][33]` 而非 `[32][32]` 消除转置时的冲突。广播（broadcast）例外：多线程访问同一地址不产生冲突。

### L2 Persistent Cache / L2 持久化缓存
CC 8.0+（CUDA 11.0+）可将 L2 的一部分（最多 75%）预留为持久化访问窗口。高频访问数据驻留 L2 减少 DRAM 带宽压力。通过 `cudaDeviceSetLimit(cudaLimitPersistingL2CacheSize, bytes)` + `cudaStreamSetAttribute` 的 `accessPolicyWindow` 配置。

### Local Memory / 局部内存
名为"本地"但实际在片外（物理上是全局内存）。编译器在寄存器不足时自动将大数组/结构体溢出到 local memory。访问延迟与 global memory 相同（~数百周期）。用 `-ptxas-options=-v` 查看 `lmem` 用量。

### Constant Memory / 常量内存
64KB 只读缓存内存。所有线程读同一地址时速度接近寄存器（cache hit）；读不同地址时按唯一地址数串行化。适用于广播给所有线程的参数、查找表。

### Texture Memory / 纹理内存
具有 2D 空间局部性缓存的只读内存，支持插值、归一化坐标、边界处理（clamp/wrap）。适用于具有 2D 访问模式的数据（图像处理等）。

---

### Occupancy / 占用率
SM 上活跃 warp 数与最大可容纳 warp 数的比值（0~100%）。低 occupancy 会限制延迟隐藏能力，导致性能下降。影响因素：register 用量、shared memory 用量、block 大小。工具：Nsight Compute 内置 Occupancy Calculator。

### Register Pressure / 寄存器压力
每线程使用的寄存器过多，导致单 SM 上可驻留的 block/warp 数减少，occupancy 下降。控制方法：`-maxrregcount=N`（文件级）或 `__launch_bounds__(maxTPB, minBPSM)`（kernel 级）。

### `__launch_bounds__` / 启动边界限定符
Kernel 级限定符，声明最大线程数/block 和最小 block 数/SM，引导编译器优化 register 分配以达到目标 occupancy。

### Thread Block Heuristics / Block 大小启发规则
- 最小 64 线程/block（多 block 并发时）
- 推荐 128~256 线程/block 作为初始实验值
- Block 大小应为 32（warpSize）的倍数
- 频繁调用 `__syncthreads()` 的 kernel 偏好较小 block

### Concurrent Kernel Execution / 并发 kernel 执行
多个非默认 stream 上的 kernel 在同一设备上同时执行，充分利用 SM 资源。前提：设备支持（`concurrentKernels` field）且使用非零 stream ID。

### Multiple Contexts / 多上下文
同一 GPU 上多个 CUDA context 时间片轮转执行，产生切换开销。推荐使用 Primary Context（`cuDevicePrimaryCtxRetain`）以避免不必要的 context 创建。

---

### Warp Divergence / Warp 分歧
同一 warp 内线程因条件分支走不同路径，各路径串行执行，吞吐量下降（最差降至 1/32）。优化：让条件按 warpSize 对齐（如 `threadIdx.x / warpSize`），或用 branch predication。

### Branch Predication / 分支预测化
编译器将短 if-else 转换为带 predicate 的指令序列（所有指令都调度，false predicate 的指令不写结果）。避免实际分支跳转，适用于受控指令数 ≤ 阈值的情形。`#pragma unroll` 配合使用可消除循环分支。

### `#pragma unroll` / 循环展开指令
nvcc 编译器 pragma，手动指定展开因子或完全展开循环，减少分支开销、提升指令级并行。

---

### FMA (Fused Multiply-Add) / 融合乘加
单条指令完成 a×b+c，避免中间舍入，精度和性能均优于分开执行。CUDA 编译器默认启用 FMA；`-ftz=true` 等 flag 进一步影响浮点行为。

### Fast Math Intrinsics / 快速数学内置函数
`__sinf`、`__cosf`、`__expf`、`__logf`、`__fdividef` 等直接映射到硬件指令，比标准库函数快但精度较低（约 2 ULP vs 0.5 ULP）。`-use_fast_math` 全局启用（等价于 `-ftz=true -prec-div=false -prec-sqrt=false`）。

### `sincos` / `sincosf` / 正弦余弦同步计算
同时需要 sin 和 cos 时使用 `sincosf(x, &s, &c)` 而非分别调用，单次计算两个值，性能提升约 2 倍。

### `rsqrtf` / 倒数平方根
`1.0f / sqrtf(x)` 的快速近似版本。仅当 `-prec-div=false` 且 `-prec-sqrt=false` 时编译器才自动转换；建议直接调用 `rsqrtf()`。

### Integer Division / Modulo 优化
整除和取模是高开销运算。若除数为 2 的幂：`i/n` → `i >> log2(n)`，`i%n` → `i & (n-1)`。编译器对字面量常数自动优化，变量除数需手动处理。

### Signed Loop Counter / 有符号循环计数器
C 标准中无符号整数溢出行为有定义，但有符号溢出是 UB，编译器可更激进优化。循环计数器即使恒正也应用 `int` 而非 `unsigned int`，以获得略好的性能。

### `exp2f` / `exp10f` vs `powf` / 指数函数优化
以 2 或 10 为底的指数应用 `exp2f`/`exp10f` 而非 `powf(2, x)`，速度可快 10 倍。1/3 次方用 `cbrtf` 而非 `powf(x, 1.0f/3.0f)`。

---

### Compute Capability / 计算能力
GPU 架构版本号（如 8.9、9.0），决定支持的指令集和硬件特性。高版本向下兼容。通过 `cudaGetDeviceProperties` 查询 `major.minor`。

### PTX (Parallel Thread Execution) / 并行线程执行汇编
CUDA 的虚拟 ISA，介于源码和机器码之间。cubin 与架构绑定（X.y → X.z≥y 才可运行），PTX 可 JIT 编译到新架构（forward compatible）。`-gencode arch=compute_XX,code=compute_XX` 保留 PTX。

### cubin / CUDA Binary
特定 SM 架构的本地二进制。`-gencode arch=compute_XX,code=sm_XX` 生成。兼容规则：sm_X.y 仅运行在 X.z（z≥y）上。

### JIT Compilation / 即时编译
CUDA 驱动在运行时将 PTX 编译为目标设备的 cubin。一次编译多设备可运行，代价是首次运行时间增加（有 JIT cache 缓解）。

### CUDA Compatibility / CUDA 兼容性
- **Forward Compat（前向兼容）**：新 toolkit 编译的代码在旧 driver 上运行（CUDA 10+）
- **Enhanced Compat（增强兼容）**：11.1+ 语义版本，同 major 内 minor 版本二进制兼容
- **Binary Compat（二进制兼容）**：Driver API 接口保证向后兼容

### NVRTC / 运行时 CUDA 编译库
Runtime compilation library，接受 CUDA C++ 字符串源码，生成 PTX 或 cubin（11.1+）。用于动态生成 kernel、JIT 框架。

### nvidia-smi / NVIDIA System Management Interface
命令行 GPU 管理工具。查询：ECC 错误、利用率、温度、功耗、进程列表。设置：ECC 模式、计算模式、persistence mode。可输出 XML 或纯文本。

### NVML / NVIDIA Management Library
C API，提供与 nvidia-smi 相同的查询和控制功能，用于构建第三方管理工具。CUDA 8.0+ 随 toolkit 发布，向后兼容。

### `CUDA_VISIBLE_DEVICES` / 可见设备环境变量
逗号分隔的设备索引列表，控制应用程序可见的 GPU 子集。例：`CUDA_VISIBLE_DEVICES=0,2` 使应用看到设备 0 和 1（实际为系统 0 和 2）。

### Error Handling Pattern / 错误处理模式
所有 CUDA Runtime API 返回 `cudaError_t`；kernel launch（`<<<>>>`）返回 void，需在调用后立即检查 `cudaGetLastError()`。异步错误在下次同步时暴露（`cudaMemcpy`、`cudaDeviceSynchronize`）。

### Building for Maximum Compatibility / 多架构编译
用多组 `-gencode` 生成多架构 cubin + PTX，驱动运行时选择最优版本：
`-gencode arch=compute_80,code=sm_80 -gencode arch=compute_90,code=sm_90 -gencode arch=compute_90,code=compute_90`
最后一组 `code=compute_XX` 保留 PTX 以支持未来架构。

Volta 架构（CC 7.0+）引入，每个线程维护自己的 PC 和调用栈，允许 warp 内线程独立前进。但需在汇聚点用 `__syncwarp()` 显式同步。

---

## CUDA Runtime API Reference 词汇表

### 6.1 Device Management / 设备管理

#### cudaGetDeviceCount / 获取设备数量
`cudaGetDeviceCount(int* count)` — 返回系统中计算能力 ≥ 2.0 的 CUDA 设备总数。

#### cudaSetDevice / 设置当前设备
`cudaSetDevice(int device)` — 设置当前线程使用的 GPU 设备（device ID 从 0 起）。后续所有 CUDA 调用都针对此设备。

#### cudaGetDevice / 获取当前设备
`cudaGetDevice(int* device)` — 返回当前线程正在使用的设备 ID。

#### cudaGetDeviceProperties / 获取设备属性
`cudaGetDeviceProperties(cudaDeviceProp* prop, int device)` — 填充 `cudaDeviceProp` 结构体，含 SM 数、最大线程数、共享内存大小、计算能力等全部硬件信息。

#### cudaDeviceGetAttribute / 查询单项属性
`cudaDeviceGetAttribute(int* value, cudaDeviceAttr attr, int device)` — 查询单个设备属性（比 cudaGetDeviceProperties 更轻量）。

#### cudaDeviceReset / 设备重置
`cudaDeviceReset()` — 销毁当前设备上的所有分配和状态，重置为初始状态。

#### cudaDeviceSynchronize / 设备同步
`cudaDeviceSynchronize()` — 阻塞 host 直到当前设备上所有流中的所有任务完成。

#### cudaDeviceSetCacheConfig / 设置缓存配置
设置当前设备的全局 L1/shared memory 偏好（可被 per-function 设置覆盖）。

#### cudaDeviceSetMemPool / 设置设备内存池
`cudaDeviceSetMemPool(int device, cudaMemPool_t memPool)` — 为设备设置当前 memory pool（用于 cudaMallocAsync）。

#### cudaDeviceGetDefaultMemPool / 获取默认内存池
返回设备的默认 memory pool。

### 6.3 Error Handling / 错误处理

#### cudaGetLastError / 获取最后错误
`cudaGetLastError()` — 返回此线程最后产生的错误码并将其重置为 cudaSuccess。

#### cudaPeekAtLastError / 窥视最后错误
`cudaPeekAtLastError()` — 返回最后错误码但**不重置**。

#### cudaGetErrorName / 获取错误名
`cudaGetErrorName(cudaError_t error)` — 返回错误码的名称字符串（如 `"cudaErrorMemoryAllocation"`）。

#### cudaGetErrorString / 获取错误描述
`cudaGetErrorString(cudaError_t error)` — 返回错误码的人类可读描述字符串。

### 6.4 Stream Management / 流管理

#### cudaStreamCreate / 创建流
`cudaStreamCreate(cudaStream_t* pStream)` — 创建异步流（默认非阻塞）。

#### cudaStreamCreateWithFlags / 带标志创建流
flags: `cudaStreamDefault`（隐式同步 default stream）或 `cudaStreamNonBlocking`（与 default stream 无隐式同步）。

#### cudaStreamCreateWithPriority / 带优先级创建流
`cudaStreamCreateWithPriority(cudaStream_t* pStream, unsigned int flags, int priority)` — 数值越小优先级越高。范围通过 `cudaDeviceGetStreamPriorityRange` 查询。

#### cudaStreamDestroy / 销毁流
`cudaStreamDestroy(cudaStream_t stream)` — 等待流中排队任务完成后销毁流对象。

#### cudaStreamSynchronize / 流同步
`cudaStreamSynchronize(cudaStream_t stream)` — 阻塞 host 直到该流中所有命令完成。

#### cudaStreamWaitEvent / 流等待事件
`cudaStreamWaitEvent(cudaStream_t stream, cudaEvent_t event, unsigned int flags)` — 让流等待某事件完成后才继续执行后续命令。实现流间依赖。

#### cudaStreamQuery / 流状态查询
`cudaStreamQuery(cudaStream_t stream)` — 非阻塞查询流是否完成。返回 cudaSuccess（完成）或 cudaErrorNotReady。

#### cudaStreamBeginCapture / 开始图捕获
`cudaStreamBeginCapture(cudaStream_t stream, cudaStreamCaptureMode mode)` — 将后续流操作录制为 Graph 节点（而非执行）。

#### cudaStreamEndCapture / 结束图捕获
`cudaStreamEndCapture(cudaStream_t stream, cudaGraph_t* pGraph)` — 结束捕获并返回 cudaGraph_t 对象。

#### cudaStreamBeginCaptureToGraph / 捕获到已有图
将捕获操作附加到已有 graph 中指定的依赖节点上。

### 6.5 Event Management / 事件管理

#### cudaEventCreate / 创建事件
`cudaEventCreate(cudaEvent_t* event)` — 创建事件对象。

#### cudaEventCreateWithFlags / 带标志创建事件
flags: `cudaEventDefault` | `cudaEventBlockingSync`（host 等待时让出 CPU）| `cudaEventDisableTiming`（不记录时间，减少开销）| `cudaEventInterprocess`（跨进程 IPC）。

#### cudaEventRecord / 记录事件
`cudaEventRecord(cudaEvent_t event, cudaStream_t stream=0)` — 在流中标记一个时间点。

#### cudaEventSynchronize / 事件同步
`cudaEventSynchronize(cudaEvent_t event)` — 阻塞 host 直到事件完成。

#### cudaEventElapsedTime / 事件计时
`cudaEventElapsedTime(float* ms, cudaEvent_t start, cudaEvent_t stop)` — 返回两个事件间的毫秒数（分辨率约 0.5 μs）。

#### cudaEventQuery / 事件状态查询
`cudaEventQuery(cudaEvent_t event)` — 非阻塞查询事件是否完成。

#### cudaEventDestroy / 销毁事件
`cudaEventDestroy(cudaEvent_t event)` — 销毁事件对象。

### 6.7 Execution Control / 执行控制

#### cudaLaunchKernel / 启动内核
`cudaLaunchKernel(const void* func, dim3 gridDim, dim3 blockDim, void** args, size_t sharedMem, cudaStream_t stream)` — 标准 kernel 启动（等价于 `<<<grid, block, smem, stream>>>` 语法）。

#### cudaLaunchKernelExC / 扩展属性启动
`cudaLaunchKernelExC(const cudaLaunchConfig_t* config, const void* func, void** args)` — 带 launch-time 属性的 kernel 启动。`cudaLaunchConfig_t` 中可设置 cluster 尺寸、动态 shared memory 等 attrs 数组。

#### cudaLaunchCooperativeKernel / 协作内核启动
所有 blocks 可调用 `grid.sync()` 互相同步。总 blocks 数不能超过 maxActiveBlocks × SM 数。需 `cudaDevAttrCooperativeLaunch` 支持。

#### cudaLaunchHostFunc / 流中排队 Host 函数
`cudaLaunchHostFunc(cudaStream_t stream, cudaHostFn_t fn, void* userData)` — 在流中插入一个 host 端回调。可在回调中调用 CUDA API（替代已弃用的 cudaStreamAddCallback）。

#### cudaFuncSetAttribute / 设置函数属性
`cudaFuncSetAttribute(const void* func, cudaFuncAttribute attr, int value)` — 设置 kernel 属性：
- `cudaFuncAttributeMaxDynamicSharedMemorySize` — 动态 shared memory 上限
- `cudaFuncAttributePreferredSharedMemoryCarveout` — L1/shared 分割比例
- `cudaFuncAttributeRequiredClusterWidth/Height/Depth` — Cluster 尺寸

#### cudaFuncGetAttributes / 获取函数属性
返回 `cudaFuncAttributes` 结构体：maxThreadsPerBlock、sharedSizeBytes、numRegs、ptxVersion 等。

#### cudaFuncSetCacheConfig / 设置函数缓存偏好
`cudaFuncCachePreferNone` / `PreferShared` / `PreferL1` / `PreferEqual`。

### 6.9 Occupancy / 占用率

#### cudaOccupancyMaxActiveBlocksPerMultiprocessor / 最大活跃块数
`cudaOccupancyMaxActiveBlocksPerMultiprocessor(int* numBlocks, const void* func, int blockSize, size_t dynamicSMemSize)` — 给定 blockSize 和动态 shared memory，返回每 SM 最大活跃 blocks。

#### cudaOccupancyMaxPotentialBlockSize / 最优 Block 大小（C++）
自动搜索使占用率最大化的 blockSize（同时返回最小 grid 尺寸保证充分占用）。

#### cudaOccupancyAvailableDynamicSMemPerBlock / 可用动态共享内存
给定 numBlocks 和 blockSize，返回每 block 可使用的最大动态 shared memory。

#### cudaOccupancyMaxActiveClusters / 最大活跃 Cluster 数
返回可同时共存的最大 cluster 数（需 cudaLaunchConfig_t 中指定 cluster 尺寸）。

### 6.10 Memory Management / 内存管理

#### cudaMalloc / 设备内存分配
`cudaMalloc(void** devPtr, size_t size)` — 分配线性设备内存（未初始化）。

#### cudaMallocPitch / 2D 对齐分配
`cudaMallocPitch(void** devPtr, size_t* pitch, size_t width, size_t height)` — pitch 保证每行对齐，配合 cudaMemcpy2D 使用。

#### cudaMalloc3D / 3D 分配
返回 `cudaPitchedPtr`，用于 3D 数组的带 pitch 对齐分配。

#### cudaMallocManaged / 统一内存分配
`cudaMallocManaged(void** devPtr, size_t size, unsigned int flags=cudaMemAttachGlobal)` — Unified Memory，CPU 和 GPU 共享同一指针。flags: `cudaMemAttachGlobal`（所有设备可访问）/ `cudaMemAttachHost`（仅 CPU 及显式关联设备）。

#### cudaMallocHost / 页锁定内存分配
`cudaMallocHost(void** ptr, size_t size)` — 分配 page-locked (pinned) host 内存。DMA 传输可直接使用，避免额外拷贝。

#### cudaHostAlloc / 带标志页锁定分配
flags:
- `cudaHostAllocDefault` — 默认页锁定
- `cudaHostAllocPortable` — 所有 CUDA context 可见
- `cudaHostAllocMapped` — 映射到设备地址空间（zero-copy）
- `cudaHostAllocWriteCombined` — 写合并（CPU 写快/读慢，PCIe 传输高效）

#### cudaFree / 释放设备内存
`cudaFree(void* devPtr)` — 释放 cudaMalloc 分配的设备内存。

#### cudaFreeHost / 释放页锁定内存
`cudaFreeHost(void* ptr)` — 释放 cudaMallocHost/cudaHostAlloc 分配的内存。

#### cudaHostRegister / 注册页锁定
`cudaHostRegister(void* ptr, size_t size, unsigned int flags)` — 将已有 pageable 内存注册为 page-locked。flags: `Portable`/`Mapped`/`ReadOnly`。

#### cudaMemcpy / 内存拷贝（同步）
`cudaMemcpy(void* dst, const void* src, size_t count, cudaMemcpyKind kind)` — kind: `H2H`/`H2D`/`D2H`/`D2D`/`Default`。阻塞直到拷贝完成。

#### cudaMemcpyAsync / 内存拷贝（异步）
`cudaMemcpyAsync(..., cudaStream_t stream)` — 非阻塞。H↔D 需页锁定内存才真正异步。

#### cudaMemcpy2D / 2D 内存拷贝
用于 pitched 内存间的拷贝，指定 dpitch/spitch/width/height。

#### cudaMemset / 内存初始化
`cudaMemset(void* devPtr, int value, size_t count)` — 按字节填充设备内存。

#### cudaMemGetInfo / 查询内存信息
`cudaMemGetInfo(size_t* free, size_t* total)` — 返回当前设备的空闲/总显存字节数。

#### cudaMemAdvise / 统一内存迁移建议
对 Unified Memory 范围设置迁移提示：
- `cudaMemAdviseSetReadMostly` — 允许 read-duplication
- `cudaMemAdviseSetPreferredLocation` — 设置偏好位置减少迁移
- `cudaMemAdviseSetAccessedBy` — 建立映射避免 page fault

#### cudaMemPrefetchAsync / 异步预取
`cudaMemPrefetchAsync(const void* devPtr, size_t count, int dstDevice, cudaStream_t stream)` — 主动将 Unified Memory 迁移到目标设备（或 CPU: `cudaCpuDeviceId`）。

### 6.12 Stream Ordered Memory Allocator / 流顺序内存分配器

#### cudaMallocAsync / 流顺序分配
`cudaMallocAsync(void** devPtr, size_t size, cudaStream_t hStream)` — 在流中插入分配操作。指针立即返回但需等操作完成后才可使用。从流关联设备的 memory pool 分配。

#### cudaFreeAsync / 流顺序释放
`cudaFreeAsync(void* devPtr, cudaStream_t hStream)` — 在流中插入释放操作。释放后不可再访问。

#### cudaMallocFromPoolAsync / 从指定池分配
`cudaMallocFromPoolAsync(void** ptr, size_t size, cudaMemPool_t memPool, cudaStream_t stream)` — 从指定 memory pool 分配（pool 可在不同设备上）。

#### cudaMemPoolCreate / 创建内存池
`cudaMemPoolCreate(cudaMemPool_t* memPool, const cudaMemPoolProps* poolProps)` — 创建自定义 memory pool。poolProps 指定 backing 设备、IPC 句柄类型、maxSize 等。

#### cudaMemPoolDestroy / 销毁内存池
待所有 outstanding 分配释放后自动回收资源。销毁当前 mempool 会回退到默认 pool。

#### cudaMemPoolTrimTo / 修剪内存池
`cudaMemPoolTrimTo(cudaMemPool_t memPool, size_t minBytesToKeep)` — 将池保留内存缩减到 minBytesToKeep，释放多余 OS 内存。

#### cudaMemPoolSetAttribute / 设置池属性
可设置：`cudaMemPoolAttrReleaseThreshold`（释放阈值）、`ReuseFollowEventDependencies`/`ReuseAllowOpportunistic`/`ReuseAllowInternalDependencies`（重用策略）。

#### cudaMemPoolGetAttribute / 获取池属性
查询：`ReservedMemCurrent/High`、`UsedMemCurrent/High`、`ReleaseThreshold`、`MaxPoolSize` 等。

#### cudaMemPoolSetAccess / 设置池访问权限
`cudaMemPoolSetAccess(cudaMemPool_t memPool, const cudaMemAccessDesc* descList, size_t count)` — 控制哪些设备可访问池中内存。

#### cudaMemPoolExportToShareableHandle / 导出池句柄（IPC）
将 IPC capable mempool 导出为 OS 句柄，用于跨进程共享。

#### cudaMemPoolImportFromShareableHandle / 导入池句柄（IPC）
从另一进程的句柄导入 mempool。导入的 pool 不可创建新分配。

#### cudaMemPoolExportPointer / 导出指针（IPC）
导出特定分配的 export data，通过任意 IPC 机制传递给对端进程。

#### cudaMemPoolImportPointer / 导入指针（IPC）
对端进程用 export data 导入指针。

### 6.13 Unified Addressing / 统一寻址

#### Unified Virtual Address Space / 统一虚拟地址空间（UVA）
64位进程自动启用。CPU 和 GPU 共享统一地址空间：同一指针值可在 host 和 device 使用。
- `cudaMemcpyDefault` 可自动推断拷贝方向
- cudaMallocHost 分配的内存无需 cudaHostGetDevicePointer 即可在 kernel 中使用

#### cudaPointerGetAttributes / 查询指针属性
`cudaPointerGetAttributes(cudaPointerAttributes* attributes, const void* ptr)` — 返回指针的 type（`Unregistered`/`Host`/`Device`/`Managed`）、device ID、devicePointer、hostPointer。

### 6.14 Peer Device Memory Access / P2P 内存访问

#### cudaDeviceCanAccessPeer / 查询 P2P 可达性
`cudaDeviceCanAccessPeer(int* canAccessPeer, int device, int peerDevice)` — 返回 1 表示 device 可直接访问 peerDevice 的内存。

#### cudaDeviceEnablePeerAccess / 启用 P2P 访问
`cudaDeviceEnablePeerAccess(int peerDevice, unsigned int flags)` — 启用后，peerDevice 的 cudaMalloc 内存可被当前设备直接访问。单向的（需对称调用）。

#### cudaDeviceDisablePeerAccess / 禁用 P2P 访问
`cudaDeviceDisablePeerAccess(int peerDevice)` — 关闭直接访问。


---

## Compute Capabilities Appendix 词汇表

> 来源：CUDA Programming Guide - 5.1 Compute Capabilities（2026-05 更新）

### CC-1 计算能力与架构代号

| 英文 | 中文 | 说明 |
|------|------|------|
| Compute Capability (CC) | 计算能力 | 定义 GPU 硬件特性与指令集版本，格式 major.minor |
| Streaming Multiprocessor (SM) | 流多处理器 | GPU 的基本并行计算单元 |
| Architecture | 架构 | GPU 微架构代号（Turing/Ampere/Hopper/Blackwell 等） |
| Turing | 图灵 | CC 7.5，引入 RT Core + Tensor Core（GeForce RTX 20 系列） |
| Ampere | 安培 | CC 8.0/8.6/8.7/8.9，引入 A100/RTX 30 系列 |
| Ada Lovelace | 阿达·洛芙莱斯 | CC 8.9，RTX 40 系列/L4/L40S |
| Hopper | 霍珀 | CC 9.0，H100/H200/GH200，引入 Transformer Engine |
| Blackwell | 布莱克威尔 | CC 10.0/10.3/12.0/12.1，B200/GB200/RTX 50 系列 |
| Baseline Feature Set | 基线特性集 | 每代引入且后续架构保留的通用特性 |
| Architecture-Specific Feature Set | 架构专属特性集 | 仅特定 CC 可用的加速特性，需 `a` 后缀编译 |
| Family-Specific Feature Set | 家族特性集 | 同家族多 CC 共享的特性子集，需 `f` 后缀编译 |

### CC-2 硬件规格参数

| 英文 | 中文 | 说明 |
|------|------|------|
| Threads per Warp | 每 Warp 线程数 | 固定 32，所有 CC 一致 |
| Maximum Threads per Block | 每 Block 最大线程数 | 固定 1024 |
| Maximum Resident Blocks per SM | 每 SM 最大驻留 Block | CC 依赖：16/32/24 |
| Maximum Resident Warps per SM | 每 SM 最大驻留 Warp | CC 依赖：32/64/48 |
| Maximum Resident Threads per SM | 每 SM 最大驻留线程 | CC 依赖：1024/2048/1536 |
| 32-bit Registers per SM | 每 SM 32位寄存器数 | 所有 CC 均为 64K |
| Maximum Registers per Thread | 每线程最大寄存器 | 固定 255 |
| Shared Memory per SM | 每 SM 共享内存 | 64KB~228KB，CC 依赖 |
| Shared Memory per Block | 每 Block 共享内存 | 最大可达 227KB（CC 9.0+），需 opt-in |
| Shared Memory Banks | 共享内存 Bank 数 | 固定 32 |
| Local Memory per Thread | 每线程本地内存 | 512KB |
| Constant Memory | 常量内存 | 64KB，每 SM 8KB 缓存 |
| Maximum Grid Dimensions | 最大网格维度 | x=2³¹-1, y/z=65535 |
| Maximum Block Dimensions | 最大 Block 维度 | x/y=1024, z=64 |
| Resident Grids per Device | 设备最大驻留网格数 | 128 |

### CC-3 Tensor Core 支持矩阵

| 英文 | 中文 | 说明 |
|------|------|------|
| Tensor Core | 张量核心 | 混合精度矩阵乘加硬件单元 |
| TF32 (TensorFloat-32) | TF32 格式 | 19位浮点（8E+10M），Ampere+ Tensor Core 默认精度 |
| BF16 (BFloat16) | BF16 格式 | 16位（8E+7M），训练友好精度 |
| FP16 (Half Precision) | 半精度浮点 | 16位（5E+10M），推理常用 |
| FP8 (E4M3/E5M2) | FP8 格式 | 8位浮点，Hopper+ 支持，推理加速 |
| FP6 | FP6 格式 | 6位浮点，Blackwell 引入 |
| FP4 | FP4 格式 | 4位浮点，Blackwell 引入，极致量化推理 |
| INT8 | 8位整数 | 整数量化推理 |
| INT4 | 4位整数 | 超低精度量化（Turing/Ampere 支持） |
| FP64 Tensor Core | 双精度张量核心 | CC 8.0/9.0/10.0 支持，科学计算 |

### CC-4 架构专属特性

| 英文 | 中文 | 说明 |
|------|------|------|
| DPX Instructions | DPX 指令 | 动态规划加速指令（Hopper+ 原生支持） |
| Distributed Shared Memory (DSMEM) | 分布式共享内存 | Cluster 内跨 Block 访问其他 Block 的 SMEM |
| Thread Block Cluster | 线程块集群 | 多 Block 组成的协作组，共享 DSMEM |
| Tensor Memory Accelerator (TMA) | 张量内存加速器 | 异步批量数据搬运单元（Hopper+） |
| Transformer Engine | 变换器引擎 | FP8 自动混合精度训练引擎（Hopper+） |
| Green Contexts | 绿色上下文 | SM 资源分区隔离执行（CC 9.0+ 引入） |
| L2 Cache Residency Management | L2 缓存驻留管理 | 控制数据在 L2 的持久驻留窗口（CC 8.0+） |
| Hardware memcpy_async | 硬件异步拷贝 | 绕过 RF 直接 Global→Shared 搬运（CC 8.0+） |
| Asynchronous Barrier | 异步屏障 | 硬件加速 split arrive/wait 屏障（CC 8.0+） |
| Warp Reduce Functions | Warp 规约函数 | 硬件 warp 级规约操作（CC 8.0+） |

### CC-5 编译目标与兼容性

| 英文 | 中文 | 说明 |
|------|------|------|
| compute_XX | 虚拟架构目标 | PTX 中间码目标，向前兼容 |
| sm_XX | 真实架构目标 | SASS 机器码目标，仅对应 CC 可执行 |
| compute_XXa | 架构专属目标 | 启用全部架构专属特性，仅该 CC 可运行 |
| compute_XXf | 家族目标 | 启用家族共享特性，同家族 CC 可运行 |
| PTX (Parallel Thread Execution) | 并行线程执行 | NVIDIA 虚拟 ISA 中间表示 |
| SASS | SASS 汇编 | GPU 真实机器指令集 |
| JIT Compilation | 即时编译 | 驱动将 PTX 编译为当前 GPU 的 SASS |
| Fat Binary | 胖二进制 | 包含多 CC 版本 SASS + PTX 的可执行文件 |
| nvcc | NVIDIA CUDA 编译器 | `-arch`/`-code` 指定目标 CC |
