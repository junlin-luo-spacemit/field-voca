---
tags: [software, os, bianbu, spacemit]
domain: software
aliases: [Bianbu, 边部系统, Bianbu OS]
status: approved
created: 2026-08-06
updated: 2026-10-29
source: C:\Users\luojunlin\docs-bianbu\zh\
superseded_by:
---

# Bianbu

**中文译名**：边部系统

## 定义

Bianbu 是为 RISC-V 架构的处理器深度优化的操作系统，基于 Ubuntu 社区源码构建，为进迭时空 AI CPU 提供系统底座。

## 为什么重要

- **官方适配**：针对 SpacemiT K1/K3 芯片深度优化
- **完整生态**：基于 Ubuntu，可使用 apt 安装数万个软件包
- **开箱即用**：预装 BSP、驱动、工具链，免除环境配置
- **多种版本**：提供桌面环境与命令行版本，满足不同场景需求

## 系统版本

Bianbu 为开发者和用户提供以下版本镜像：

1. **GNOME 桌面版本**
   - 原生桌面版，预装 GNOME Shell 桌面环境
   - 预装 Chromium、LibreOffice、MPV 等应用
   
2. **LXQt 桌面版本**
   - 基于 LXQt 重新设计和开发的轻桌面
   - 用于轻量级，对资源占用和性能有要求的场景
   
3. **命令行版本**
   - 最小系统版本，提供命令行界面

## 版本历史

| 版本 | 基于 Ubuntu | 支持芯片 | 状态 |
|------|------------|---------|------|
| **Bianbu 4.x** | Ubuntu 26.04 | **K3** | ✅ 活跃（K3 推荐版本）|
| **Bianbu 2.x** | Ubuntu 23.10 | **K1** | ✅ 长期维护版本（K1 推荐）|
| Bianbu 3.x | — | K1 | 🔄 过渡版本 |
| Bianbu 1.x | — | K1 | ❌ 已停止维护 |

### Bianbu 4.0（K3）

- **最新版本**：V4.0.7（2026-09-16）
- **基于**：Ubuntu 26.04
- **主要特性**：
  - LXQt 桌面环境优化
  - UEFI 固件支持
  - 休眠/唤醒稳定性改进
  - 多屏显示支持

### Bianbu 2.3（K1）

- **状态**：长期维护版本
- **基于**：Ubuntu 23.10
- **主要特性**：
  - GNOME/LXQt 双桌面选择
  - 完整硬件驱动支持
  - ROS 2 Humble/Jazzy 适配

## 系统架构

Bianbu 采用分层架构：

```
┌─────────────────────────────────────────┐
│           应用层（Applications）          │
│  GNOME/LXQt、Chromium、LibreOffice、VSCode│
├─────────────────────────────────────────┤
│          框架层（Frameworks）             │
│  Electron、GTK、Qt、FFmpeg、GStreamer    │
│  onnxruntime（硬件加速）                  │
├─────────────────────────────────────────┤
│          运行时（Runtime）                │
│  Python、Java、Node.js、Rust            │
├─────────────────────────────────────────┤
│             库（Libraries）               │
│  OpenCV（RVV优化）、OpenSSL、MPP、Mesa   │
│  OpenGLES、Vulkan、OpenCL                │
├─────────────────────────────────────────┤
│          Linux 内核（Kernel）             │
│  6.6 LTS / 6.18（K3）                    │
├─────────────────────────────────────────┤
│      引导加载程序（Bootloader）            │
│  U-Boot 2022.10                          │
├─────────────────────────────────────────┤
│      固件（Firmware）                     │
│  OpenSBI 1.3/1.6                         │
└─────────────────────────────────────────┘
```

## 软件组件

### 应用

- GNOME/LXQt 桌面及其常用应用
- 远程桌面
- Chromium 浏览器
- LibreOffice 办公套件
- Visual Studio Code
- Docker

### 框架

**应用框架**：
- Electron
- GTK
- Qt

**多媒体框架**：
- FFmpeg（支持硬件加速）
- GStreamer（支持硬件加速）
- PipeWire

**推理框架**：
- onnxruntime（支持硬件加速）

### 运行时

- Python
- Java
- Node.js
- Rust

### 库

- **OpenCV**（支持 RVV 加速）
- **OpenSSL**（支持硬件加速）
- **MPP**（进迭时空多媒体处理平台，提供 C API 和示例）
- Mesa 3D
- OpenGLES/Vulkan/OpenCL

### Linux 内核

支持版本与源码链接：
- **6.6**：长期支持（LTS）— [gitee](https://gitee.com/spacemit-buildroot/linux-6.6) / [github](https://github.com/spacemit-com/linux-6.6)
- **6.18**：K3 最新版本 — [gitee](https://gitee.com/spacemit-buildroot/linux-6.18) / [github](https://github.com/spacemit-com/linux-6.18)
- 6.1：已停止维护（EOL）

### U-Boot

- **版本**：u-boot-2022.10
- **源码**：[gitee](https://gitee.com/spacemit-buildroot/uboot-2022.10) / [github](https://github.com/spacemit-com/uboot-2022.10)

### OpenSBI

- **版本**：1.3 / 1.6
- **源码**：
  - 1.3: [gitee](https://gitee.com/spacemit-buildroot/opensbi) / [github](https://github.com/spacemit-com/opensbi)
  - 1.6 (K3): [gitee](https://gitee.com/spacemit-buildroot/opensbi/tree/k3-br-v1.0.y/) / [github](https://github.com/spacemit-com/opensbi/tree/k3-br-v1.0.y)

## 支持的设备

### K1 系列
- BPI-F3
- Milk-V Jupiter
- MUSE Card
- MUSE Pi
- MUSE Pi Pro
- MUSE Box
- MUSE Book

### K3 系列
- Pico ITX
- CoM260 Kit

## 与其他 OS 对比

| 操作系统 | 特点 | 适用场景 |
|---------|------|---------|
| **Bianbu** | Ubuntu 基础，完整桌面，官方优化 | 开发、通用计算、智能终端 |
| Buildroot | 最小化定制，构建工具 | 嵌入式、资源受限设备 |
| OpenWrt | 路由器固件 | 网络设备、网关 |
| OpenHarmony | 多终端物联网 | IoT、鸿蒙生态设备 |

## 使用场景

- 单板机与开发套件（MUSE Pi / MUSE Book 等）
- 开发与测试环境
- 桌面与服务器应用
- 教学与演示
- ROS 2 机器人开发

## 获取方式

- **官方镜像下载**：[https://archive.spacemit.com/image/k1/version/bianbu/](https://archive.spacemit.com/image/k1/version/bianbu/)
- **文档中心**：Software / SDK / Bianbu
- **问题反馈**：[https://gitee.com/bianbu/bianbu-docs/issues](https://gitee.com/bianbu/bianbu-docs/issues)

## 相关词条

- [[SDK]] — 软件开发套件
- [[../hardware/products/K1]] — K1 支持 Bianbu 2.x
- [[../hardware/products/K3]] — K3 支持 Bianbu 4.x
- [[Buildroot]] — 另一种系统构建方案
- [[ROS2]] — 机器人操作系统，Bianbu 原生支持

## 来源文档

- SpacemiT 文档中心：Software / Bianbu
- Bianbu 发布说明：[Bianbu 4.0](https://www.spacemit.com/community/document?lang=zh&nodepath=software/SDK/bianbu/release_notes/bianbu_4.0.md)
- Bianbu 源码仓库：[Gitee](https://gitee.com/bianbu) / [GitHub](https://github.com/spacemit-com)
