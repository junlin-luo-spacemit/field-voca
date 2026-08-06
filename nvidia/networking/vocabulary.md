---
tags: [nvidia, networking, vocabulary]
domain: networking
last_updated: 2026-07-22
source: https://docs.nvidia.com/networking/
---

# 网络互连词汇表

> 涵盖：NVLink、NVSwitch、InfiniBand、RoCE、RDMA、GPUDirect、SHARP、Spectrum-X、DOCA、BlueField DPU、集合通信
> 来源：https://docs.nvidia.com/networking/

---

## GPU 节点内互连（Intra-node Interconnect）

### NVLink / NVLink
NVIDIA 专有的 GPU-GPU 高速点对点互连技术，带宽远超 PCIe。

| 版本 | 对应架构 | 双向带宽（per link） | H100 总带宽 |
|------|----------|---------------------|-------------|
| NVLink 3.0 | Ampere (A100) | 100 GB/s | 600 GB/s（12 links）|
| NVLink 4.0 | Hopper (H100) | 112.5 GB/s | 900 GB/s（18 links）|
| NVLink 5.0 | Blackwell (B200) | 200 GB/s | 1.8 TB/s（18 links）|

### NVSwitch / NVSwitch
实现多 GPU 全互连（All-to-All）的专用交换芯片，每颗提供非阻塞全双工带宽。

| 代际 | 搭配架构 | per-switch 带宽 | DGX 配置 |
|------|----------|-----------------|----------|
| 第二代 | Ampere | 7.2 TB/s | DGX A100 × 6 颗 |
| 第三代 | Hopper | 13.6 TB/s | DGX H100 × 4 颗 |
| 第四代 | Blackwell | — | DGX B200 |

### NVLink-C2C (Chip-to-Chip) / NVLink 芯片间互连
NVLink 的封装级扩展，连接同一封装内或紧邻封装的芯片（CPU-GPU）。Grace Hopper GH200 通过 C2C 实现 900 GB/s CPU-GPU 带宽，延迟极低，提供统一内存地址空间。

### Grace Hopper Superchip (GH200) / Grace Hopper 超级芯片
NVIDIA Grace CPU（Arm Neoverse V2，72核）+ Hopper H100 GPU，通过 NVLink-C2C 紧耦合。HBM3e 显存 + LPDDR5X 主存共 624 GB，统一内存地址空间。

### Grace Blackwell Superchip (GB200) / Grace Blackwell 超级芯片
Grace CPU + Blackwell B200 GPU 的下一代超级芯片，通过 NVLink-C2C 互连，用于 DGX B200 / NVL72 系统。

### NVLink Network / NVLink 网络（机柜级互连）
超越节点边界的 NVLink 扩展，通过 NVLink Switch 芯片在机柜级别构建 NVLink 域（NVLink Domain），支持 GPU 跨节点直接访问，延迟比 InfiniBand 更低。NVL72 系统由 72 颗 B200 GPU 组成单一 NVLink 域。

### PCIe (Peripheral Component Interconnect Express) / PCIe
CPU 与 GPU / NIC 之间的通用总线。PCIe 5.0 × 16 提供约 64 GB/s 双向带宽，远低于 NVLink，是节点内非 NVLink 场景的潜在瓶颈。

---

## 数据中心网络技术（Data Center Networking）

### InfiniBand / InfiniBand
用于 HPC 和 AI 数据中心的高带宽、极低延迟网络互连技术（NVIDIA 通过收购 Mellanox 提供）。具备原生 RDMA 支持，是 AI 大规模训练的主流选择。

| 标准 | 单端口带宽 | 典型延迟 |
|------|-----------|---------|
| HDR InfiniBand | 200 Gb/s | < 600 ns |
| NDR InfiniBand | 400 Gb/s | < 500 ns |
| XDR InfiniBand | 800 Gb/s | — |

### RDMA (Remote Direct Memory Access) / 远程直接内存访问
允许一台计算机的 NIC 直接读写另一台计算机的内存，完全绕过远端 CPU 和操作系统。延迟 < 1 µs，减少 CPU 开销。RDMA 操作分为：
- **Two-sided**（双边）：Send/Receive，双方均需 CPU 参与
- **One-sided**（单边）：Read/Write/Atomic，仅发起方 CPU 参与，目标方零感知

### RoCE (RDMA over Converged Ethernet) / 以太网 RDMA
在标准以太网上运行 RDMA 协议的技术。
- **RoCE v1**：基于以太网帧，不可路由
- **RoCE v2**：基于 UDP/IP，可跨子网路由（主流版本）
- 需要无损以太网（PFC + ECN）配合使用

