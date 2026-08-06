---
tags: [nvidia, glossary, vocabulary]
domain: all
last_updated: 2026-07-01
source: https://docs.nvidia.com/
---

# 主词汇表（Master Glossary）

跨领域汇总，按字母序排列。各领域详细词汇见子目录。

> 格式：`英文术语 / 中文译名`
> 标注：**领域标签** | 来源文档

---

## A

| 英文 | 中文 | 领域 | 备注 |
|------|------|------|------|
| API (Application Programming Interface) | 应用程序编程接口 | 通用 | |
| Accelerated Computing | 加速计算 | CUDA / Hardware | NVIDIA 核心产品方向 |
| Agentic AI | 智能体 AI | AI Software | 自主执行任务的 AI 系统 |
| Async Barrier（cuda::barrier） | 异步屏障 | CUDA | arrive-wait 两阶段屏障，允许线程在两阶段间执行独立工作；CC 8.0+ 硬件加速 |
| ATS (Address Translation Service) | 地址转换服务 | CUDA/Hardware | 允许 GPU 通过 PCIe ATS 直接访问 CPU 虚拟地址空间，需 CPU 侧 IOMMU 支持 |

## B

| 英文 | 中文 | 领域 | 备注 |
|------|------|------|------|
| Batch Inference | 批量推理 | AI/TensorRT | 同时处理多个输入以提高吞吐量 |
| Bandwidth | 带宽 | Hardware | 数据传输速率，单位 GB/s 或 TB/s |

## C

| 英文 | 中文 | 领域 | 备注 |
|------|------|------|------|
| CUDA Core | CUDA 核心 | CUDA/Hardware | GPU 中执行浮点和整数运算的基本处理单元 |
| Compute Capability | 计算能力 | CUDA | 标识 GPU 架构特性的版本号，如 8.0（Ampere） |

## D

| 英文 | 中文 | 领域 | 备注 |
|------|------|------|------|
| Data Parallelism | 数据并行 | CUDA/AI | 将数据切分到多个处理单元同时计算 |
| Deep Learning | 深度学习 | AI | 基于多层神经网络的机器学习方法 |

## F

| 英文 | 中文 | 领域 | 备注 |
|------|------|------|------|
| Floating Point (FP16/FP32/FP64) | 浮点数精度 | CUDA/Hardware | 半精度/单精度/双精度浮点格式 |
| Foundation Model | 基础模型 | AI | 大规模预训练，可微调用于多种任务的模型 |

## G

| 英文 | 中文 | 领域 | 备注 |
|------|------|------|------|
| GEMM (General Matrix Multiply) | 通用矩阵乘法 | CUDA/CUTLASS | 深度学习计算的核心操作 |
| GPU (Graphics Processing Unit) | 图形处理器 | Hardware | |

## I

| 英文 | 中文 | 领域 | 备注 |
|------|------|------|------|
| Independent Thread Scheduling | 独立线程调度 | CUDA | CC 7.0+ 每线程独立 PC 和调用栈，warp 内线程可分叉重聚合；旧 lockstep 代码需改用 `__syncwarp()` |
| Inference | 推理 | AI/TensorRT | 使用训练好的模型对新数据进行预测 |

## K

| 英文 | 中文 | 领域 | 备注 |
|------|------|------|------|
| Kernel | 内核（核函数） | CUDA | 在 GPU 上并行执行的函数，由大量线程同时运行 |

## L

| 英文 | 中文 | 领域 | 备注 |
|------|------|------|------|
| Latency | 延迟 | Hardware/AI | 从请求发出到收到响应的时间 |

## M

| 英文 | 中文 | 领域 | 备注 |
|------|------|------|------|
| Microservice | 微服务 | AI/NIM | 独立部署、松耦合的服务单元 |
| Model Parallelism | 模型并行 | AI | 将大模型切分到多 GPU 上运行 |

## P

