---
tags: [software, build, buildroot, spacemit, embedded]
domain: software
aliases: [Buildroot, 进迭Buildroot]
status: approved
created: 2026-08-06
updated: 2026-10-29
source: C:\Users\luojunlin\docs-buildroot\zh\k1_buildroot\
superseded_by:
---

# Buildroot

## 定义

Buildroot 是用于生成嵌入式 Linux 根文件系统与镜像的构建工具。SpacemiT 基于 Buildroot 构建完整的 Linux SDK，适配 K 系列芯片，包含监管程序接口（OpenSBI）、引导加载程序（U-Boot/UEFI）、Linux 内核、根文件系统（包含各种中间件和库）以及示例等，目标是为客户提供处理器 Linux 支持，并且可以开发驱动或应用。

## 为什么重要

- **完整 BSP 栈**：OpenSBI + U-Boot/UEFI + Linux Kernel + 完整中间件
- **硬件加速集成**：GPU (PowerVR)、VPU、JPU、2D 加速、AI 推理（onnxruntime）
- **最小化可定制**：只包含必要组件，内存与存储占用小
- **产品化就绪**：包含工厂测试、MPP、FFmpeg/GStreamer 硬件加速
- **快速构建**：自动化交叉编译与打包，支持 Docker 容器编译

## 核心组件

**已集成组件：**
- **OpenSBI** — RISC-V 监管程序接口
- **U-Boot** — 引导加载程序（支持 UEFI）
- **Linux Kernel** — Linux 6.1 或 6.6
- **Buildroot** — 根文件系统构建框架
- **onnxruntime（硬件加速）** — AI 推理框架
- **ai-support** — AI 演示程序
- **img-gpu-powervr** — GPU DDK（PowerVR 驱动）
- **mesa3d** — 3D 图形库
- **QT 5.15（GPU 加速）** — GUI 框架
- **k1x-vpu-firmware** — 视频处理单元固件
- **k1x-vpu-test** — VPU 测试程序
- **k1x-jpu** — JPEG 处理单元 API
- **k1x-cam** — CMOS Sensor 和 ISP API
- **mpp** — Media Process Platform（媒体处理平台）
- **FFmpeg（硬件加速）** — 多媒体框架
- **GStreamer（硬件加速）** — 多媒体管道框架
- **v2d-test** — 2D 加速单元测试程序
- **factorytest** — 工厂测试应用

**关键依赖：**
- **esos.elf** — RCPU (Real-Time CPU) 固件，负责硬件模块初始化和 HDMI Audio 中断转发，缺失会导致无法启动（位于 `/lib/firmware/esos.elf`）

## 版本与分支

| 版本 | manifest.xml | 分支 | Linux 内核 |
|------|-------------|------|-----------|
| v1.0 | `bl-v1.0.y.xml` | `bl-v1.0.y` | Linux 6.1 |
| v2.0 | `bl-v2.0.y.xml` | `bl-v2.0.y` | Linux 6.6 |
| v2.1 | `k1-bl-v2.1.y.xml` | `k1-bl-v2.1.y` | Linux 6.6 |
| v2.2 | `k1-bl-v2.2.y.xml` | `k1-bl-v2.2.y` | Linux 6.6 |

**重要说明：**
- Gitee 单仓库容量限制导致 v2.0 和 v2.1 的 linux-6.6 分支被移动到独立仓库
- GitHub 仅托管 v2.2 及以后版本
- 自 Buildroot 2.2.7 开始，默认在 Docker 容器中构建（需安装 Docker CE）

## 使用场景

- **嵌入式设备与 IoT**：资源受限环境的最小化系统
- **高度定制产品**：完全控制组件选择与配置
- **工业边缘设备**：硬件加速、实时性能需求
- **快速原型开发**：自动化构建，快速迭代
- **产品化部署**：包含工厂测试、量产工具链

## 与 Bianbu 对比

| 特性 | Buildroot | Bianbu |
|------|-----------|--------|
| 基础 | 从零构建 | 基于 Ubuntu 23.10/26.04 |
| 大小 | 最小（几十 MB） | 完整（几 GB） |
| 软件生态 | 有限，手动添加 | Ubuntu apt 生态 |
| 定制性 | 极高 | 中等 |
| 桌面环境 | 可选，通常无 | GNOME/LXQt |
| 适用场景 | 嵌入式、资源受限、产品化 | 开发、桌面、通用计算 |

## 开发环境要求

**硬件推荐配置：**
- CPU: 12th Gen Intel Core i5 或以上
- Memory: 16GB 或以上
- Disk: SSD，256GB 或以上

**操作系统：**
- Buildroot 2.2.7+：Ubuntu 20.04+ 或支持 Docker 的 Linux 发行版
- Buildroot 2.2.6-：Ubuntu 20.04+ LTS

**依赖安装（Ubuntu 20.04+）：**
```bash
sudo apt-get install git build-essential cpio unzip rsync file bc wget \
  python3 python-is-python3 libncurses5-dev libssl-dev dosfstools mtools \
  u-boot-tools flex bison python3-pip
sudo pip3 install pyyaml
```