### iWARP (Internet Wide Area RDMA Protocol) / iWARP
在 TCP/IP 上实现 RDMA，延迟略高于 RoCE，但对网络质量要求更低。

### libibverbs / libibverbs
Linux 用户空间 RDMA 编程库，提供统一的 InfiniBand / RoCE 编程接口（RDMA Verb 操作）。

### Quantum InfiniBand Switch / Quantum InfiniBand 交换机
NVIDIA 高性能 InfiniBand 交换机产品系列：
- **Quantum**（HDR）：200 Gb/s，40 端口
- **Quantum-2**（NDR）：400 Gb/s，64 端口，内置 SHARP v3 + 自适应路由

### Spectrum Ethernet Switch / Spectrum 以太网交换机
NVIDIA 数据中心以太网交换机产品系列，支持 RoCE 无损网络和 Spectrum-X 功能集。

### Spectrum-X / Spectrum-X
NVIDIA 专为 AI 工作负载设计的以太网平台，由 Spectrum-4 交换机 + BlueField-3 SuperNIC 组成。提供与 InfiniBand 相近的 AI 网络性能，适合大规模以太网 AI 集群。

---

## GPUDirect 技术族

### GPUDirect RDMA (GDR) / GPU 直连 RDMA
允许第三方设备（InfiniBand NIC、存储控制器）通过 PCIe 直接读写 GPU 显存，绕过 CPU 和系统内存。

**数据路径对比：**
- 标准路径：GPU 显存 → 系统内存 → NIC → 网络
- GDR 路径：GPU 显存 → NIC（直接） → 网络（降低延迟约 30%）

### GPUDirect Storage (GDS) / GPU 直连存储
允许存储设备（NVMe、网络存储）通过 DMA 直接读写 GPU 显存，绕过 CPU 和系统内存，大幅降低数据加载延迟。使用 `cuFile` API 编程。

### GPUDirect P2P (Peer-to-Peer) / GPU 点对点
同一 PCIe / NVLink 域内的 GPU 之间直接内存访问，无需经过 CPU。通过 `cudaDeviceEnablePeerAccess()` 启用。

### GPUDirect Video / GPU 直连视频
允许视频采集设备直接将帧数据写入 GPU 显存，用于实时视频处理场景。

---

## 网络内计算（In-Network Computing）

### SHARP (Scalable Hierarchical Aggregation and Reduction Protocol) / 可扩展分层聚合规约协议
在 InfiniBand Quantum 交换机内部执行集合操作（AllReduce 等），数据在传输过程中即完成规约，无需到达目标节点后再处理。
- 延迟与集群规模无关（O(1) vs 标准 O(log N)）
- All-Reduce 带宽提升可达 2×
- 需要 Quantum / Quantum-2 交换机硬件支持

### SHARP v3 / SHARP v3
Quantum-2（NDR）内置的第三代 SHARP，支持更大消息尺寸（up to 1 MB）和更多并发操作。

### in-network computing / 网络内计算
广义上指在交换机、路由器或智能 NIC 上执行计算任务，SHARP 是 NVIDIA 的典型实现。

---

## BlueField DPU 与 DOCA

### DPU (Data Processing Unit) / 数据处理单元
NVIDIA BlueField 系列网卡，集成网络处理器、ARM 核心和硬件加速引擎，作为独立"基础设施处理器"处理网络、存储、安全任务，释放主机 CPU。被称为"三大处理器"之一（CPU / GPU / DPU）。

| 型号 | 网络接口 | ARM 核心 | 特性 |
|------|----------|---------|------|
| BlueField-2 | 100GbE / HDR IB | 8× A72 | 网络/存储卸载 |
| BlueField-3 | 400GbE / NDR IB | 16× A78AE | 零信任安全、AI加速 |
| BlueField-3 SuperNIC | 400GbE | 无 ARM | Spectrum-X 专用，纯网络加速 |

### SmartNIC / 智能网卡
具备可编程处理能力的网络接口卡，BlueField 是含完整 SoC 的高端形态。

### DOCA (Data-center Infrastructure-on-a-Chip Architecture) / 数据中心片上基础设施架构
NVIDIA 基于 BlueField DPU 的软件开发框架，提供 SDK、库和参考应用，用于在 DPU 上开发网络、安全、存储服务。

**DOCA 核心组件：**
- **DOCA Flow**：数据包处理流水线，类似 P4 编程模型
- **DOCA Firefly**：精确时间协议（PTP/IEEE 1588）
- **DOCA Crypto**：IPsec、TLS 硬件加速
- **DOCA DMA**：主机与 DPU 间高效数据传输
- **DOCA App Shield**：基于 DPU 的进程完整性监控（零信任）

