# 硬件平台学习笔记

> 文档入口：
> - DGX: https://docs.nvidia.com/dgx/index.html
> - Jetson: https://docs.nvidia.com/jetson/index.html
> - vGPU: https://docs.nvidia.com/vgpu/index.html

---

## NVIDIA GPU 产品线划分

```
数据中心 (Data Center)
├── H100 / H200 (Hopper) — AI 训练 + 推理旗舰
├── B200 / GB200 (Blackwell) — 新一代 AI 超算
├── A100 (Ampere) — 上一代主力训练卡
└── L40S / L4 (Ada) — 推理优化型

专业图形 (Professional Visualization)
├── RTX 6000 Ada — 工作站 GPU
└── RTX 4500 / 4000 Ada

消费级 (Consumer)
└── GeForce RTX 40 系列 (Ada Lovelace)

边缘计算 (Edge / Embedded)
└── Jetson AGX Orin / Xavier NX / Nano
```

---

## H100 关键规格速查

| 参数 | H100 SXM5 | H100 PCIe |
|------|-----------|-----------|
| 架构 | Hopper (GH100) | Hopper (GH100) |
| CUDA Cores | 16,896 | 14,592 |
| Tensor Cores | 528 (第4代) | 456 (第4代) |
| 显存 | 80GB HBM3 | 80GB HBM2e |
| 显存带宽 | 3.35 TB/s | 2.0 TB/s |
| FP16 算力 | 989 TFLOPS | 756 TFLOPS |
| FP8 算力 | 1,979 TOPS | 1,513 TOPS |
| NVLink 带宽 | 900 GB/s | — |
| TDP | 700W | 350W |

---

## A100 关键规格速查（官方 nvidia.com/a100 核实，2026-08-07）

| 参数 | A100 40GB PCIe | A100 80GB SXM |
|------|-----------------|----------------|
| 架构 | Ampere (GA100) | Ampere (GA100) |
| FP64 | 9.7 TFLOPS | 9.7 TFLOPS |
| FP64 Tensor Core | 19.5 TFLOPS | 19.5 TFLOPS |
| FP32 | 19.5 TFLOPS | 19.5 TFLOPS |
| TF32 Tensor Core | 156 TFLOPS（312 TFLOPS 稀疏） | 312 TFLOPS（稀疏） |
| BF16/FP16 Tensor Core | 312 TFLOPS（624 稀疏） | 624 TFLOPS（稀疏） |
| INT8 Tensor Core | 624 TOPS（1248 稀疏） | 1248 TOPS（稀疏） |
| 显存 | 80GB HBM2e | 80GB HBM2e |
| 显存带宽 | 1,935 GB/s | 2,039 GB/s |
| Max TDP | 300W | 400W（HGX 定制散热 SKU 最高 500W） |
| 互连 | NVLink Bridge 600 GB/s（限2 GPU）/ PCIe Gen4 64 GB/s | NVLink 600 GB/s / PCIe Gen4 64 GB/s |
| MIG | 最多 7 个实例 @ 10GB | 最多 7 个实例 @ 10GB |

> 注：TF32/BF16/FP16/INT8 括号内数值为启用**结构化稀疏（sparsity）**后的峰值，非稠密算力。

## H200 关键规格速查（官方 nvidia.com/h200 核实，2026-08-07）

| 参数 | H200 SXM | H200 NVL |
|------|----------|----------|
| 架构 | Hopper (GH100，首款 HBM3e) | Hopper (GH100) |
| FP64 | 34 TFLOPS | 30 TFLOPS |
| FP64 Tensor Core | 67 TFLOPS | 60 TFLOPS |
| FP32 | 67 TFLOPS | 60 TFLOPS |
| TF32 Tensor Core | 989 TFLOPS（稀疏） | 835 TFLOPS（稀疏） |
| BF16/FP16 Tensor Core | 1,979 TFLOPS（稀疏） | 1,671 TFLOPS（稀疏） |
| FP8/INT8 Tensor Core | 3,958 TFLOPS/TOPS（稀疏） | 3,341 TFLOPS/TOPS（稀疏） |
| 显存 | 141GB HBM3e | 141GB HBM3e |
| 显存带宽 | 4.8 TB/s | 4.8 TB/s |
| Max TDP | 700W（可配置） | 600W（可配置） |
| 互连 | NVLink 900 GB/s / PCIe Gen5 128 GB/s | NVLink Bridge 900 GB/s/GPU（2/4路）/ PCIe Gen5 128 GB/s |
| MIG | 最多 7 个实例 @18GB | 最多 7 个实例 @16.5GB |
| 服务器选项 | HGX H200（4/8 GPU） | MGX H200 NVL（最多 8 GPU） |

> 注：以上为 NVIDIA 官网标注的"preliminary specifications, may be subject to change"（初步规格，可能变更）。

## B200 关键规格（⚠️ 待核实，暂缺官方逐项数据表）

