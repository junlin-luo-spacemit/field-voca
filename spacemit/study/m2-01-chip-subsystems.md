---
tags: [study, m2, chip, architecture, spacemit]
domain: chip
aliases: [芯片子系统, Chip Subsystems]
status: needs_review
created: 2026-11-19
updated: 2026-11-19
---

# 一颗芯片里有什么

> **本讲目标**：建立芯片的硬件骨架，知道一颗 [SoC](GLOSSARY.md#soc) 里装了哪些功能模块，它们各自负责什么。
> 
> **📌 说明**：本模块以 **K3** 为例讲解芯片架构，但子系统划分、规格解读、数据流等概念适用于所有 SoC 芯片。

## 一、一颗 SoC 不是一个 CPU

**[SoC](GLOSSARY.md#soc)**（System on Chip，系统级芯片）不是一个纯粹的处理器，而是一个集成了多种功能的完整系统。

**关键理解**：CPU 是 SoC 的一个组成部分。
- **SoC** = 完整的系统级芯片
- **CPU 子系统** = SoC 的一个子系统（K3 有 11 个子系统，CPU 子系统是其中之一）
- **CPU 核心**（X100/A100）= CPU 子系统的计算核心

类比关系：
```
SoC（整栋大楼）
  ├─ CPU 子系统（一层楼）
  │   ├─ X100 核心（办公室）
  │   └─ A100 核心（办公室）
  ├─ 内存子系统（另一层楼）
  ├─ 图像子系统（另一层楼）
  └─ ... 其他 8 个子系统
```

K3 芯片包含 11 个子系统（Subsystem）：

| 子系统 | 英文 | 负责什么 |
|---|---|---|
| CPU 子系统 | CPU Subsystem | 通用计算核与 AI 核 |
| [内存与存储](GLOSSARY.md#内存与存储) | Memory & Storage | [LPDDR5](GLOSSARY.md#lpddr5)、[eMMC](GLOSSARY.md#emmc)、SD 卡 |
| 图像子系统 | Image Subsystem | 摄像头输入、[ISP](GLOSSARY.md#isp) 处理 |
| 视频子系统 | Video Subsystem | 编码与解码（H.264/H.265/AV1） |
| 显示子系统 | Display Subsystem | 屏幕输出（MIPI-DSI、DP/eDP） |
| 音频子系统 | Audio Subsystem | 音频输入输出、编解码 |
| 互联子系统 | Connectivity Subsystem | PCIe（Peripheral Component Interconnect Express，高速串行互联）、USB、以太网 |
| 安全子系统 | Security Subsystem | 安全启动、TEE（Trusted Execution Environment，可信执行环境）、密钥管理 |
| 系统外设 | System Peripherals | GPIO、I2C、SPI、UART 等 |
| 时钟与复位 | Clock & Reset | 时钟树、复位控制 |
| 启动模式 | Boot Mode | 启动介质选择 |

规格参数章节按这个顺序逐个讲解每个子系统的能力。

## 二、芯片规格 vs 产品规格

**芯片规格**是芯片本身支持的最大能力。**产品规格**是某个具体产品实际引出的接口。

举例（K3 CoM260）：
- 芯片支持 4 个 CSI 摄像头接口，但 CoM260 载板只引出 2 个。
- 芯片支持 2 个 DP/eDP 显示输出，但 CoM260 只引出 1 个 DP。

**为什么不全引出？**
- 目标场景用不到 4 路摄像头，引出更多会增加面积、连接器和成本。
- 客户可以定制载板，按需选择要引出的接口。

**规律**：产品资料记录实际引出的接口，芯片手册记录芯片本身的能力上限。

## 三、计算核心：通用核与 AI 核

K3 有两种核心：

| 核心类型 | 型号 | 数量 | 架构 | 干什么 |
|---|---|---|---|---|
| 通用核 | X100 | 8 个 | RISC-V RVA23 + RVV 1.0 + RVH 1.0 | 运行操作系统、应用程序、通用计算 |
| AI 核 | A100 | 8 个 | RISC-V + IME 扩展 | 神经网络推理、AI 加速 |

**同构融合计算**：X100 和 A100 都基于 RISC-V 指令集（同构），在同一芯片上紧密协作（融合）。
- **同构**：统一的基础指令集，开发者不需要学习两套架构。
- **融合**：共享内存、统一调度，数据交换直接高效。
- **分工**：X100 运行操作系统和调度任务，A100 执行 AI 密集计算。

**类比**：就像一个团队里，所有人说同一种语言（RISC-V），但有些人接受过 AI 专业训练（IME），沟通成本低、协作效率高。这比 x86 CPU + NVIDIA GPU 的异构方案更简单。

**关键术语**：
- **[RVA23](GLOSSARY.md#rva23)**：RISC-V 应用处理器标准 2023 版
- **[RVV 1.0](GLOSSARY.md#rvv-10)**：向量扩展，VLEN=256 位，支持 SIMD 计算
- **[RVH 1.0](GLOSSARY.md#rvh-10)**：虚拟化扩展，GEILEN=8，支持 8 个[虚拟机](GLOSSARY.md#虚拟机)
- **[IME](GLOSSARY.md#ime)**：Intelligent Matrix Engine，智能矩阵引擎，专门优化矩阵运算和卷积

## 四、数据流：从输入到输出

一条完整的数据流经过多个子系统：

```
摄像头 → 图像子系统（ISP）→ 内存（LPDDR5）→ 视频子系统（编码）→ 互联子系统（网络/存储）
```

另一条流：

```
内存（模型）→ CPU 子系统的 A100 核（推理）→ 内存（结果）→ 显示子系统（屏幕输出）
```

**关键点**：
- 内存是数据中转站，所有子系统都要通过内存交换数据。
- [内存带宽](GLOSSARY.md#带宽)决定了数据流的效率上限。
- 子系统之间不直接连接，都通过总线（Bus）和内存协作。

## 快速自测

1. SoC 与单纯的 CPU 有什么区别？
2. 为什么产品只引出芯片支持的一部分接口？
3. X100 和 A100 的分工是什么？
4. 为什么说内存带宽决定数据流效率上限？

## 参考来源

| 来源 | 章节 | 内容 |
|---|---|---|
| docs-chip · key_stone/k3/k3_docs/k3_ds.md | 2. 规格参数 | 11 个子系统的完整列表 |
| docs-chip · key_stone/k3/k3_docs/k3_ds.md | 1.3 架构框图 | 子系统之间的连接关系 |
| docs-chip · key_stone/k3/k3_docs/k3_ds.md | 2.1 CPU 子系统 | X100 与 A100 的规格 |
| docs-product · k3_series/k3_com260/root_overview.md | 产品规格 | CoM260 实际引出的接口 |