| 英文 | 中文 | 领域 | 备注 |
|------|------|------|------|
| PDL (Programmatic Dependent Launch) | 程序化依赖启动 | CUDA | CC 9.0+，允许有依赖的两个 kernel 在同一 stream 中部分重叠执行，减少 launch overhead |
| Peer-to-Peer (P2P) Memory Access | 点对点内存访问 | CUDA/Multi-GPU | GPU 直接读写另一 GPU 显存，NVLink 路径下无需经过 host |
| Pipeline（cuda::pipeline） | 异步流水线同步原语 | CUDA | 多阶段 producer-consumer 同步，用于实现 double/multi-buffer 异步加载 |
| Precision (INT8/FP16/FP32) | 计算精度 | TensorRT/CUDA | 影响推理速度与精度的权衡 |
| PTX (Parallel Thread Execution) | 并行线程执行 ISA | CUDA | CUDA 编译中间表示，驱动 JIT 编译为目标架构二进制，保证前向兼容性 |

## S

| 英文 | 中文 | 领域 | 备注 |
|------|------|------|------|
| Scoped Atomic（cuda::atomic） | 作用域原子操作 | CUDA | 将 C++ 原子语义与线程作用域结合；使用最窄 scope + 最弱 memory_order 以获得最佳性能 |
| SDK (Software Development Kit) | 软件开发工具包 | 通用 | |
| SIMT (Single Instruction, Multiple Thread) | 单指令多线程 | CUDA/Hardware | GPU 执行模型，warp（32 线程）同时执行一条指令；SIMD 暴露宽度，SIMT 对程序员透明 |
| Streaming Multiprocessor (SM) | 流式多处理器 | CUDA/Hardware | GPU 的基本计算单元，包含多个 CUDA Core 和 Tensor Core |

## T

| 英文 | 中文 | 领域 | 备注 |
|------|------|------|------|
| Tensor Core | 张量核心 | Hardware/CUDA | 专为矩阵乘法加速设计的硬件单元，支持 FP16/BF16/INT8 |
| Thread Block Cluster | 线程块簇 | CUDA | CC 9.0+ 层级，保证一组 blocks 同时调度在同一 GPC，支持 distributed shared memory 访问 |
| Thread Scope | 线程作用域 | CUDA | 定义内存操作可见性范围：thread / block / device / system（cluster），影响原子和屏障的有效范围 |
| Throughput | 吞吐量 | Hardware/AI | 单位时间内处理的数据量或请求数 |
| TMA (Tensor Memory Accelerator) | 张量内存加速器 | CUDA/Hardware | CC 9.0（Hopper）专用 DMA 硬件，支持大块多维张量异步拷贝（global ↔ shared，含 cluster）|
| Training | 训练 | AI | 通过数据调整模型参数的过程 |

## U

| 英文 | 中文 | 领域 | 备注 |
|------|------|------|------|
| Unified Memory | 统一内存 | CUDA | CPU 和 GPU 共享的虚拟内存空间，简化编程模型 |

## V

| 英文 | 中文 | 领域 | 备注 |
|------|------|------|------|
| VRAM (Video RAM) | 显存 | Hardware | GPU 专用内存，如 H100 有 80GB HBM3 |

## CUDA Part 4