**Docker 编译（2.2.7+）：**
- 仅需安装 Docker CE
- 切换到宿主机构建：`export DIRECT_BUILD=1`

## 获取与构建

**预编译镜像下载：**
- 镜像地址：https://archive.spacemit.com/image/k1/version/bianbu-linux/
- 格式：`.zip`（Titan Flasher 刷机包）或解压后用 fastboot 刷机

**源码下载（v2.2 示例）：**

从 Gitee：
```bash
mkdir ~/buildroot-sdk-2.2
cd ~/buildroot-sdk-2.2
repo init -u git@gitee.com:spacemit-buildroot/manifests.git -b main -m k1-bl-v2.2.y.xml
repo sync
repo start k1-bl-v2.2.y --all
```

从 GitHub：
```bash
repo init -u git@github.com:spacemit-com/manifests.git -b main -m k1-bl-v2.2.y.xml
```

**编译流程：**
```bash
cd ~/buildroot-sdk
make envconfig  # 首次编译或修改 defconfig 后使用
# 选择方案：
# 1. spacemit_k1_upstream_defconfig
# 2. spacemit_k1_minimal_defconfig
# 3. spacemit_k1_plt_defconfig
# 4. spacemit_k1_rt_defconfig
# 5. spacemit_k1_v2_defconfig (Buildroot 2.x)

# 输出：
# output/k1_v2/images/buildroot-k1_v2.zip
# output/k1_v2/images/sdcard.img
```

**新命令（2.2.7+）：**
```bash
make help              # 查看新命令列表
make k1_v2-build       # 直接编译指定方案
```

## 目录结构

```
buildroot-sdk/
├── bsp-src/              # BSP 源码
│   ├── linux-6.6         # Linux 内核源码
│   ├── opensbi           # OpenSBI 源码
│   └── uboot-2022.10     # U-Boot 源码
├── buildroot/            # Buildroot 主目录
├── buildroot-ext/        # 客制化扩展
│   ├── board/            # 板级配置
│   ├── configs/          # 方案配置（defconfig）
│   ├── package/          # 自定义软件包
│   └── patches/          # 补丁
├── package-src/          # 本地展开的应用或库源码
│   ├── ai-support
│   ├── img-gpu-powervr   # GPU 驱动
│   ├── k1x-cam           # Camera/ISP API
│   ├── k1x-jpu           # JPEG 处理
│   ├── k1x-vpu-firmware  # VPU 固件
│   ├── mpp               # 媒体处理平台
│   ├── mesa3d            # 3D 图形
│   └── v2d-test          # 2D 加速测试
├── scripts/              # 编译脚本
└── Makefile              # 顶层 Makefile
```

## 设备与方案管理

SpacemiT Buildroot SDK 支持设备管理、方案管理、分区表配置、启动参数定制等高级功能，详细文档位于：
- 设备管理：`docs-buildroot/zh/k1_buildroot/device/device_management.md`
- 方案管理：`docs-buildroot/zh/k1_buildroot/device/solution_management.md`
- 启动开发：`docs-buildroot/zh/k1_buildroot/device/boot.md`

## 外设驱动支持

包含完整的外设驱动文档（22+ 外设），位于 `docs-buildroot/zh/k1_buildroot/device/peripheral_driver/`：
- PINCTRL, GPIO, PWM, IR-RX, UART, I2C, QSPI, SDHC
- GMAC（以太网）, PCIe, Display, V2D（2D 加速）
- PMIC, CAN, Cpufreq, Clock, Audio, CRYPTO, DMA, EtherCAT

## Camera/ISP 开发

包含完整的 Camera 和 ISP 开发文档：
- Camera 开发指南：`camera_development_guide.md`
- ISP API 开发指南：`isp_api_development_guide.md`
- ISP PQ 工具使用指南：`isp_pq_tools_user_guide.md`

## 在 SpacemiT 中

- **K 系列专用**：当前适配 K1 系列芯片（K1 有 5 种方案配置）
- **完整硬件栈**：GPU (PowerVR) + VPU + JPU + 2D + AI (onnxruntime)
- **产品化工具链**：工厂测试、Titan Flasher 刷机、EEPROM 自适应
- **容器化构建**：Docker 支持，环境隔离，构建可复现

## 相关词条

- [[Bianbu]] — SpacemiT 官方桌面 Ubuntu 发行版
- [[OpenWrt]] — 网络设备专用系统（K1 软路由/NAS）
- [[SDK]] — 软件开发套件
- [[../hardware/products/K1]] — K1 芯片（Buildroot 主要支持平台）

## 来源文档

- SpacemiT Buildroot 文档：C:\Users\luojunlin\docs-buildroot\zh\k1_buildroot\
- Buildroot 官网：https://buildroot.org/
- SpacemiT 镜像站：https://archive.spacemit.com/
