---
tags: [product, hardware, k1, muse, card, sbc, spacemit]
domain: hardware
aliases: [K1 MUSE Card, MUSE Card, K1单板卡]
status: approved
created: 2026-08-06
updated: 2026-10-29
source: C:\Users\luojunlin\docs-product\zh\k1_series\k1-x_muse_card\
superseded_by:
---

# K1 MUSE Card

**中文译名**：K1 MUSE Card 单板卡

## 定义

K1 MUSE Card 搭载进迭时空八核 RISC-V AI CPU，提供更小巧更高效的 SBC 产品体验。

## 产品特点

- 基于进迭时空 RISC-V 八核高性能中央处理器
- CPU 融合 2 TOPS@INT8 AI 算力
- 支持 32bit LPDDR4X
- 支持双显，MIPI DSI & HDMI 接口，FHD 高清显示
- 支持 4K 编、解码
- USB3.0 × 1，USB2.0 × 1，40Pin 标准 GPIO 插座
- 支持双 M.2 2242 SSD
- 1000M RJ45 网口
- Type-C 接口供电

## 关键参数

### 处理器与算力
- **处理器**：SpacemiT M1
- **架构**：8 核 64 位 RISC-V 处理器
- **AI 算力**：融合 2 TOPS AI 算力

### 显示输出
- **HDMI**：Type-A 接口，最高支持 1080P@60Hz
- **MIPI DSI**：4lane FPC 接口，最高支持 1080P@60Hz

### 内存与存储
- **内存**：LPDDR4X，2400MT/s 速率，可选配 8GB/16GB 容量
- **本地存储**：SPI NOR FLASH，64Mb 容量
- **扩展存储**：
  - 2 个 M.2 2242 M-Key 连接器，可装配 2 个 NVMe SSD
  - TF 卡接口，支持 UHS-II 模式存储卡

### 网络与通讯
- **有线网络**：支持 1 路以太网，RJ45 接口，1000M/100M 自适应

### 接口
- **音频接口**：3.5mm 音频耳麦接口
- **USB 接口**：
  - 1 路 USB3.0 Type-A host 接口
  - 1 路 USB2.0 Type-A host 接口
  - 1 路 USB2.0 Type-C device 接口，支持 12V 3A 供电
- **调试接口**：40Pin 标准 GPIO 接口，附带 3 个侧边按键，用于硬件复位、开关机和烧录升级
- **MIPI 接口**：
  - 1 路 4lane MIPI DSI
  - 2 路 4lane MIPI CSI，支持"4+4"或"4+2+2"组合链路

### 物理特性
- **外观形态**：单板计算机，可选装透明亚克力外壳，尺寸为 85 × 56mm
- **操作系统**：支持 Bianbu Desktop、Bianbu NAS、Ubuntu、OpenKylin、Deepin、Fedora 等操作系统
- **电源输入**：支持 USB PD3.0 供电
- **可靠性**：
  - 外设接口 ESD 可防护接触 ±4kV，空气 ±8kV
  - 可选消费级 -20°C ~ 70°C 或工业级 -40°C ~ 85°C

## 应用场景

- 紧凑型嵌入式应用
- 边缘 AI 计算
- 教育与原型开发
- 小型服务器应用

## 相关词条

- [[K1]] — K1 芯片详细规格
- [[../atoms/X60-Core]] — X60 核心架构
- [[../software/Bianbu]] — 支持的操作系统
- [[K1-MUSE-Pi]] — 标准尺寸版本

## 来源文档

- [K1 MUSE Card 产品页面](https://www.spacemit.com/community/document)
- 商务咨询：0571-89000775 / business@spacemit.com
