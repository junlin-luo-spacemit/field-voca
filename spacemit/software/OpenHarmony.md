---
tags: [software, os, openharmony, spacemit, riscv, harmony]
domain: software
aliases: [OpenHarmony, 开鸿系统, 开放鸿蒙, 鸿蒙]
status: approved
created: 2026-08-06
updated: 2026-10-29
source: C:\Users\luojunlin\docs-openharmony\zh\
superseded_by:
---

# OpenHarmony

**中文译名**：开鸿系统 / 开放鸿蒙

## 定义

OpenHarmony 是开源操作系统，面向多终端设备与物联网场景。SpacemiT 推出**全球首个 RISC-V + OpenHarmony 5.0 原生鸿蒙解决方案**（K1 OH5.0），采用 K1 新 RISC-V AI CPU 芯片，集成 OpenHarmony 5.0.0.71，基于 RISC-V 同构融合 AI 算力和开源大模型，构建新 AI 算力平台。方案已经过完整系统集成测试和可靠性测试，提供全套开发文档、量产工具和原厂支持通道。

## 为什么重要

- **RISC-V 原生鸿蒙**：全球首个 RISC-V + OpenHarmony 5.0 量产方案
- **多终端统一**：统一 OS 架构，覆盖平板、开发板、笔记本等
- **分布式能力**：设备间协同、资源共享
- **AI 算力融合**：RISC-V 同构融合 AI 算力，支持 Ollama + DeepSeek、YOLO、TTS/ASR、VLM
- **国产生态**：OpenAtom 基金会托管，国内开发者与厂商支持度高
- **量产就绪**：完整工厂测试应用、刷机/写号工具、完整文档支持

## 支持版本与平台

| 平台 | OpenHarmony 版本 | 支持情况 | 备注 |
|------|-----------------|---------|------|
| **K1** | **OpenHarmony 5.0** | **可量产** | 全球首发，完整测试 |
| K1 | OpenHarmony 6.1 | 开发中 | |
| K3 | OpenHarmony 6.1 | 开发中 | |

## 支持设备

| 生态产品 | MUSE Paper2 | MUSE Card | MUSE Pi | MUSE Pi Pro | MUSE Book |
|---------|------------|-----------|---------|-------------|-----------|
| **产品形态** | 平板电脑 | 开发板 | 开发板 | 开发板 | 笔记本电脑 |
| **启动方式** | eMMC / TF卡 | TF卡 | eMMC | eMMC | NOR + SSD |
| **显示输出** | MIPI DSI | MIPI DSI / HDMI | MIPI DSI / HDMI | MIPI DSI | MIPI DSI 转 eDP |
| **屏幕触摸** | ✓ | ✓ | ✓ | ✓ | × |
| **Wi-Fi** | ✓ | ✓ | ✓ | ✓ | ✓ |
| **蓝牙** | ✓ | × | × | × | × |
| **以太网** | ✓（USB转） | ✓ | ✓ | ✓ | × |
| **摄像头** | ✓（支持双摄） | × | × | × | × |
| **双屏显示** | × | × | × | × | ✓（接HDMI） |

**固件下载地址**：https://archive.spacemit.com/image/k1/version/openharmony/k1_oh5.0/

## K1 OH5.0 版本历史

| 版本 | 发布时间 | 主要特性 |
|------|---------|---------|
| OH5.0 V1.0RC1 | 2024-12-25 | Camera 480P 预览/拍照/录像、浏览器、生态产品全家桶、蓝牙基础功能 |
| OH5.0 V1.0RC2 | 2025-05-31 | Camera 1080P30fps、4K60fps H.264播放、AI标准框架、MUSE Pi Pro、USB存储/摄像头、双摄、4G通信模组 |
| OH5.0 V1.0RC2.1 | 2025-06-12 | 双屏显示（MUSE Book）、应用商城、ASR/TTS演示demo |
| OH5.0 V1.0RC2.2 | 2025-06-20 | Python 3.10、5G模组上网、Miracast sink端 |
| OH5.0 V1.0RC2.3 | 2025-07-25 | Miracast source端、卡刷机、A/B分区OTA升级 |
| **OH5.0 V1.0RC2.4** | **2025-08-31** | 震动马达、工程测试应用、指南针、闪光灯、GNSS定位 |
| **OH5.0 V1.0RC2.5** | **2025-09-26** | **AI框架、光距感传感器、霍尔传感器** |

## AI 应用支持

SpacemiT K1 OH5.0 提供完整的 AI 应用框架和演示应用（7 个 AI demo）：

1. **AI 检测识别应用（YOLO）** — 目标检测与识别
2. **AI 聊天应用（LLM Chat）** — 基于 Ollama + DeepSeek-R1-Distill-Qwen-1.5B-Q4_0 的本地大模型聊天
3. **RVV 应用** — RISC-V Vector 扩展优化演示
4. **TTS 应用（文本转语音）** — 语音合成
5. **ASR 应用（语音识别）** — 语音转文本
6. **OpenCV 应用** — 计算机视觉处理
7. **VLM 应用（视觉问答）** — 视觉语言模型应用

