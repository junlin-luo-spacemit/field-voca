---
tags: [nvidia, nccl, vocabulary]
domain: nccl
last_updated: 2026-07-21
part: Overview + Usage ✅
source: https://docs.nvidia.com/deeplearning/nccl/user-guide/docs/
---

# NCCL 词汇表

> 来源：https://docs.nvidia.com/deeplearning/nccl/user-guide/docs/
> 版本：NCCL 2.x (Jul 2026)
> 格式：英文术语 → 中文译名 + 官方定义 + 例句

---

## 核心概念（Core Concepts）

### NCCL (NVIDIA Collective Communications Library) / NCCL 集合通信库
NVIDIA 提供的多 GPU 和多节点通信库。实现了针对 NVIDIA GPU 优化的集合操作（AllReduce、Broadcast 等），在单机多卡和多机多卡环境下实现高带宽、低延迟的数据传输。
> *The NVIDIA Collective Communications Library (NCCL) implements multi-GPU and multi-node communication primitives optimized for NVIDIA GPUs and Networking.*

### collective operation / 集合操作
涉及一组 GPU 的协同通信操作。所有参与的 rank 必须使用相同的 count 和 datatype 调用操作，否则会导致未定义行为（挂起、崩溃或数据损坏）。
> *Collective operations have to be called for each rank using the same count and the same datatype, to form a complete collective operation.*

### rank / 秩（rank）
集合通信中每个 GPU 的唯一标识符，取值范围 0 到 nranks-1。每个 rank 对应一个 CUDA 设备。
> *Each rank is associated with a CUDA device, and the operations are performed on the corresponding CUDA device.*

### nranks / 参与进程总数
参与集合操作的总 rank 数量，即总 GPU 数量。
> *nranks is the total number of ranks participating in the collective operation.*

### communicator / 通信器（ncclComm_t）
NCCL 通信器对象，定义了一组参与集合操作的 rank。每个 rank 必须创建自己的通信器实例。
> *A communicator defines a set of ranks that can call a collective operation.*

### ncclUniqueId / 唯一通信器 ID
用于在多个进程间初始化同一个通信器的唯一标识符。通常由 rank 0 生成并广播给所有其他 rank。
> *A unique identifier used to initialize communicators across multiple processes.*

---

## 集合操作（Collective Operations）

### AllReduce / 全局归约
在所有 rank 上执行归约操作（sum、prod、min、max 等），并将结果分发到所有 rank。最常用的集合操作，用于神经网络训练的梯度同步。
> *AllReduce performs a reduction operation (such as sum, product, minimum, or maximum) on data across all ranks, and makes the result available on all ranks.*

### Broadcast / 广播
将 root rank 的数据复制到所有其他 rank。
> *Broadcast copies data from the root rank to all other ranks.*

### Reduce / 归约
在所有 rank 上执行归约操作，但结果仅在 root rank 上可用。
> *Reduce performs a reduction operation on data across all ranks, and makes the result available only on the root rank.*

### AllGather / 全局收集
从所有 rank 收集数据并将完整数据分发到所有 rank。输出大小是输入的 nranks 倍。
> *AllGather gathers data from all ranks and makes it available on all ranks. The output size is nranks times the input size.*

### ReduceScatter / 归约散播
执行归约操作并将结果分散到所有 rank，每个 rank 收到结果的一部分。是 AllReduce 的前半部分。
> *ReduceScatter performs a reduction operation on data, and then scatters the result to all ranks. Each rank receives a portion of the result.*

### AlltoAll / 全对全
每个 rank 向所有其他 rank 发送不同的数据，并从所有其他 rank 接收不同的数据。
> *AlltoAll operation where each rank sends different data to all other ranks and receives different data from all other ranks.*

### Gather / 收集
将所有 rank 的数据收集到 root rank。仅 root 有完整输出，其他 rank 的 recvbuff 未使用。
> *Gather collects data from all ranks to the root rank. Only the root rank has the complete output.*

### Scatter / 散播
将 root rank 的数据分散到所有 rank，每个 rank 收到一部分。仅 root 的 sendbuff 使用，其他 rank 的 sendbuff 未使用。
> *Scatter distributes data from the root rank to all ranks. Each rank receives a portion of the data.*

---

## 归约操作符（Reduction Operations）

### ncclSum / 求和归约
对所有 rank 的数据执行逐元素求和。用于梯度平均。
> *Sum reduction operation. Commonly used for gradient averaging in data parallel training.*

### ncclProd / 乘积归约
对所有 rank 的数据执行逐元素乘积。
> *Product reduction operation.*

