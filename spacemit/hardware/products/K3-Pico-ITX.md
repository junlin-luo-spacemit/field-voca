---
tags: [product, hardware, k3, pico-itx, cluster, spacemit]
domain: hardware
aliases: [K3 Pico-ITX, K3 Pico ITX, K3 小尺寸主板]
status: approved
created: 2026-08-06
updated: 2026-10-29
source: C:\Users\luojunlin\docs-product\zh\k3_series\
superseded_by:
---

# K3 Pico-ITX

**产品形态**：Pico-ITX 小尺寸主板  
**芯片平台**：K3（8×X100 + 8×A100，60 TOPS AI）

## 定义

K3 Pico-ITX 是基于 K3 芯片的 Pico-ITX 标准小尺寸主板方案，适合体积受限的边缘设备与嵌入式场景。集成 60 TOPS AI 算力、10GbE 网络和 UFS2.2 存储，提供强大的计算能力和 I/O 性能。

## 为什么重要

- **高性能边缘计算**：60 TOPS AI 算力，支持 30B 参数大模型
- **紧凑尺寸**：Pico-ITX 标准（100×72mm），适合空间受限场景
- **10GbE 网络**：高速数据传输，适合网络密集型应用
- **UFS2.2 存储**：高性能闪存存储，低延迟
- **丰富扩展**：PCIe、USB 3.0、多显示输出

## 核心规格

### 处理器
- **芯片**：K3（8×X100 CPU + 8×A100 AI 核心）
- **CPU**：8×X100 高性能核心 @ 2.4GHz
- **AI 算力**：约 **60 TOPS (Int8)**
- **架构**：RISC-V RVA23 profile

### 内存
- **类型**：LPDDR5-6400
- **容量**：待确认（通常 8GB/16GB 可选）

### 存储
- **板载存储**：UFS2.2（高性能闪存）
- **容量**：待确认（通常 64GB/128GB 可选）
- **扩展**：支持 PCIe 存储扩展

### 网络
- **以太网**：**10GbE**（10 Gigabit Ethernet）
- **无线**：待确认（可能支持 Wi-Fi 6/6E）

### 显示输出
- **HDMI**：支持（数量待确认）
- **DP**：可能支持 DisplayPort
- **分辨率**：支持 4K@60Hz（待确认）

### 扩展接口
- **PCIe**：PCIe 3.0/4.0（用于扩展卡、存储、网络）
- **USB**：USB 3.0/3.1
- **GPIO**：通用 I/O 接口
- **其他**：UART、I2C、SPI 等

### 电源
- **供电**：待确认（通常 12V DC）
- **功耗**：待确认（预计 TDP 30-50W）

## 主板尺寸

- **标准**：Pico-ITX
- **尺寸**：100mm × 72mm（标准 Pico-ITX 规格）
- **安装**：标准 Pico-ITX 螺丝孔位

## 支持的操作系统

- **Bianbu** — Ubuntu 26.04 基础桌面/服务器系统
- **OpenWrt** — 网络设备/软路由系统（K3 支持开发中）
- **Buildroot** — 嵌入式 Linux 最小化系统
- **OpenHarmony** — 鸿蒙系统（K3 OH6.1 开发中）

## 使用场景

### 边缘 AI 推理
- 本地大语言模型推理（30B 参数）
- 计算机视觉处理（目标检测、识别）
- 语音识别/合成（ASR/TTS）
- 实时视频分析

### 网络设备
- 高性能软路由（10GbE）
- 网络功能虚拟化（NFV）
- 边缘网关
- SD-WAN 设备

### 工业嵌入式
- 工业控制系统
- 边缘数据中心节点
- 机器视觉检测设备
- 高性能数据采集

### 边缘服务器
- 边缘计算节点
- 内容分发节点（CDN）
- 本地缓存服务器
- 视频转码服务器

## 与其他 K3 产品对比

| 特性 | K3 Pico-ITX | K3-CoM260-Kit | K3-Shelf |
|-----|------------|--------------|----------|
| **形态** | Pico-ITX 主板 | CoM 核心板套件 | 服务器主板 |
| **尺寸** | 100×72mm | CoM 模块 + 底板 | ATX/服务器尺寸 |
| **网络** | 10GbE | 待确认 | 多路 10GbE |
| **存储** | UFS2.2 | 待确认 | NVMe/SATA |
| **扩展** | 中等 | 高（底板定制） | 最高 |
| **适用场景** | 嵌入式、边缘网关 | 定制化产品 | 数据中心、集群 |

## 与 RV2768 对比

| 特性 | K3 Pico-ITX | RV2768 |
|-----|------------|--------|
| **核心数** | 16 核（8 CPU + 8 AI） | 768 核（96×K1 芯片） |
| **AI 算力** | 60 TOPS | 2880 TOPS |
| **形态** | 单板主板 | 集群服务器 |
| **功耗** | ~30-50W | 数千瓦 |
| **适用场景** | 边缘设备 | 数据中心 AI 集群 |

## 技术优势

### 高性能 AI
- 60 TOPS AI 算力（K3 A100 专用核心）
- 支持 30B 参数大模型本地推理
- 硬件加速的 ONNXRuntime、Llama.cpp

### 紧凑设计
- Pico-ITX 标准尺寸（100×72mm）
- 适合空间受限的工业机柜
- 标准螺丝孔位，易于安装

### 高速 I/O
- 10GbE 网络（10 Gbps 以太网）
- UFS2.2 高性能存储
- PCIe 扩展支持

### 工业级可靠性
- 宽温工作范围（待确认）
- 无风扇被动散热设计（可选）
- 长生命周期支持

## 开发资源

- **Bianbu OS**：完整 Ubuntu 桌面/服务器系统
- **AI SDK**：ONNXRuntime、Llama.cpp、vLLM
- **开发工具**：SpacemiT Studio（Web IDE）、TitanTools（刷机）、交叉编译工具链
- **文档**：完整 BSP 文档、外设驱动开发指南

## 获取与支持

- **产品咨询**：访问 SpacemiT 官网
- **文档中心**：https://www.spacemit.com/community/document
- **开发工具**：SpacemiT Studio (https://studio.spacemit.com/)

## 相关词条

### 芯片与核心
- [[K3]] — K3 芯片（8×X100 + 8×A100，60 TOPS）
- [[../atoms/X100-Core]] — K3 高性能 CPU 核心
- [[../atoms/A100-AI-Core]] — K3 专用 AI 加速核心

### 其他 K3 产品
- [[K3-CoM260-Kit]] — K3 核心板套件
- [[K3-Shelf]] — K3 服务器主板
- [[RV2768]] — 768 核 RISC-V 集群服务器

### 软件系统
- [[../../software/Bianbu]] — Ubuntu 基础系统
- [[../../software/OpenWrt]] — 网络设备系统
- [[../../software/Buildroot]] — 嵌入式系统

### 开发工具
- [[../../tools/SpacemiT-Studio]] — Web 开发工作台
- [[../../tools/Flashing-Tool]] — TitanTools 刷机工具
- [[../../tools/Cross-Compilation-Toolchain]] — 交叉编译工具链

## 来源文档

- SpacemiT 产品文档：C:\Users\luojunlin\docs-product\zh\k3_series\
- SpacemiT 官网：https://www.spacemit.com/
- 文档中心：https://www.spacemit.com/community/document?lang=zh