### offload / 卸载
将原本由主机 CPU 执行的任务（网络处理、加密、压缩、存储 I/O）转移到 DPU / SmartNIC 执行，释放 CPU 算力用于应用层。

### OVS (Open vSwitch) offload / OVS 卸载
将虚拟化网络的 OVS 数据平面从主机 CPU 卸载到 BlueField DPU 上执行，是云计算基础设施的核心优化手段。

### Isolated Mode vs Embedded Mode / 隔离模式 vs 嵌入模式
- **Embedded Mode**：DPU 作为主机 NIC 工作，主机可访问 DPU 的 ARM 核心
- **Isolated Mode（零信任）**：DPU 独立于主机，主机无法干预 DPU 上的基础设施服务，是安全场景推荐模式

---

## 集合通信（Collective Communication）

### NCCL (NVIDIA Collective Communication Library) / NVIDIA 集合通信库
→ 详见 [[nccl/vocabulary]]

### All-Reduce / 全规约
将所有参与节点的张量逐元素规约（求和/平均），并将结果广播回所有节点。数据并行训练梯度同步的核心操作。

### All-Gather / 全收集
每个节点将自己的分块数据发给所有其他节点，最终每个节点拥有完整数据。用于模型并行前向、ZeRO stage 2/3 参数还原。

### Reduce-Scatter / 规约分散
将所有节点数据规约后，每个节点只保留结果的一个分块。与 All-Gather 组合即为 Ring All-Reduce。用于 ZeRO 梯度更新。

### Broadcast / 广播
将一个节点的数据发送给所有其他节点。用于参数初始化、模型权重同步。

### All-to-All / 全对全通信
每个节点向所有其他节点发送不同数据，同时从所有节点接收不同数据。MoE 模型专家路由的核心通信原语。

### Ring All-Reduce / 环形全规约
将参与节点排成逻辑环，通过两轮（Reduce-Scatter + All-Gather）完成 All-Reduce。带宽利用率最优（趋近 100%），是 NCCL 的默认大消息算法。

### Tree All-Reduce / 树形全规约
使用二叉树拓扑完成 All-Reduce，延迟与节点数成 O(log N) 关系，适合小消息场景。

---

## 网络性能与调优

### bandwidth / 带宽
单位时间内可传输的数据量（GB/s 或 Gb/s）。注意区分理论峰值与实测有效带宽，以及单向/双向。

### latency / 延迟
数据从发送端到接收端所需时间（µs 或 ns）。InfiniBand < 1 µs，PCIe NIC 约 5–10 µs。

### message rate / 消息速率
每秒可完成的小消息传输次数（Mmsg/s），衡量 NIC 小包处理能力，与延迟直接相关。

### congestion control / 拥塞控制
防止网络过载的机制：InfiniBand 使用自适应路由 + FECN/BECN；RoCE 需配合 PFC + ECN。

### PFC (Priority Flow Control) / 优先级流量控制
802.1Qbb 标准，以太网无损传输的基础机制。通过 PAUSE 帧停止特定优先级队列的发送，防止缓冲区溢出丢包。RoCE 依赖 PFC 实现无损网络。

### ECN (Explicit Congestion Notification) / 显式拥塞通知
路由器在队列接近满时标记数据包，接收方将拥塞信息反馈给发送方主动降速，比 PFC 更精细，是 RoCE v2 拥塞控制的重要机制。

### adaptive routing / 自适应路由
根据网络实时状态动态选择最优路径，减少热点和拥塞，提升大规模训练的整体吞吐量。Quantum-2 交换机内置支持。

---

## 存储与工具

### NVMe-oF (NVMe over Fabrics) / NVMe 网络传输
将 NVMe 协议扩展到网络上（通过 InfiniBand RDMA 或 RoCE），实现以接近本地延迟访问远程 NVMe SSD，用于分布式存储和 AI 数据管道。

### cuFile API / cuFile API
GPUDirect Storage 的用户空间编程接口，允许应用程序发起存储设备到 GPU 显存的直接 DMA 传输。

### MLNX_OFED / Mellanox OpenFabrics Enterprise Distribution
NVIDIA（原 Mellanox）提供的 InfiniBand / RoCE 驱动和完整软件栈，含 RDMA 核心驱动、性能测试工具和 MPI 支持库。

### perftest / 性能测试工具集
MLNX_OFED 附带的 RDMA 基准测试工具集，常用命令：
- `ib_write_bw` / `ib_read_bw`：测量 RDMA 写/读带宽
- `ib_write_lat` / `ib_read_lat`：测量 RDMA 写/读延迟
- `ib_send_bw`：测量双边 Send 带宽