| 术语 | 中文 | 简述 |
|------|------|------|
| Unified Memory | 统一内存 | CPU/GPU 共享地址空间，按需迁移页面 |
| CUDA Graph | CUDA 图 | 将 kernel/memcpy 序列捕获为可重放图对象 |
| Stream Capture | 流捕获 | 录制流操作以构建 CUDA Graph |
| Stream-Ordered Allocator | 流有序分配器 | cudaMallocAsync/cudaFreeAsync，按流序分配显存 |
| Cooperative Groups | 协作组 | 灵活线程分组同步原语，支持 warp/block/grid 级同步 |
| Programmatic Dependent Launch | 程序化依赖启动 | Primary kernel 主动触发 Secondary kernel 提前启动 |
| Green Contexts | 绿色上下文 | 将 SM 资源子集分配给独立执行上下文 |
| Lazy Loading | 懒加载 | 模块中函数按需加载，减少启动内存占用 |
| Error Log Management | 错误日志管理 | CUDA driver 运行时日志收集与导出 API |
| Async Barrier | 异步屏障 | cuda::barrier 支持异步拷贝完成通知 |
| Pipeline | 管线 | cuda::pipeline 实现生产者/消费者异步流水线 |
| LDGSTS | 全局→共享内存异步拷贝 | cp.async 指令，CC 8.0+，绕过 L1 直通共享内存 |
| TMA | 张量内存加速器 | CC 9.0+ 硬件单元，支持多维块拷贝，含 multicast |
| Cluster Launch Control | 集群启动控制 | CC 10.0+ Blackwell，设备端取消集群启动 |
| L2 Cache Persistence | L2 持久化缓存 | 预留 L2 区域供高频数据驻留，减少 DRAM 访问 |
| Memory Synchronization Domains | 内存同步域 | CC 9.0+ Hopper，隔离 NCCL 通信与计算的 fence 域 |
| CUDA IPC | 进程间通信 | 跨进程共享显存句柄和 CUDA event |
| Virtual Memory Management | 虚拟内存管理 | Driver API 细粒度物理/虚拟内存分离控制 |
| Extended GPU Memory | 扩展 GPU 内存 | Grace Hopper NVLink-C2C，GPU 高带宽访问 CPU NUMA 内存 |
| Dynamic Parallelism CDP2 | 动态并行第2版 | 设备端直接启动子 kernel，CUDA 12.0+ CC 9.0+ |
| Graphics Interoperability | 图形互操作 | CUDA 与 OpenGL/Vulkan/D3D 共享缓冲区和同步对象 |
| Driver Entry Point Access | 驱动入口访问 | 运行时获取指定版本 Driver API 函数指针 |

## CUDA C++ Best Practices Guide

| Term | 中文 | 定义 |
|------|------|------|
| APOD Cycle | 评估-并行化-优化-部署循环 | Assess→Parallelize→Optimize→Deploy 迭代开发方法论 |
| Amdahl's Law | 阿姆达尔定律 | 加速比上限 S=1/(1-P+P/N)，P 为可并行部分比例 |
| Strong Scaling | 强扩展 | 固定问题规模增加处理器数，受 Amdahl 定律约束 |
| Weak Scaling | 弱扩展 | 处理器数增加时同比扩大问题规模，遵循 Gustafson 定律 |
| Theoretical Bandwidth | 理论带宽 | 由硬件规格推算的内存带宽峰值 |
| Effective Bandwidth | 有效带宽 | 实测读写字节数/时间，与理论带宽对比量化优化空间 |
| Pinned Memory | 页锁定内存 | cudaHostAlloc 分配，不可换出，PCIe 传输峰值 ~12 GB/s |
| Zero-Copy Memory | 零拷贝内存 | 主机内存映射到 GPU 地址空间，适合集成 GPU |
| Async Data Transfer | 异步数据传输 | cudaMemcpyAsync + stream 与 kernel 计算重叠 |
| Coalesced Access | 合并访问 | warp 32线程访问连续对齐内存，合并为最少事务数 |
| Bank Conflict | Bank 冲突 | shared memory 同 bank 多地址访问串行化，padding 消除 |
| Occupancy | 占用率 | 活跃 warp / SM 最大支持 warp，影响延迟隐藏能力 |
| Register Pressure | 寄存器压力 | 每线程寄存器过多导致 occupancy 下降 |
| Warp Divergence | Warp 分歧 | warp 内线程走不同分支路径，各路径串行执行 |
| FMA | 融合乘加 | 单条指令完成 a×b+c，精度和性能均优于分开执行 |
| Fast Math Intrinsics | 快速数学内置函数 | __sinf/__expf 等映射硬件指令，精度约 2 ULP |
| JIT Compilation | 即时编译 | 驱动运行时将 PTX 编译为目标设备 cubin |
| CUDA Compatibility | CUDA 兼容性 | Forward/Enhanced/Binary 三层兼容性机制 |
| NVML | NVIDIA 管理库 | C API 提供与 nvidia-smi 等同的查询控制接口 |
| CUDA_VISIBLE_DEVICES | 可见设备变量 | 环境变量限制应用可见 GPU 子集 |


