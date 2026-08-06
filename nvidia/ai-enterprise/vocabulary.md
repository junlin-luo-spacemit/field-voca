---
tags: [nvidia, ai-enterprise, vocabulary]
domain: ai-enterprise
last_updated: 2026-07-21
part: Overview + Software + Lifecycle ✅
source: https://docs.nvidia.com/ai-enterprise/
---

# NVIDIA AI Enterprise 词汇表

> 来源：https://docs.nvidia.com/ai-enterprise/
> 格式：英文术语 → 中文译名 + 官方定义 + 例句

---

## 核心概念（Core Concepts）

### NVIDIA AI Enterprise / NVIDIA AI 企业版
NVIDIA 的完整企业级软件平台，加速 AI 全生命周期（从原型到生产），覆盖云、数据中心、边缘环境。由 Application（应用开发）和 Infrastructure（基础设施管理）两层组成，均提供带 SLA 的企业级支持。
> *NVIDIA AI Enterprise is a software platform that accelerates every stage of the AI lifecycle - from prototyping to production - across cloud, data center, and edge environments.*

### composable stack / 可组合技术栈
NVIDIA AI Enterprise 的架构原则：基础组件对所有部署通用，其余组件根据具体用例自由组装。
> *NVIDIA AI Enterprise is a composable stack - assemble the Application Development and Infrastructure Management layers based on your use case.*

### Application Development layer / 应用开发层
包含 NIM 微服务、NeMo 智能体工具、Omniverse 物理 AI 库、AI 框架和 ML 库，均构建在 CUDA-X 和 CUDA 之上。
> *Application Development - NIM microservices, NeMo tools for agentic AI, Omniverse libraries for physical AI, and AI frameworks and ML libraries, all built on CUDA-X and CUDA.*

### Infrastructure Management layer / 基础设施管理层
包含 GPU 驱动、Run:ai 工作负载编排、vGPU/MIG GPU 分区、Kubernetes operators、Base Command Manager 集群管理工具。
> *Infrastructure Management - GPU drivers, Run:ai for workload orchestration, vGPU and MIG for GPU partitioning, Kubernetes operators, and Base Command Manager for deploying and managing GPU infrastructure at scale.*

### independent release cadence / 独立发布节奏
Application 层和 Infrastructure 层拥有各自独立的发布周期和生命周期，使组织能在不打乱已验证基础设施的前提下采用新的应用能力。
> *The platform is organized into two layers with independent release cadences, allowing organizations to adopt new application capabilities without disrupting validated infrastructure.*

### Entitlement Certificate / 授权证书
用于激活 NVIDIA Enterprise Account、访问 NGC Catalog、Support Portal 和 Licensing Portal 的凭证。
> *You've redeemed your Entitlement Certificate, created your NVIDIA Enterprise Account, and have access to the NVIDIA NGC Catalog.*

---

## 发布分支类型（Release Branch Types）

### Feature Branch (FB) / 特性分支
提供最新上游特性和优化的分支类型，支持窗口仅 1 个月（下一个月发布即取代当前版本）。适合开发、测试、概念验证和研究场景。
> *Feature Branch (FB) picks up the latest upstream releases, including those from short-lived development branches. FB support is limited to one month before the next release supersedes it.*

### Production Branch (PB) / 生产分支
面向任务关键型应用的分支类型，提供 9 个月支持窗口，每月安全补丁，每 6 个月发布主版本。API 在支持窗口内保持稳定。
> *Production Branch (PB): mission-critical applications needing API stability. 9 months support, monthly security patches, major release every 6 months.*

### Long-Term Support Branch (LTSB) / 长期支持分支
面向高度监管行业的分支类型，提供 3 年支持窗口，每季度安全补丁。适合医疗、金融、政府等需要长期认证周期的场景。
> *Long-Term Support Branch (LTSB): highly regulated industries needing extended support. 3 years support, quarterly security patches.*

### Infrastructure Branch / 基础设施分支
面向 GPU 驱动、Kubernetes operators、容器运行时等基础设施组件的分支类型。标准支持 1 年，可指定为 LTSB Infrastructure Release 延长至 3 年。
> *Infrastructure Branch: GPU drivers, operators, or infrastructure stack components. 1 year support (3 years for LTSB Infrastructure releases).*

### upstream software / 上游软件
NVIDIA AI Enterprise 所基于的开源项目（如 PyTorch、NVIDIA RAPIDS、TensorRT）。PB/LTSB 仅采用来自长期维护上游分支的版本，以保证补丁可持续交付。
> *NVIDIA AI Enterprise curates software from upstream open-source projects such as PyTorch, NVIDIA RAPIDS, and NVIDIA TensorRT.*

### EOL (End of Life) / 生命周期终止
分支或版本停止维护、不再获得更新和支持的状态。文档提供 EOL Notices 帮助规划迁移。
> *End of Life Notices: Review for any upcoming deprecations and plan migrations before branches reach end-of-life.*