### ncclMax / 最大值归约
对所有 rank 的数据执行逐元素取最大值。
> *Maximum reduction operation.*

### ncclMin / 最小值归约
对所有 rank 的数据执行逐元素取最小值。
> *Minimum reduction operation.*

### ncclAvg / 平均值归约
对所有 rank 的数据执行逐元素求平均（sum / nranks）。
> *Average reduction operation (sum divided by nranks).*

---

## 点对点通信（Point-to-Point Communication）

### ncclSend / 发送
从当前 rank 向指定 peer rank 发送数据。必须在 ncclGroupStart/End 内使用。
> *Send data from the current rank to a peer rank. Must be used within ncclGroupStart/End.*

### ncclRecv / 接收
从指定 peer rank 接收数据到当前 rank。必须在 ncclGroupStart/End 内使用。
> *Receive data from a peer rank to the current rank. Must be used within ncclGroupStart/End.*

### peer / 对等节点
点对点通信中的目标或源 rank。
> *The target or source rank in point-to-point communication.*

### sendrecv / 发送接收（双边通信）
两个 rank 同时互相发送和接收数据。通过将 ncclSend 和 ncclRecv 合并到一个 group 中实现。
> *In MPI terms, a sendrecv operation is when two ranks exchange data, both sending and receiving at the same time.*

### one-to-all (scatter) / 一对多（散播）
一个 root rank 向所有其他 rank 发送不同数据。通过合并多个 ncclSend 和所有 rank 的 ncclRecv 实现。
> *A one-to-all operation from a root rank can be expressed by merging all send and receive operations in a group.*

### all-to-one (gather) / 多对一（收集）
所有 rank 向一个 root rank 发送数据。
> *All ranks send data to one root rank.*

### one-sided communication / 单边通信
仅调用 ncclSend 或 ncclRecv 之一的通信模式。只有发送方或接收方知道通信发生，另一方通过 ncclRecv(NULL) 或 ncclSend(NULL) 匹配。
> *One-sided communication where only the sender or receiver knows about the communication.*

---

## 组调用（Group Calls）

### ncclGroupStart() / 组调用开始
标记一组 NCCL 调用的开始。组内的操作被合并并同时启动，提高性能并避免死锁。
> *ncclGroupStart marks the start of a group of NCCL calls. Operations within the group are aggregated and launched simultaneously.*

### ncclGroupEnd() / 组调用结束
标记一组 NCCL 调用的结束。调用此函数后，组内的所有操作被启动。
> *ncclGroupEnd marks the end of a group of NCCL calls. After this call, all operations in the group are launched.*

### group call / 组调用
在 ncclGroupStart 和 ncclGroupEnd 之间的多个 NCCL 调用。可以包含多个集合操作、点对点操作或混合。
> *Multiple NCCL calls between ncclGroupStart and ncclGroupEnd. Can include multiple collective or point-to-point operations.*

### aggregation / 聚合
NCCL 将组内的多个操作合并，以优化内核启动和通信模式。
> *NCCL aggregates multiple operations within a group to optimize kernel launches and communication patterns.*

---

## CUDA 流语义（CUDA Stream Semantics）

### stream / CUDA 流
传递给 NCCL 集合操作的 CUDA 流。NCCL 调用将操作入队到该流后立即返回，操作在设备上异步执行。
> *NCCL calls are associated to a stream which is passed as the last argument. The NCCL call returns when the operation has been enqueued to the given stream.*

### asynchronous execution / 异步执行
NCCL 操作入队后，控制权立即返回给主机代码，操作在 GPU 上异步执行。使用 cudaStreamSynchronize 或 CUDA 事件查询状态。
> *The collective operation is executed asynchronously on the CUDA device. The operation status can be queried using cudaStreamSynchronize or CUDA events.*

### stream ordering / 流序
NCCL 操作遵守 CUDA 流的顺序语义。同一流上的操作按入队顺序执行。
> *NCCL operations respect CUDA stream ordering semantics. Operations on the same stream execute in the order they were enqueued.*

### multi-stream / 多流
在同一个 ncclGroupStart/End 组内使用不同的流。允许细粒度的计算-通信重叠。
> *Using different streams within the same ncclGroupStart/End group. Allows fine-grained compute-communication overlap.*

---

## 通信器管理（Communicator Management）

### ncclCommInitRank() / 初始化通信器（指定 rank）
为当前进程初始化通信器，指定其 rank、总 nranks 和 ncclUniqueId。
> *Initialize a communicator for the current process, specifying its rank, total nranks, and ncclUniqueId.*