## CUDA Runtime API

| 术语 | 中文 | 简述 |
|------|------|------|
| cudaMalloc / cudaFree | 显存分配/释放 | 基础设备内存管理 API |
| cudaMallocAsync / cudaFreeAsync | 异步显存分配/释放 | 流有序内存池分配，减少同步开销 |
| cudaMemPool | 内存池 | 管理流有序分配的底层池对象 |
| cudaMemcpy | 内存拷贝 | Host↔Device / Device↔Device 数据传输 |
| cudaMemcpyAsync | 异步内存拷贝 | 非阻塞数据传输，与计算重叠 |
| Unified Virtual Addressing (UVA) | 统一虚拟寻址 | 单一地址空间覆盖 Host+所有 Device |
| Peer-to-Peer (P2P) | 对等访问 | GPU 间直接显存访问/拷贝，绕过 Host |
| cudaLaunchKernel | 核函数启动 | Runtime API 级 kernel 启动入口 |
| cudaStream_t | CUDA 流 | 异步操作队列，支持并发执行 |
| cudaEvent_t | CUDA 事件 | 流中插入标记用于计时/同步 |
| cudaGraph_t | CUDA 图 | 捕获操作序列为可重放 DAG |
| cudaGraphExec_t | 图执行实例 | 图实例化后的可启动对象 |
| Occupancy API | 占用率 API | 计算最优 block 大小以最大化 SM 利用率 |
| cudaDeviceProp | 设备属性结构体 | 查询 GPU 计算能力、内存大小等硬件信息 |
| cudaSetDevice | 设置当前设备 | 选择后续操作所针对的 GPU |
| cudaDeviceSynchronize | 设备同步 | 阻塞 Host 等待 Device 所有操作完成 |
| cudaStreamSynchronize | 流同步 | 阻塞 Host 等待指定流操作完成 |
| cudaMallocManaged | 托管内存分配 | 分配 Unified Memory，系统自动迁移 |
| cudaMemPrefetchAsync | 异步预取 | 将托管内存页预迁移到目标设备 |
| cudaMemAdvise | 内存建议 | 向驱动提示数据访问模式(ReadMostly/PreferredLocation等) |
| Surface Object | 表面对象 | 支持读写的纹理内存绑定 |
| Texture Object | 纹理对象 | 只读缓存优化的数据访问对象 |
| cudaExternalMemory | 外部内存 | 导入 Vulkan/D3D 分配的内存到 CUDA |
| cudaExternalSemaphore | 外部信号量 | 跨 API (Vulkan/D3D↔CUDA) 同步原语 |
| cudaLaunchCooperativeKernel | 协作核函数启动 | 支持 grid 级同步的 kernel 启动方式 |

## Compute Capabilities