### Government Ready containers / 政府就绪容器
仅在 PB 和 LTSB 提供的 STIG 加固、FIPS 启用的容器，用于满足 FedRAMP High / 主权部署要求。
> *Government Ready Containers (STIG-hardened, FIPS-enabled) are available on Production Branch (PB) and Long-Term Support Branch (LTSB) only.*

### VEX (Vulnerability Exploitability eXchange) / 漏洞可利用性交换
CycloneDX 格式的文件，说明已知开源软件漏洞对 NVIDIA 容器镜像的实际影响分析结果。
> *The analysis can be accessed through the NGC Container Scanning UI and is downloadable as a Vulnerability Exploitability eXchange (VEX) file in CyclonDX format.*

---

## 基础设施软件组件（Infrastructure Software Components）

### NVIDIA GPU Data Center Driver / GPU 数据中心驱动
为 NVIDIA GPU 提供硬件支持的核心驱动，是所有 GPU 加速工作负载的基础。
> *Provides hardware support for NVIDIA GPUs.*

### NVIDIA Virtual GPU Manager / 虚拟 GPU 管理器
部署在 Hypervisor 中的 GPU 驱动，支持虚拟化环境中的多租户 GPU 共享、实时迁移和监控。
> *NVIDIA GPU driver deployed in the hypervisor for virtualized environments. Enables multi-tenant GPU sharing, live migration, and monitoring.*

### vGPU for Compute Guest Driver / vGPU 计算客户机驱动
部署在虚拟机或裸机 OS 中的 GPU 驱动，使多个虚拟机能同时直接访问单个物理 GPU。
> *NVIDIA GPU driver deployed in the VM or on bare metal OS to enable multiple VMs to have simultaneous, direct access to a single physical GPU.*

### MIG (Multi-Instance GPU) / 多实例 GPU
将单个物理 GPU 划分为多个独立 GPU 实例的硬件级分区技术，每个实例有专属计算核心和显存，提供硬件级隔离。
> *vGPU and MIG for GPU partitioning.*

### NVIDIA Container Toolkit / NVIDIA 容器工具包
为容器引擎（Docker、containerd、CRI-O）提供 GPU 加速容器所需运行时组件和工具的软件包。
> *Enables GPU-accelerated containers by providing runtime components and utilities for container engines.*

### NVIDIA Run:ai / NVIDIA Run:ai
Kubernetes 原生的编排和管理平台，通过高级调度和资源管理最大化 AI 工作负载的 GPU 利用率。提供 Self-Hosted（客户自管控制面）和 SaaS（NVIDIA 托管云服务）两种部署方式。
> *Provides a Kubernetes-native orchestration and management platform that maximizes GPU utilization for AI workloads through advanced scheduling and resource management.*

### NVIDIA GPU Operator / GPU Operator
自动化管理 Kubernetes 中 GPU 配置所需全部 NVIDIA 软件组件的工具，简化 NVIDIA AI Enterprise 部署。
> *Simplifies deployment of NVIDIA AI Enterprise by automating management of all NVIDIA software components needed to provision GPUs in Kubernetes.*

### NVIDIA Network Operator / 网络 Operator
自动化管理 NVIDIA ConnectX NIC 和 SuperNIC 的工具，优化 Kubernetes 中的东西向流量和 RDMA 传输。
> *Simplifies deployment of high-speed networking by automating management of NVIDIA ConnectX NICs and SuperNICs required to optimize East-West traffic and RDMA transfers in Kubernetes.*

### NVIDIA DPU Operator (DPF) / DPU Operator（DOCA Platform Framework）
自动化 BlueField DPU 和 DOCA 微服务的配置、编排和生命周期管理的工具，为 Kubernetes 提供 DPU 加速的南北向网络能力。
> *Enables cluster administrators to automate provisioning, orchestration, and lifecycle management of BlueField DPUs and DOCA Microservices.*

### NVIDIA NIM Operator / NIM Operator
使集群管理员能在 Kubernetes 中操作运行 LLM、embedding 等模型所需的 NIM 微服务软件组件和服务的工具（已在 NIM 章节学习）。
> *Enables cluster administrators to operate the software components and services required to run LLM, embedding, and other models using NVIDIA NIM microservices in Kubernetes.*

### NVIDIA Fabric Manager / Fabric Manager
管理 DGX/HGX 等多 GPU 系统中基于 NVSwitch 的 GPU 到 GPU 互连的组件。
> *Manages NVSwitch-based GPU-to-GPU interconnects in multi-GPU systems such as DGX and HGX platforms.*

### NVIDIA Base Command Manager (BCM) / Base Command Manager
用于 DGX 系统的集群管理和配置工具，不通过 NGC 分发，需企业许可支持。
> *NVIDIA Base Command Manager (BCM): Cluster management and provisioning tool for NVIDIA DGX systems.*

