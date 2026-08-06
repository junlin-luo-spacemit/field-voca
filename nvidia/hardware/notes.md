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
