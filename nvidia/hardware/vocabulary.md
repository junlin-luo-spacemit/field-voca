---
tags: [nvidia, hardware, vocabulary]
domain: hardware
last_updated: 2026-07-21
source: https://docs.nvidia.com/
---

# 硬件平台词汇表

> 涵盖：DGX、Jetson、vGPU、GPU 架构
> 来源：https://docs.nvidia.com/

---

## GPU 架构（GPU Architecture）

### GPU Architecture Generations / GPU 架构世代

| 架构代号 | 发布年份 | 代表产品 | 重要特性 |
|----------|----------|----------|----------|
| Volta | 2017 | V100 | 首代 Tensor Core |
| Turing | 2018 | RTX 20 系列 | RT Core（光线追踪），INT8/INT4 |
| Ampere | 2020 | A100, RTX 30 系列 | 第三代 Tensor Core，MIG，NVLink 3.0 |
| Hopper | 2022 | H100 | 第四代 Tensor Core，Transformer Engine，NVLink 4.0 |
| Ada Lovelace | 2022 | RTX 40 系列 | 第四代 RT Core，DLSS 3 |
| Blackwell | 2024 | B200, GB200 | 第五代 Tensor Core，FP4，NVLink 5.0 |

### die / 芯片裸片
封装前的半导体芯片本体。H100 SXM 使用单裸片；GB200 NVL72 使用多裸片互连。

### package / 封装
将裸片固定在基板上并引出引脚的工艺。常见封装：SXM（服务器模块）、PCIe、NVL。

### TDP (Thermal Design Power) / 热设计功耗
芯片在持续满负载时的最大散热需求，H100 SXM5 TDP 为 700W。

### NVLink / NVLink
NVIDIA 高速 GPU-GPU 互连技术，H100 支持 NVLink 4.0，双向带宽 900 GB/s。

### NVSwitch / NVSwitch
NVIDIA GPU 互连交换芯片，DGX H100 内置 4 个 NVSwitch，实现 8 GPU 全互连。

### PCIe (Peripheral Component Interconnect Express) / 高速串行总线
CPU-GPU 互连标准，PCIe 5.0 x16 带宽约 128 GB/s（双向）。

### SXM / SXM
NVIDIA 服务器 GPU 专用高带宽接口模块，通过 NVLink 高速连接，性能优于 PCIe 版本。

### HBM (High Bandwidth Memory) / 高带宽内存
通过硅通孔（TSV）堆叠的 DRAM，封装与 GPU 同一基板，极低延迟、极高带宽。
| 型号 | HBM 版本 | 显存大小 | 带宽 |
|------|----------|----------|------|
| A100 80GB SXM | HBM2e | 80 GB | 2.039 TB/s（官方 nvidia.com/a100 核实） |
| H100 SXM5 | HBM3 | 80 GB | 3.35 TB/s（官方核实） |
| H200 SXM/NVL | HBM3e | 141 GB | 4.8 TB/s（官方 nvidia.com/h200 核实） |
| B200 | HBM3e | ⚠️ 待核实 | ⚠️ 待核实（官方逐项规格页暂未成功抓取，勿凭记忆填写，见 hardware/notes.md 说明） |

### MIG (Multi-Instance GPU) / 多实例 GPU
A100/H100 的虚拟化特性，将单个 GPU 硬件切分为最多 7 个完全隔离的 GPU 实例。

---

## DGX 平台

### DGX / DGX
NVIDIA 面向企业 AI 训练与推理的集成系统，包含 GPU、CPU、内存、存储、网络的完整解决方案，"通用系统"（universal system）定位，覆盖分析、训练、推理全场景。

| 型号 | GPU | GPU 数量 | GPU 互连 |
|------|-----|----------|----------|
| DGX A100 | A100 80GB | 8 | NVLink 3.0 + NVSwitch |
| DGX H100 | H100 80GB（共 640GB） | 8 | NVLink 4.0（4×NVSwitch, 900 GB/s） |
| DGX H200 | H200（共 1,128GB） | 8 | NVLink 4.0（4×NVSwitch, 900 GB/s） |
| DGX B200 | B200 192GB | 8 | NVLink 5.0 + NVSwitch |

### DGX H100/H200 硬件规格速查
| 组件 | 规格 |
|------|------|
| CPU | 2× Intel Xeon 8480C（56 核，PCIe Gen5） |
| 系统内存 | 2TB（32×DIMM） |
| OS 存储 | 2× 1.92TB NVMe M.2（RAID 1） |
| 数据缓存存储 | 8× 3.84TB NVMe U.2 SED（RAID 0） |
| 集群网络卡 | 4× OSFP，8× ConnectX-7 单口 IB 卡，单卡最高 400Gbps |
| 存储/管理网络卡 | 2× ConnectX-7 双口以太网卡 |
| 电源 | 6× 3.3kW（4+2 冗余） |
| 形态 | 8U 机架式，约 130kg |

### BMC (Baseboard Management Controller) / 基板管理控制器
DGX 系统的带外（out-of-band）管理芯片，支持 Redfish、IPMI、SNMP、KVM 等协议，独立于主系统电源状态运行。

### NVSM (NVIDIA System Management) / NVIDIA 系统管理
为 DGX 节点提供主动健康监控和系统告警的软件，支持命令行健康检查。