| 术语 | 中文 | 简述 |
|------|------|------|
| Compute Capability (CC) | 计算能力 | GPU 硬件特性版本号，格式 major.minor |
| Turing (CC 7.5) | 图灵架构 | T4/RTX 2080，引入 RT Core + Tensor Core |
| Ampere (CC 8.0/8.6) | 安培架构 | A100/RTX 3090，TF32 + hardware memcpy_async |
| Ada Lovelace (CC 8.9) | 阿达架构 | L4/RTX 4090，FP8 Tensor Core |
| Hopper (CC 9.0) | 霍珀架构 | H100，TMA + DSMEM + Transformer Engine |
| Blackwell (CC 10.0/12.0) | 布莱克威尔架构 | B200/RTX 5090，FP4/FP6 Tensor Core |
| Tensor Core | 张量核心 | 混合精度矩阵乘加硬件单元 |
| TF32 | TensorFloat-32 | 19位浮点格式，Ampere+ 默认 matmul 精度 |
| FP8 (E4M3/E5M2) | 8位浮点 | Hopper+ 推理/训练低精度格式 |
| FP4 | 4位浮点 | Blackwell 引入的极致量化格式 |
| Architecture-Specific Feature | 架构专属特性 | 仅特定 CC 可用，需 compute_XXa 编译 |
| Family-Specific Feature | 家族特性 | 同家族 CC 共享，需 compute_XXf 编译 |
| PTX | 并行线程执行 | NVIDIA 虚拟指令集中间表示 |
| SASS | GPU 机器码 | 真实硬件指令集 |
| Fat Binary | 胖二进制 | 包含多 CC SASS + PTX 的可执行文件 |
| JIT Compilation | 即时编译 | 驱动将 PTX 编译为当前 GPU 的 SASS |
| DPX Instructions | 动态规划加速指令 | Hopper+ 原生支持 |
| Distributed Shared Memory | 分布式共享内存 | Cluster 内跨 Block 访问其他 Block 的 SMEM |
| Thread Block Cluster | 线程块集群 | Hopper+ 多 Block 协作组 |
| Transformer Engine | 变换器引擎 | FP8 自动混合精度训练引擎 |


### TensorRT Developer Guide 核心术语

| 术语 | 中文 | 简述 |
|------|------|------|
| Q/DQ Layer | 量化/反量化层 | 显式标注量化边界的网络节点 |
| PTQ (Post-Training Quantization) | 训练后量化 | 用校准数据集确定 scale |
| QAT (Quantization-Aware Training) | 量化感知训练 | 训练中模拟量化误差 |
| MXFP8 | 微缩放 FP8 | E8M0 scale + FP8 数据的分块量化 |
| Strongly Typed Network | 强类型网络 | 用户显式指定每层精度（11.0+ 唯一模式） |
| Optimization Profile | 优化配置 | 为动态维度指定 min/opt/max |
| Shape Tensor | 形状张量 | 内容为维度值的特殊张量 |
| IPluginV3 | 插件 V3 接口 | 当前推荐的 TRT 自定义层基类 |
| Fused Attention | 融合注意力 | QKV+softmax+V 合一 kernel |
| KV Cache | KV 缓存 | 存储历史 K/V 避免 LLM 重复计算 |
| RoPE | 旋转位置编码 | 相对位置编码（LLaMA 系列） |
| Mixture of Experts (MoE) | 混合专家 | 稀疏激活，每 token 路由到 k 个专家 |
| Weight Streaming | 权重流式 | Host→Device 按需传输超大模型权重 |
| Refitting | 权重刷新 | 运行时更新引擎权重无需重建 |
| Weight Stripping | 权重剥离 | Plan 不嵌权重，部署时 refit 注入 |
| Version Compatibility | 版本兼容 | 引擎跨 TRT minor 版本可加载 |
| Hardware Compatibility | 硬件兼容 | 引擎在同代不同 GPU 间可移植 |
| Layer Fusion | 层融合 | Builder 合并多层为单一 kernel |
| Timing Cache | 计时缓存 | 缓存 kernel 择优结果加速构建 |
| Builder Optimization Level | 构建优化等级 | 0-5 控制搜索深度与构建时间 |
| IDistCollectiveLayer | 分布式集合层 | AllReduce/AllGather 等多 GPU 通信原语 |
| Context Parallelism | 上下文并行 | 长序列拆分到多 GPU 并行注意力 |
| Polygraphy | 调试工具包 | 多后端对比/ONNX 编辑/精度诊断 |
| trtexec | TRT 命令行工具 | 基准测试/引擎构建/缓存生成 |
| Global Performance Tuning | 全局性能调优 | 自动化 build route 搜索 |