### NVIDIA DOCA (Data Center-on-a-Chip Architecture) / DOCA
BlueField DPU 的软件框架，提供网络、存储和安全工作负载加速与卸载服务。
> *NVIDIA DOCA Microservices: Infrastructure acceleration and offload services for NVIDIA BlueField, enabling accelerated networking, storage, and security workloads.*

---

## 支持与合规（Support and Compliance）

### SLA (Service Level Agreement) / 服务级别协议
NVIDIA AI Enterprise 各组件企业级支持所承诺的响应时间和服务质量保证。
> *All components include enterprise support with SLAs, security patches, and maintenance updates.*

### Support Matrix / 支持矩阵
用于验证部署配置（GPU 架构、操作系统、Hypervisor、Kubernetes 发行版、云服务商）是否受支持的交互式工具。
> *Use the support matrix to verify supported configurations for your deployment - filter by deployment type, OS, hypervisor, orchestration platform, GPU architecture, Kubernetes distribution, or cloud provider.*

### Interactive Lifecycle and Compatibility Explorer / 交互式生命周期与兼容性浏览器
基于网页的工具，用于验证 GPU 驱动、Operator、Run:ai 等组件版本间的跨栈兼容性，替代手动交叉对比多个文档页面。
> *The Interactive Lifecycle and Compatibility Explorer replaces manual cross-referencing of compatibility tables across multiple pages.*

### FedRAMP / 联邦风险与授权管理计划
美国联邦政府云服务安全认证标准。Government Ready containers 面向 FedRAMP High 或同等主权部署场景。
> *For FedRAMP High / Sovereign use cases, choose PB or LTSB, then follow the Government Ready deployment guidance.*

### FIPS (Federal Information Processing Standards) / 联邦信息处理标准
美国联邦政府对加密模块的安全标准。Government Ready containers 启用 FIPS 加密。
> *STIG-hardened, FIPS-enabled container details for government deployment.*

### STIG (Security Technical Implementation Guide) / 安全技术实施指南
美国国防部的安全加固配置标准。Government Ready containers 采用 STIG 加固。
> *Government Ready Containers - STIG-hardened, FIPS-enabled.*

---

## 部署路径（Deployment Paths）

### bare metal deployment / 裸机部署
直接在物理服务器操作系统上部署 GPU 驱动和工作负载，不经过虚拟化层。
> *The Quick Start Guide covers three deployment paths: Bare Metal, Virtualized (vGPU), and Public Cloud.*

### virtualized deployment (vGPU) / 虚拟化部署
通过 vGPU 技术在虚拟机环境中部署，支持多租户共享物理 GPU 资源。
> *Virtualized (vGPU) deployment path for NVIDIA AI Enterprise.*

### public cloud deployment / 公共云部署
在 AWS、Azure、GCP 等公共云平台上部署 NVIDIA AI Enterprise 组件。
> *Public Cloud deployment path covered in the Quick Start Guide.*

### reference architecture / 参考架构
针对特定部署场景（裸机、虚拟化、云）的标准化架构设计和分步部署指南。
> *Planning & Deployment provides reference architectures and step-by-step deployment guides for bare metal, virtualized, or cloud environments.*

---

## 生态整合（Ecosystem Integration）

### NGC (NVIDIA GPU Cloud) Catalog / NGC 目录
NVIDIA AI Enterprise 所有受支持软件（AI 框架、微服务、预训练模型、SDK、工具）的分发平台，提供企业级支持。
> *NVIDIA NGC Catalog: Lists all NVIDIA AI Enterprise supported software available on NGC.*

### SUSE AI Factory with NVIDIA / SUSE AI Factory 合作
NVIDIA AI Enterprise 通过 SUSE AI Factory 提供的合作伙伴主导支持模式，SUSE 直接为部署在 SLES 上的客户提供技术支持。
> *NVIDIA AI Enterprise is also available through SUSE AI Factory with NVIDIA, where SUSE delivers technical support directly for deployments on SLES.*

### NVIDIA-Certified Servers / NVIDIA 认证服务器
经过标准化 GPU passthrough 和 vGPU 计算资质测试的 OEM 硬件服务器认证。
> *Virtualization Software Qualification Test Plan: Standardized GPU passthrough and vGPU for Compute qualification tests for NVIDIA-Certified Servers.*

---

## 来源文档

- NVIDIA AI Enterprise 主页: https://docs.nvidia.com/ai-enterprise/index.html
- NVIDIA AI Enterprise Software: https://docs.nvidia.com/ai-enterprise/software/latest/overview.html
- Infrastructure Layer Software: https://docs.nvidia.com/ai-enterprise/software/latest/infrastructure-software.html
- NVIDIA AI Enterprise Lifecycle Policy: https://docs.nvidia.com/ai-enterprise/lifecycle/latest/index.html
- Choosing the Right Release Branch: https://docs.nvidia.com/ai-enterprise/lifecycle/latest/choosing-a-branch.html