### ncclCommInitRankConfig() / 初始化通信器（带配置）
使用配置选项初始化通信器。可指定 blocking、minCTAs、maxCTAs、cgaClusterSize、netName 等参数。
> *Initialize a communicator with specific configuration options such as blocking, minCTAs, maxCTAs, cgaClusterSize, and netName.*

### ncclCommInitAll() / 一次初始化所有通信器
单线程情况下，一次性为所有设备初始化通信器数组。
> *Initialize an array of communicators for all devices in a single-threaded context.*

### ncclCommDestroy() / 销毁通信器
释放通信器资源。在销毁前必须确保所有操作已完成。
> *Destroy a communicator and free its resources. All operations must be complete before destroying.*

### ncclCommAbort() / 中止通信器
立即中止通信器的所有待处理操作，用于错误恢复。
> *Abort all pending operations on a communicator. Used for error recovery.*

### ncclCommSplit() / 分割通信器
将现有通信器分割成多个子通信器。用于分层通信模式。
> *Split an existing communicator into multiple sub-communicators. Used for hierarchical communication patterns.*

---

## 配置选项（Configuration Options）

### ncclConfig_t / NCCL 配置结构
通信器配置参数结构。包含 blocking、minCTAs、maxCTAs、cgaClusterSize、netName、splitShare 等字段。
> *Configuration structure for NCCL communicator. Contains fields like blocking, minCTAs, maxCTAs, cgaClusterSize, netName, and splitShare.*

### blocking / 阻塞模式
设置为 0 表示 NCCL 在任何调用中都不阻塞（非阻塞模式），调用失败时立即返回错误。默认为 1（阻塞模式）。
> *Set to 0 to ask NCCL to never block in any NCCL call. When set to 1 (default), NCCL may block if needed.*

### minCTAs / 最小 CTA 数
每个操作使用的最小 CTA（Cooperative Thread Array，即 CUDA block）数量。
> *Minimum number of CTAs (Cooperative Thread Arrays) to use per operation.*

### maxCTAs / 最大 CTA 数
每个操作使用的最大 CTA 数量。
> *Maximum number of CTAs to use per operation.*

### cgaClusterSize / CGA 集群大小
Cooperative Group Array 集群大小，影响内核启动配置。
> *Cooperative Group Array cluster size, affecting kernel launch configuration.*

### netName / 网络名称
指定使用的网络协议。可选值如 "Socket"、"IB"（InfiniBand）、"RDMA" 等。
> *Specifies the network protocol to use, such as "Socket", "IB" (InfiniBand), or "RDMA".*

### splitShare / 分割共享
在 ncclCommSplit 中控制资源共享行为。设置为 1 表示子通信器共享资源。
> *Controls resource sharing behavior in ncclCommSplit. Set to 1 to allow sub-communicators to share resources.*

---

## 数据类型（Data Types）

### ncclDataType_t / NCCL 数据类型
支持的数据类型枚举。包括 ncclInt8、ncclUint8、ncclInt32、ncclInt64、ncclFloat16、ncclFloat32、ncclFloat64、ncclBfloat16 等。
> *Enumeration of supported data types including ncclInt8, ncclUint8, ncclInt32, ncclInt64, ncclFloat16, ncclFloat32, ncclFloat64, ncclBfloat16.*

### ncclFloat16 / FP16 数据类型
16 位浮点数据类型（IEEE 754 half precision）。
> *16-bit floating-point data type (IEEE 754 half precision).*

### ncclBfloat16 / BF16 数据类型
16 位 Brain Float 数据类型，保留 FP32 的指数范围但降低尾数精度。
> *16-bit Brain Float data type, maintaining FP32 exponent range but with reduced mantissa precision.*

### ncclFloat32 / FP32 数据类型
32 位浮点数据类型（IEEE 754 single precision）。
> *32-bit floating-point data type (IEEE 754 single precision).*

---

## 拓扑与优化（Topology and Optimization）

### ring algorithm / 环形算法
NCCL 集合操作的一种实现算法。数据沿着环形拓扑传递，带宽利用率高。
> *One implementation algorithm for NCCL collective operations. Data is passed along a ring topology with high bandwidth utilization.*

### tree algorithm / 树形算法
NCCL 集合操作的另一种实现算法。使用树形拓扑，延迟更低但带宽利用率可能较低。
> *Another implementation algorithm using a tree topology, with lower latency but potentially lower bandwidth utilization.*