### DCGM (Data Center GPU Manager) / 数据中心 GPU 管理器
提供节点级和集群级 GPU 管理能力的软件，用于监控、诊断和管理数据中心规模的 GPU 部署。

### DGX SuperPOD / DGX SuperPOD
由多个 DGX 节点和 NVIDIA Quantum InfiniBand 网络组成的超级计算集群参考架构。

### InfiniBand / InfiniBand
高性能计算（HPC）和 AI 数据中心常用的高速网络互连技术，NVIDIA 通过 Mellanox（现 ConnectX 系列）提供。

### DensiLink / DensiLink
DGX H100/H200 中用于将 ConnectX-7 网卡直连到背板 OSFP 接口的高密度线缆技术，取代传统独立网卡走线方式。

---

## Jetson 平台（边缘 AI）

### Jetson / Jetson
NVIDIA 边缘 AI 计算模块系列，将 GPU、CPU、内存集成在紧凑的 SOM（System on Module）上，面向自主机器（Autonomous Machines）场景。

| 型号 | 定位 | GPU 架构 | 备注 |
|------|------|----------|------|
| Jetson Nano | 入门级 | Maxwell | 已停产，教育场景常用 |
| Jetson Xavier NX | 中端 | Volta | 上一代边缘推理主力 |
| Jetson Orin Nano | 入门/嵌入式 | Ampere | 8GB/4GB 商用版，含带 SD 卡槽版本 |
| Jetson Orin NX | 中端 | Ampere | 16GB/8GB 商用模组 |
| Jetson AGX Orin | 旗舰 | Ampere (2048 CUDA Cores) | 32GB/64GB 商用模组，另有工业级版本 |

### developer kit vs module / 开发者套件 与 生产模组
Jetson 开发者套件（developer kit）搭配非量产规格模组和参考载板，仅用于开发调试；生产模组（module）单独销售、不预装软件，需配合客户自研载板并自行刷写系统镜像。

### P-number / P 编号
Jetson 模组和开发者套件的物料编号体系，源自其 699 级零件号，用于唯一标识具体型号（如 AGX Orin 64GB 为 P3701-0005）。

### JetPack SDK / JetPack SDK
Jetson 平台的完整软件开发包，包含 Jetson Linux（L4T）及加速库、API、示例应用、开发工具和文档。

### L4T (Linux for Tegra) / Tegra Linux
Jetson 平台的定制化 Linux 操作系统，由 NVIDIA 维护，提供内核、Bootloader、驱动、刷写工具等。

### SDK Manager / SDK 管理器
用于在 Jetson 开发者套件上安装 Jetson Linux 及其他 JetPack 组件的官方主机端工具。

### flashing / 刷写
将 Bootloader、内核、文件系统等镜像写入 Jetson 模组存储介质的过程，可通过 `flash.sh` 脚本或 SDK Manager 完成。

### A/B redundancy / A/B 冗余分区
Jetson 的双槽（slot A/slot B）启动与文件系统冗余机制，支持 OTA 更新失败时自动回退到可用槽位。

---

## GPU 虚拟化（vGPU）

### vGPU (Virtual GPU) / 虚拟 GPU
将物理 GPU 切分为多个虚拟 GPU，分配给不同虚拟机，实现 GPU 资源共享。

### vGPU Manager / vGPU 管理器
安装在 Hypervisor 层的组件，负责创建和管理虚拟 GPU 实例，是 vGPU 软件栈的核心。

### GRID vGPU / GRID vGPU
NVIDIA vGPU 软件的早期名称，现统称 NVIDIA Virtual GPU Software。

### hypervisor / 虚拟机监控器
运行虚拟机的软件层（如 VMware vSphere、Microsoft Hyper-V、Citrix XenServer）。

### SR-IOV (Single Root I/O Virtualization) / 单根 I/O 虚拟化
PCI-SIG 标准，允许单个 PCIe 设备对多个虚拟机呈现为独立的虚拟功能（VF）。

### vWS (Virtual Workstation) / 虚拟工作站
vGPU 产品类型之一，面向专业图形应用，支持完整的 NVIDIA RTX 驱动特性集。

### vCS (Virtual Compute Server) / 虚拟计算服务器
vGPU 产品类型之一，面向计算服务器场景，用于 CUDA/AI 计算负载，不含图形驱动特性。

### vPC / vApps / 虚拟 PC 与虚拟应用
面向虚拟桌面（VDI）和应用虚拟化场景的 vGPU 产品类型，定位低于 vWS。

### Production Branch vs Long-Term Support Branch / 生产分支 与 长期支持分支
vGPU 软件发布策略：Production Branch 支持周期约 1 年，面向需要新特性的场景；Long-Term Support Branch 支持周期约 3 年，面向生产环境的稳定性需求。

### time-sliced vGPU / 时分复用 vGPU
多个虚拟机通过时间片轮转共享同一物理 GPU 的传统 vGPU 调度模式。

### MIG-backed vGPU / MIG 支持的 vGPU
在支持 MIG 的 GPU（如 A100/H100）上，将 vGPU 实例绑定到硬件隔离的 MIG 分区，兼具虚拟化灵活性与硬件级资源隔离。
