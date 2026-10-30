---
tags: [software, os, openwrt, spacemit, router, networking]
domain: software
aliases: [OpenWrt, 进迭OpenWrt]
status: approved
created: 2026-08-06
updated: 2026-10-29
source: C:\Users\luojunlin\docs-openwrt\zh\root_overview.md
superseded_by:
---

# OpenWrt

## 定义

OpenWrt 是基于 Linux 的嵌入式路由器与网络设备操作系统。SpacemiT 基于原生 OpenWrt 23.05 集成 Stone 系列芯片（K1）的 BSP，提供软路由和 NAS 方案支持，包含完整的启动栈（OpenSBI、U-Boot/UEFI）、Linux 内核、根文件系统及硬件加速组件。

## 为什么重要

- **软路由与 NAS**：专为路由器、网关、NAS 等网络设备优化
- **完整 BSP 集成**：包含 OpenSBI、U-Boot、Linux Kernel、硬件加速的 MPP/FFmpeg
- **包管理系统**：opkg 包管理器，可自由添加功能模块
- **硬件加速**：支持 MPP（Media Process Platform）、VPU、JPU 硬件编解码
- **资源高效**：内存与存储占用小，适合嵌入式设备

## 核心组件

**当前已集成组件：**
- **OpenSBI 1.3** — RISC-V 监管程序接口
- **U-Boot 2022.10** — 引导加载程序（支持 UEFI）
- **Linux Kernel 6.1/6.6** — 两个分支（bl-v1.0.y 基于 6.1，bl-v2.0.y 基于 6.6）
- **MPP (Media Process Platform)** — 媒体处理平台
- **FFmpeg（硬件加速）** — 支持 K1 硬件编解码

**正在适配中的组件：**
- onnxruntime（硬件加速）— AI 推理框架
- ai-support — AI 演示程序
- k1x-vpu-firmware — 视频处理单元固件
- k1x-vpu-test — VPU 测试程序
- k1x-jpu — JPEG 处理单元 API
- GStreamer（硬件加速）— 多媒体框架

## 版本与分支

| 分支 | Linux 内核 | OpenWrt 版本 | 维护状态 |
|------|-----------|-------------|---------|
| **bl-v2.0.y** | Linux 6.6 | 基于 master | 主要维护分支 |
| bl-v1.0.y | Linux 6.1 | OpenWrt 23.05 | 有限维护 |

**建议使用 bl-v2.0.y 分支**（后续主要维护分支）

## 支持设备

**当前支持的硬件：**
- BPI-F3
- [[../hardware/products/K1-MUSE-Pi|MUSE-Pi]]

**默认网络配置（软路由方案）：**
- **eth1** — LAN 口
- **eth2** — WAN 口
- **WiFi** — 默认开启 AP 模式

## 使用场景

- **软路由**：家庭/企业网关，多 WAN 负载均衡
- **NAS 存储**：网络附加存储服务器
- **防火墙**：网络安全边界设备
- **无线 AP**：WiFi 接入点与中继器
- **边缘计算**：网络功能虚拟化（NFV）边缘节点

## 获取与构建

**预编译固件下载：**
- 官方发布站点：https://archive.spacemit.com/openwrt/releases/
- BPI-F3/MUSE-Pi 固件：https://archive.spacemit.com/openwrt/releases/23.05.2/targets/spacemit/DEVICE_debX/

**源码仓库：**
```bash
git clone https://gitee.com/bianbu-linux/openwrt.git -b bl-v2.0.y
```

**编译环境要求：**
- 基于 Ubuntu 22.04
- 必需依赖：build-essential, clang, flex, bison, gcc-multilib, gettext, git, libncurses-dev, libssl-dev, python3-distutils, rsync, unzip, device-tree-compiler

**固件格式：**
- `*.zip` — Titan Flasher 刷机包（板载存储）
- `*sdcard.img` — SD 卡启动镜像（使用 dd 命令写入）

## 包管理

**SpacemiT 维护的软件源：**
```bash
# /etc/opkg/distfeeds.conf
src/gz openwrt_base https://archive.spacemit.com/openwrt/releases/bl-v2.0.y/packages/riscv64_riscv64/base
src/gz openwrt_luci https://archive.spacemit.com/openwrt/releases/bl-v2.0.y/packages/riscv64_riscv64/luci
src/gz openwrt_packages https://archive.spacemit.com/openwrt/releases/bl-v2.0.y/packages/riscv64_riscv64/packages
src/gz openwrt_spacemit_packages https://archive.spacemit.com/openwrt/releases/bl-v2.0.y/targets/spacemit/DEVICE_MUSE-N1/packages
```

## 设备自适应（EEPROM）

通过 EEPROM 存储设备信息（product_name、MAC Address、Serial Number 等），实现一个固件支持多个板型。

**支持的 EEPROM：**
- `atmel,24c02`

**写号命令（U-Boot shell）：**
```bash
tlv_eeprom set 0x21 k1-x_MUSE-Pi  # 设置 product_name
tlv_eeprom write                    # 写入 EEPROM
```

**支持的 TLV 字段：**
- 0x21 — Product Name
- 0x23 — Serial Number
- 0x24 — Base MAC Address
- 0x25 — Manufacture Date
- 0x26 — Device Version
- 0x2A — MAC Addresses
- 0x2B — Manufacturer
- 0x40 — SDK Version

## 在 SpacemiT 中

- **K1 平台专用**：当前仅适配 K1 系列芯片
- **完整硬件加速**：MPP、VPU、JPU 支持，FFmpeg/GStreamer 硬件加速
- **方案管理**：支持 k1-sbc、k1-nas 等多种方案，可自定义分区表、设备树、启动参数
- **UEFI 支持**：U-Boot 支持 UEFI 启动模式

## 相关词条

- [[Bianbu]] — SpacemiT 官方桌面 Linux 发行版（Ubuntu 基础）
- [[Buildroot]] — 另一种嵌入式系统构建方案
- [[../hardware/products/K1]] — K1 芯片（8×X60 核心，2 TOPS AI）
- [[../hardware/products/K1-MUSE-Pi]] — MUSE-Pi 开发板

## 来源文档

- SpacemiT OpenWrt 文档：C:\Users\luojunlin\docs-openwrt\zh\
- OpenWrt 官方：https://openwrt.org/
- SpacemiT 文档中心：https://www.spacemit.com/community/document?lang=zh