### double binary tree / 双二叉树算法
结合两棵二叉树的算法，平衡延迟和带宽。
> *Algorithm combining two binary trees to balance latency and bandwidth.*

### NVLink / NVLink
NVIDIA GPU 间的高速互连技术。NCCL 自动检测并利用 NVLink 实现更高的带宽。
> *High-speed interconnect technology between NVIDIA GPUs. NCCL automatically detects and utilizes NVLink for higher bandwidth.*

### PCIe / PCIe
标准 PCI Express 互连。当 NVLink 不可用时，NCCL 通过 PCIe 进行 GPU 间通信。
> *Standard PCI Express interconnect. NCCL uses PCIe for inter-GPU communication when NVLink is unavailable.*

### NVSwitch / NVSwitch
NVIDIA 的高性能全连接 GPU 互连交换机。提供所有 GPU 间的直接高带宽连接。
> *NVIDIA's high-performance fully-connected GPU interconnect switch. Provides direct high-bandwidth connections between all GPUs.*

### topology detection / 拓扑检测
NCCL 自动检测 GPU 间的连接拓扑（NVLink、PCIe、网络等），并选择最优通信算法。
> *NCCL automatically detects the connection topology between GPUs (NVLink, PCIe, network) and selects optimal communication algorithms.*

---

## 网络通信（Network Communication）

### multi-node / 多节点
跨多台物理服务器的通信。NCCL 支持通过高速网络（InfiniBand、RoCE、TCP/IP）进行多节点集合操作。
> *Communication across multiple physical servers. NCCL supports multi-node collective operations over high-speed networks.*

### InfiniBand (IB) / InfiniBand
高性能 RDMA 网络协议。NCCL 原生支持 IB，提供最低延迟和最高带宽的多节点通信。
> *High-performance RDMA network protocol. NCCL natively supports InfiniBand for lowest latency and highest bandwidth multi-node communication.*

### RoCE (RDMA over Converged Ethernet) / 融合以太网 RDMA
在以太网上实现 RDMA 的协议。NCCL 支持 RoCE 用于多节点通信。
> *Protocol implementing RDMA over Ethernet. NCCL supports RoCE for multi-node communication.*

### GPUDirect RDMA / GPU 直接 RDMA
允许网络适配器直接访问 GPU 内存，无需通过 CPU 内存中转，降低延迟并提高带宽。
> *Allows network adapters to directly access GPU memory without going through CPU memory, reducing latency and improving bandwidth.*

### SHARP (Scalable Hierarchical Aggregation and Reduction Protocol) / 可扩展分层聚合归约协议
Mellanox/NVIDIA 交换机上的硬件加速归约操作。NCCL 可利用 SHARP 加速 AllReduce 等操作。
> *Hardware-accelerated reduction operations on Mellanox/NVIDIA switches. NCCL can leverage SHARP to accelerate AllReduce and similar operations.*

---

## 错误处理（Error Handling）

### ncclResult_t / NCCL 返回结果
NCCL 函数的返回类型。ncclSuccess 表示成功，其他值表示不同的错误类型。
> *Return type of NCCL functions. ncclSuccess indicates success, other values indicate different error types.*

### ncclSuccess / 操作成功
操作成功完成。
> *Operation completed successfully.*

### ncclInvalidArgument / 无效参数
传递给 NCCL 函数的参数无效。
> *Invalid argument passed to NCCL function.*

### ncclInvalidUsage / 无效使用
API 使用方式不正确，如未在 group 内调用点对点操作。
> *Incorrect API usage, such as calling point-to-point operations outside of a group.*

### ncclSystemError / 系统错误
底层系统调用失败。
> *Underlying system call failed.*

### ncclInternalError / 内部错误
NCCL 内部错误。
> *Internal NCCL error.*

### ncclInProgress / 操作进行中
异步操作仍在进行中。
> *Asynchronous operation is still in progress.*

### ncclGetErrorString() / 获取错误字符串
返回错误码对应的可读字符串描述。
> *Returns a human-readable string description of an error code.*

### ncclGetLastError() / 获取最后错误
返回给定通信器上的最后一个错误。用于非阻塞模式下的错误检查。
> *Returns the last error on a given communicator. Used for error checking in non-blocking mode.*

---

## 性能调优（Performance Tuning）

### NCCL_DEBUG / 调试环境变量
设置 NCCL 的调试日志级别。可选值：VERSION、WARN、INFO、TRACE。
> *Environment variable to set NCCL's debug logging level. Options: VERSION, WARN, INFO, TRACE.*

