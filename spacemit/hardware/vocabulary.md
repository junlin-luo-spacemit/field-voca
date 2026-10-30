---
tags: [spacemit, hardware, vocabulary]
domain: hardware
date: 2026-08-08
source: C:\Users\luojunlin\docs-chip\, C:\Users\luojunlin\docs-product\
---

# 硬件平台词汇表

> 涵盖：K/P 系列芯片、生态硬件、计算架构
> 来源：SpacemiT 文档中心 Hardware 分类

---

## 芯片产品（Chip Products）

### K Series / K 系列
SpacemiT 的 AI CPU 芯片系列，针对高性能 AI 计算场景。
- [[products/K1]] — 入门级 AI CPU
- [[products/K3]] — 高性能 AI CPU，8×X100 + 8×A100 AI 核心，60 TOPS

### P Series / P 系列
SpacemiT 的电源管理芯片系列。
- [[products/P1]] — 电源管理芯片
- [[products/P1S]] — P1 的变体型号

---

## 核心概念（Core Concepts）

### SoC (System on Chip) / 系统级芯片
将 CPU、AI 加速单元、GPU、内存控制器、I/O 等集成在单一芯片上。详见 [[../atoms/SoC]]。

### Matrix Extension / 矩阵扩展指令集
SpacemiT 为 RISC-V 架构添加的矩阵运算指令扩展，用于加速 AI 计算。详见 [[../atoms/Matrix-Extension]]。

### X60 Core / X60 核心
SpacemiT 处理器中使用的 CPU 核心架构。详见 [[../atoms/X60-Core]]。

---

## 计算架构（Compute Architecture）

### Compute Architecture / 计算架构
SpacemiT 芯片的整体计算单元组织方式，包括 CPU、AI 核心、内存子系统的布局。详见 [[Compute-Architecture]]。

### Design Philosophy / 设计哲学
SpacemiT 产品的设计原则与技术决策，如功耗平衡、可扩展性、开放生态。详见 [[Design-Philosophy]]。

---

## 内存（Memory）

### HBM (High Bandwidth Memory) / 高带宽内存
高性能内存技术，提供极高的数据传输带宽。详见 [[../atoms/HBM]]。

### DRAM / 动态随机存取内存
标准系统内存，K 系列芯片支持 LPDDR4/5。

### Memory Controller / 内存控制器
管理 CPU/AI 核心与内存之间数据传输的硬件单元。

---

## 电源管理（Power Management）

### PMIC (Power Management IC) / 电源管理集成电路
控制芯片各部分电源供应、电压调节的专用芯片。详见 [[../atoms/PMIC]]。

### TDP (Thermal Design Power) / 热设计功耗
芯片在持续满负载时的最大散热需求，影响散热方案设计。

### Power Domain / 电源域
可独立供电/断电的芯片功能区域，用于动态功耗管理。

---

## I/O 与互连（I/O & Interconnect）

### IOMMU (I/O Memory Management Unit) / I/O 内存管理单元
为外设提供内存地址转换与隔离保护。详见 [[../atoms/IOMMU]]。

### PCIe (Peripheral Component Interconnect Express)
高速串行总线标准，用于连接 GPU、SSD、网卡等高速设备。

### Ethernet / 以太网
网络连接接口，K3 支持双千兆以太网 (RGMII/SGMII)。

### M.2
固态硬盘和扩展卡接口标准，支持 NVMe 协议。

### USB
通用串行总线，K 系列支持 USB 3.0/3.1。

---

## 生态硬件（Ecosystem Hardware）

### Pico-ITX / Pico-ITX 主板
小型主板标准（100mm × 72mm），适合嵌入式系统。
- [[products/K3-Pico-ITX]]

### CoM (Computer on Module) / 计算模块
可插拔的计算模块，集成 CPU、内存、存储，简化产品开发。
- [[products/K3-CoM260-Kit]]

### Cluster Server / 集群服务器
多节点服务器机箱，用于构建 AI 计算集群。
- [[products/RV2768]] — RISC-V 集群服务器，支持多个计算节点
- [[products/N10-N48-Cluster]] — N10/N48 集群配置

### MUSE Series / MUSE 系列
面向开发者、教育和消费市场的产品线。
- [[products/K1-MUSE-Pi]] — 单板计算机
- [[products/K1-MUSE-Pi-Pro]] — 增强版单板计算机
- [[products/K1-MUSE-BOOK]] — 笔记本形态
- [[products/K1-MUSE-PAPER]] — 电子纸设备
- [[products/K1-MUSE-BOX]] — 盒子形态
- [[products/K1-MUSE-Card]] — 扩展卡形态
- [[products/K1-MUSE-Shelf]] — 机架式服务器
- [[products/K1-RISC-V-Lab-Kit]] — RISC-V 实验套件

---

## 调试与跟踪（Debug & Trace）

### JTAG
标准调试接口，用于芯片级调试与固件烧录。

### Trace / 执行跟踪
记录 CPU 指令执行流程的调试技术。

---

## 参考资料

- [SpacemiT 硬件文档](https://www.spacemit.com/community/document?lang=zh)
- K1 Datasheet, User Manual, Hardware Design Guide
- K3 Datasheet, User Manual, Hardware Design Guide
