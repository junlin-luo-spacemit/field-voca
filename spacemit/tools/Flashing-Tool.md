---
tags: [tools, flashing, spacemit, titantools, firmware]
domain: tools
aliases: [Flashing Tool, 烧录工具, 固件烧录工具, Titan Flasher, TitanTools]
status: approved
created: 2026-08-06
updated: 2026-10-29
source: C:\Users\luojunlin\docs-tool\zh\user_guide\flasher_user_guide.md
superseded_by:
---

# Flashing Tool（TitanTools）

**官方名称**：TitanTools  
**中文译名**：烧录工具 / 固件烧录工具 / 刷机工具

## 定义

SpacemiT 官方刷机工具 **TitanTools**，用于将固件或镜像写入 SpacemiT 设备存储（eMMC、SD 卡、NOR Flash 等），支持 Windows 和 Linux 平台，提供单机烧录、批量烧录、量产卡制作和设备写号功能。

## 为什么重要

- **系统部署**：将编译好的镜像写入设备（Bianbu、OpenWrt、Buildroot、OpenHarmony）
- **固件更新**：更新 OpenSBI、U-Boot、内核、根文件系统
- **恢复系统**：设备变砖后的救援手段
- **量产支持**：支持多机同时烧录、量产卡制作、EEPROM 写号
- **在线云设备**：远程设备管理功能

## 下载与安装

### PC 配置要求

- **操作系统**：Windows 或 Linux
- **存储空间**：C 盘（Windows）或 Home 目录（Linux）需大于 10GB

### 下载地址