**Ollama + DeepSeek 集成**：
- 模型文件：`deepseek-r1-distill-qwen-1.5b-q4_0.gguf`（GGUF 格式，量化压缩）
- 运行环境：Ollama（模型管理与推理框架）
- 一键安装包：提供 Windows 自动安装脚本（`setup_ohos_ollama_env_v1.0.bat`）

## 开发资源

**完整开发包清单：**

**软件文档（10 份）：**
1. K1 OH5.0 下载编译烧录说明
2. K1 OH5.0 方案添加移植说明
3. K1 OH5.0 系统定制说明
4. K1 OH5.0 系统调试说明
5. K1 OH5.0 应用开发说明
6. K1 OH5.0 AI 构建开发说明
7. K1 OH5.0 驱动开发说明
8. K1 OH5.0 方案 OTA 使用说明
9. K1 OH5.0 三方包移植说明
10. K1 OH5.0 工程测试应用使用说明

**硬件文档：**
- MUSE Paper 方案框图、原理图、PCB、用户指南
- 支持全系 MUSE 产品（Paper、Card、Pi、Pi Pro、Book）

**开发工具：**
- **TitanFlasher** — 刷机/写号工具
- **DevEco Studio 5.0.5.310** — 应用开发 IDE
- **SDK 包** — OpenHarmony SDK 12（下载地址：https://archive.spacemit.com/tools/openharmony/sdk/12/）

## 构建环境要求

**推荐 Ubuntu 22.04**（其他版本未验证）

**依赖安装：**
```bash
sudo apt-get update
sudo apt-get install binutils git git-lfs gnupg gperf build-essential zip curl \
  zlib1g-dev gcc-multilib lib32ncurses5-dev x11proto-core-dev libx11-dev \
  lib32z1-dev libxml2-utils xsltproc unzip m4 bc gnutls-bin python3-pip ruby \
  ccache libgl1-mesa-dev g++-multilib libc6-dev-i386 flex bison openjdk-19-jdk \
  openssl libelf-dev lz4 genext2fs libncurses5 libssl-dev u-boot-tools dosfstools \
  mtools libstdc++-12-dev repo python2 cpio nodejs npm default-jdk mtd-utils \
  scons gcc-arm-none-eabi
```

**额外工具：**
- `make_ext4fs` — 文件系统工具（下载：https://archive.spacemit.com/tools/openharmony/bin.zip）

## 源码下载与编译

**代码仓库**：Gitee（https://gitee.com/riscv-sig）

**下载源码：**
```bash
mkdir oh5 && cd oh5
repo init -u git@gitee.com:riscv-sig/manifest.git -b riscv/OpenHarmony-v5.0.0-Release --no-repo-verify
repo sync -j4 -c --fail-fast
repo forall -c 'git lfs pull'
repo start OpenHarmony-v5.0.0-Release --all
```

**编译流程：**
1. 准备源码和环境
2. 构建 OpenHarmony 系统
3. 生成固件包（`.zip` 格式）
4. 使用 TitanFlasher 刷机

## 使用场景

- **平板与手持设备**：MUSE Paper 系列，完整触摸屏、摄像头、传感器支持
- **开发与原型验证**：MUSE Pi/Card 系列，HDMI/MIPI-DSI 双输出
- **笔记本与双屏应用**：MUSE Book，支持 MIPI-DSI 转 eDP + HDMI 双屏
- **AI 边缘计算**：本地 LLM、语音识别/合成、视觉识别
- **鸿蒙生态适配**：国产化项目、多终端协同
- **IoT 与分布式系统**：4G/5G 模组、蓝牙、以太网连接

## 开发路径

**南向开发（系统）：**
1. 下载编译烧录固件
2. 添加自定义方案
3. 系统定制与调试

**北向开发（应用）：**
1. 下载预编译固件并烧录
2. 配置 DevEco Studio 开发环境
3. 开发鸿蒙应用

## 在 SpacemiT 中

- **全球首发**：RISC-V + OpenHarmony 5.0 原生鸿蒙量产方案
- **AI 融合**：同构 AI 算力，支持 Ollama、DeepSeek、YOLO、TTS/ASR
- **完整产品线**：覆盖平板（Paper）、开发板（Pi/Card）、笔记本（Book）
- **量产工具链**：TitanFlasher、工程测试应用、完整文档支持
- **K3 路线图**：OpenHarmony 6.1 开发中

## 相关词条

- [[Bianbu]] — SpacemiT 官方 Linux 发行版（Ubuntu 基础）
- [[../hardware/products/K1]] — K1 芯片（8×X60，2 TOPS AI）
- [[../hardware/products/K3]] — K3 芯片（8×X100+8×A100，60 TOPS AI）
- [[../hardware/products/K1-MUSE-Pi]] — MUSE Pi 开发板
- [[../hardware/products/K1-MUSE-BOOK]] — MUSE Book 笔记本

## 来源文档

- SpacemiT OpenHarmony 文档：C:\Users\luojunlin\docs-openharmony\zh\
- OpenHarmony 官网：https://www.openharmony.cn/
- SpacemiT 文档中心：https://www.spacemit.com/community/document?lang=zh