### NCCL_DEBUG_SUBSYS / 调试子系统
指定要调试的子系统，如 INIT、COLL、P2P、NET、GRAPH 等。
> *Specifies which subsystems to debug, such as INIT, COLL, P2P, NET, GRAPH.*

### NCCL_ALGO / 算法选择
强制 NCCL 使用特定算法。可选值：Ring、Tree、CollNetDirect、CollNetChain。
> *Force NCCL to use a specific algorithm. Options: Ring, Tree, CollNetDirect, CollNetChain.*

### NCCL_PROTO / 协议选择
强制 NCCL 使用特定协议。可选值：Simple、LL（Low Latency）、LL128（Low Latency 128-byte）。
> *Force NCCL to use a specific protocol. Options: Simple, LL (Low Latency), LL128 (Low Latency 128-byte).*

### NCCL_MIN_NCHANNELS / 最小通道数
设置最小通道数。增加通道数可提高带宽但增加延迟。
> *Set minimum number of channels. Increasing channels improves bandwidth but increases latency.*

### NCCL_MAX_NCHANNELS / 最大通道数
设置最大通道数。
> *Set maximum number of channels.*

### NCCL_P2P_DISABLE / 禁用点对点
禁用 GPU 间的直接点对点通信（NVLink/PCIe），强制通过主机内存中转。用于调试。
> *Disable direct peer-to-peer communication between GPUs (NVLink/PCIe), forcing communication through host memory. Used for debugging.*

### NCCL_SHM_DISABLE / 禁用共享内存
禁用同节点内通过共享内存的通信。
> *Disable shared memory communication within the same node.*

### NCCL_NET_GDR_LEVEL / GPUDirect RDMA 级别
控制 GPUDirect RDMA 的使用级别。0=禁用，1=仅 PCI，2=PCI+NVLink，3=全部启用。
> *Controls the level of GPUDirect RDMA usage. 0=disabled, 1=PCIe only, 2=PCIe+NVLink, 3=fully enabled.*

---

## 使用模式（Usage Patterns）

### single-threaded control / 单线程控制
一个主机线程控制所有 GPU。使用 cudaSetDevice 切换设备。
> *Single host thread controls all GPUs. Uses cudaSetDevice to switch between devices.*

### multi-threaded control / 多线程控制
每个 GPU 由一个独立的主机线程控制。每个线程创建自己的通信器。
> *Each GPU is controlled by a separate host thread. Each thread creates its own communicator.*

### multi-process control / 多进程控制
每个 GPU 由一个独立的进程控制。常用于 MPI 并行。每个进程创建自己的通信器。
> *Each GPU is controlled by a separate process. Commonly used with MPI parallelism. Each process creates its own communicator.*

### MPI integration / MPI 集成
NCCL 与 MPI 无缝集成。MPI 用于进程间协调和 ncclUniqueId 的广播，NCCL 用于高性能 GPU 间通信。
> *NCCL integrates seamlessly with MPI. MPI is used for inter-process coordination and ncclUniqueId broadcasting, while NCCL handles high-performance GPU-to-GPU communication.*

### data parallel training / 数据并行训练
在多个 GPU 上复制模型，每个 GPU 处理不同的数据批次。使用 AllReduce 同步梯度。
> *Replicate the model across multiple GPUs, with each GPU processing different data batches. Use AllReduce to synchronize gradients.*

### gradient averaging / 梯度平均
在数据并行训练中，使用 AllReduce(ncclSum) 对所有 GPU 的梯度求和，然后除以 GPU 数量得到平均梯度。
> *In data parallel training, use AllReduce(ncclSum) to sum gradients across all GPUs, then divide by the number of GPUs to get average gradients.*

---

## 来源文档

- NCCL Overview: https://docs.nvidia.com/deeplearning/nccl/user-guide/docs/overview.html
- NCCL Usage - Communicators: https://docs.nvidia.com/deeplearning/nccl/user-guide/docs/usage/communicators.html
- NCCL Usage - Collective Operations: https://docs.nvidia.com/deeplearning/nccl/user-guide/docs/usage/collectives.html
- NCCL Usage - Point-to-Point Communication: https://docs.nvidia.com/deeplearning/nccl/user-guide/docs/usage/p2p.html
- NCCL Usage - Group Calls: https://docs.nvidia.com/deeplearning/nccl/user-guide/docs/usage/groups.html
- NCCL Usage - CUDA Stream Semantics: https://docs.nvidia.com/deeplearning/nccl/user-guide/docs/usage/streams.html