| 平台 | 架构 | 类型 | 下载链接 |
|------|-----|------|---------|
| **Windows** | X86/X64 | Installer | [Download](https://cloud.spacemit.com/prod-api/release/download/tools?token=titantools_for_windows_X86_X64) |
| **Linux** | X64 | AppImage | [Download](https://cloud.spacemit.com/prod-api/release/download/tools?token=titantools_for_linux_64BIT_APPIMAGE) |

### Windows 安装步骤

1. 下载 `titantools_for_windows.exe`
2. 双击安装程序
3. 允许未知发布者应用权限
4. 如提示驱动程序警告，选择"始终安装此驱动软件"
5. 选择安装位置
6. 完成安装后，选择界面语言并启动工具

### Linux 安装步骤

1. 下载 `titantools_for_linux.AppImage`
2. 赋予可执行权限：`chmod +x titantools_for_linux.AppImage`
3. 双击即可运行（无需安装）

**Linux 依赖问题修复：**
```bash
# 如报错 "dlopen(): error loading libfuse.so.2"
sudo apt install libfuse2
```

## 进入刷机模式

### 方法 1：设备未上电

1. 按住 **FDL（下载）按键**不松开
2. 插上电源线
3. 松开 FDL 按键
4. 插上 USB 数据线到刷机 USB 连接器

### 方法 2：设备已上电

1. 按住 **FDL 按键**不松开
2. 短按 **Reset 键**
3. 松开 FDL 按键
4. 插上 USB 数据线到刷机 USB 连接器

### 特殊设备：MUSE Book

1. 使用取卡针长按右侧小孔内的 **Fastboot 按钮**
2. 同时按下电源键开机（持续 3 秒以上）
3. 通过靠近屏幕侧的 Type-C 接口（OTG 专用端口）连接主机电脑

## 支持的存储介质

- **eMMC** — 板载存储（主流方案）
- **SD 卡** — TF 卡启动或刷机
- **SPI NAND Flash** — 部分方案支持
- **SPI NOR Flash** — MUSE Book 等设备（NOR + SSD 组合）
- **SSD** — MUSE Book（NOR 引导 + SSD 系统）

## 核心功能

### 1. 研发工具

#### 单机烧录
1. 确保设备进入刷机模式
2. 点击"扫描设备"，选择目标设备
3. 选择刷机文件来源：
   - **本地文件**：`.zip` 固件包
   - **本地目录**：解压后的固件目录
   - **网络文件**：在线下载固件
4. 点击"开始刷机"，等待完成

**支持的固件格式：**
- `.zip` 格式（Titan Flasher 专用包）
- 解压目录（包含分区镜像）

#### 卡启动
- 将镜像写入 SD 卡（`.img` 格式）
- 从 SD 卡启动设备
- 可用于系统安装或调试

### 2. 量产工具

#### 多机烧录
- 同时连接多个设备
- 批量刷写相同固件
- 适合生产环境

#### 制作量产卡
- 创建自动刷机 SD 卡
- 插卡自动完成固件烧录
- 简化产线操作

#### 写号工具
- EEPROM 写号（TLV 格式）
- 支持字段：
  - Product Name（产品名称）
  - Serial Number（序列号）
  - MAC Address（MAC 地址）
  - Manufacture Date（生产日期）
  - Device Version（设备版本）

#### 多机写号
- 批量写号支持
- 提高产线效率

### 3. 在线云设备
- 远程设备管理（开发中）
- 云端固件分发

### 4. 设置
- 工具版本更新
- 界面语言切换（中文/English）

## 烧录流程示例

### MUSE Pi 刷机流程

```plaintext
1. 设备准备
   ├─ 按住 FDL 按键
   ├─ 插上电源线
   ├─ 松开 FDL 按键
   └─ 连接 USB 数据线

2. 扫描设备
   ├─ 打开 TitanTools
   ├─ 点击"扫描设备"
   └─ 选择识别到的设备序列号

3. 选择固件
   ├─ 选择文件来源（本地/网络）
   ├─ 浏览选择 .zip 固件包
   └─ 或选择解压后的固件目录

4. 开始刷机
   ├─ 点击"开始刷机"
   ├─ 等待进度条完成
   └─ 刷机成功后自动重启
```

## 使用场景

### 开发阶段
- 刷写开发版本固件（Bianbu、Buildroot、OpenWrt、OpenHarmony）
- 测试新内核或驱动
- 系统恢复与调试

### 生产阶段
- 批量刷写量产固件
- EEPROM 写号（MAC、S/N）
- 制作量产卡批量部署

### 维护阶段
- 固件升级
- 系统修复
- 重新分区

## 支持的设备

**MUSE 系列：**
- MUSE Pi（eMMC 启动）
- MUSE Pi Pro（eMMC 启动）
- MUSE Card（TF 卡启动）
- MUSE Book（NOR + SSD 启动）
- MUSE Paper / Paper2（eMMC / TF 卡启动）

**开发板：**
- BPI-F3
- deb1 / deb2

**集群服务器：**
- RV2768（768 核集群）

## 常见问题

### Q: 扫描不到设备？
**A**: 确认设备已进入刷机模式（FDL 模式），检查 USB 连接和驱动安装。

### Q: 刷机失败怎么办？
**A**: 
1. 检查固件包完整性
2. 重新进入刷机模式
3. 更换 USB 线或端口
4. 查看工具日志

### Q: 多个设备如何选择？
**A**: 点击设备下拉菜单，选择对应的刷机序列号。

### Q: 支持哪些固件格式？
**A**: 
- `.zip`（推荐）— Titan Flasher 专用打包格式
- 解压目录 — 包含所有分区镜像的目录

## 在 SpacemiT 生态中

- **Bianbu**：刷写 Ubuntu 桌面系统
- **OpenWrt**：刷写软路由/NAS 固件
- **Buildroot**：刷写最小化嵌入式系统
- **OpenHarmony**：刷写鸿蒙系统（K1 OH5.0）
- **工厂测试**：配合 factorytest 应用完成产测

## 相关词条

- [[JTAG-Debugging]] — 硬件级调试工具
- [[SpacemiT-Studio]] — 一站式 Web 开发工作台（集成刷机功能）
- [[../software/SDK]] — 提供待烧录镜像
- [[../software/Bianbu]] — Bianbu 系统刷机
- [[../software/OpenWrt]] — OpenWrt 固件刷机
- [[../software/Buildroot]] — Buildroot 固件刷机
- [[../software/OpenHarmony]] — OpenHarmony 固件刷机
- [[../hardware/products/K1-MUSE-Pi]] — MUSE Pi 刷机指南
- [[../hardware/products/K1-MUSE-BOOK]] — MUSE Book 刷机指南

## 来源文档

- SpacemiT 刷机工具使用手册：C:\Users\luojunlin\docs-tool\zh\user_guide\flasher_user_guide.md
- TitanTools 官方下载：https://cloud.spacemit.com/
- SpacemiT 文档中心：https://www.spacemit.com/community/document?lang=zh