截至本次核实（2026-08-07），官方 B200 独立产品页与 HGX B200 专页均未能成功抓取到结构化规格表（页面已被 Rubin/Blackwell Ultra 新一代内容覆盖，或需要 JS 渲染/PDF 才能看到完整数据）。**以下信息来自 Blackwell 架构页面的架构级描述，不是逐项产品规格表，请勿引用具体 TFLOPS/带宽数字用于正式文档**：

- Blackwell 架构采用双 die 设计，通过 10 TB/s 芯片间互连（chip-to-chip interconnect）构成统一单 GPU，单颗封装含 2080 亿晶体管，TSMC 4NP 定制工艺
- NVLink 为第五代，单个 NVL72 域内 GPU 带宽达 130 TB/s，支持扩展至 576 GPU
- 支持 FP4（NVFP4，微缩放格式）新精度，用于加速 LLM/MoE 推理与训练

**待办**：需要用户提供官方 B200 Datasheet PDF，或访问 NVIDIA 官方 Blackwell Resources Center 手动获取逐项规格（FP64/FP32/TF32/FP16/FP8/FP4 算力、显存容量与带宽、TDP、CUDA/Tensor Core 数量），再补全本表。**不要凭记忆或推算填入具体数字**。

---

## DGX H100/H200 系统架构（官方 User Guide 核实）

```
8× H100 (共640GB) / H200 (共1,128GB)
    ↕ NVLink 4.0 (全互连, via 4× NVSwitch, 900 GB/s GPU-GPU)
2× Intel Xeon 8480C (56核 PCIe Gen5)
    ↕ PCIe Gen5
系统内存: 2TB (32× DIMM)
OS存储: 2× 1.92TB NVMe M.2 (RAID 1)
数据缓存: 8× 3.84TB NVMe U.2 SED (RAID 0)
集群网络: 4× OSFP口, 8× ConnectX-7单口IB卡 (最高400Gbps/卡)
管理网络: 2× ConnectX-7双口以太网卡
电源: 6× 3.3kW (4+2冗余)
形态: 8U机架式, ~130kg
```

DGX OS 软件栈：Ubuntu + NVSM（健康监控）+ DCGM（GPU集群管理）+ GPU驱动 + Docker Engine + NVIDIA Container Toolkit + MOFED（InfiniBand驱动）。

BMC 支持 Redfish/IPMI/SNMP/KVM，独立于主系统电源状态，用于带外管理。

---

## Jetson 产品线对比（官方 Developer Guide r36.4 核实）

Jetson 分两类产品：
- **Developer Kit**：非量产规格模组 + 参考载板，仅用于开发调试，不可用于量产。
- **Module**：量产模组，单独销售，不预装软件，需配合自研载板刷写系统镜像。

| 系列 | 型号/P编号 | GPU 架构 | 备注 |
|------|-----------|----------|------|
| Jetson Orin Nano | 8GB(P3767-0003)/4GB(P3767-0004)商用；8GB带SD卡槽(P3767-0005)仅开发套件 | Ampere | 入门级 |
| Jetson Orin NX | 16GB(P3767-0000)/8GB(P3767-0001) | Ampere | 中端 |
| Jetson AGX Orin | 32GB(P3701-0004)/64GB(P3701-0005)商用；工业级(P3701-0008) | Ampere (2048 CUDA Cores) | 旗舰，另有仅开发套件版P3701-0000 |
| Jetson Xavier NX（上一代） | - | Volta | 边缘推理 |
| Jetson Nano（已停产） | - | Maxwell | 入门/教育 |

软件栈：JetPack SDK = Jetson Linux (L4T) + CUDA/cuDNN/TensorRT + 加速库 + 开发工具。刷写工具：SDK Manager（图形化）或 flash.sh 脚本。支持 A/B 双槽冗余（rootfs + bootloader）实现 OTA 失败回退。

---

## vGPU 软件版本与分支（官方文档核实，2026-07-21）

当前活跃分支：
| 分支 | 驱动分支 | 类型 | 最新版本 | EOL |
|------|---------|------|---------|-----|
| vGPU 20 | R595 | Production | 20.1 | 2027-03 |
| vGPU 19 | R580 | Long-Term Support | 19.5 | 2028-07 |
| vGPU 16 | R535 | Long-Term Support | 16.14 | 2026-07 |

选型原则：生产环境用最新 Long-Term Support Branch（3年支持周期）；测试环境追新特性用 Production Branch（1年支持周期）。

产品类型：vWS（虚拟工作站，含图形特性）、vCS（虚拟计算服务器，纯计算）、vPC/vApps（虚拟桌面/应用场景）。

---

## 待深入阅读

- [ ] H100/B200 Architecture Whitepaper
- [ ] MIG User Guide (Multi-Instance GPU)
- [ ] vGPU Software User Guide（GRID vGPU 详细配置）
- [ ] Omniverse Overview
