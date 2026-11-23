---
tags: [study, m4, boot, boot-mode, fastboot, firmware, spacemit]
domain: chip
aliases: [启动模式与固件刷写, Boot Mode and Firmware Flashing]
status: needs_review
created: 2026-11-23
updated: 2026-11-23
---

# 启动模式与固件刷写

> **本讲目标**：理解 K3 如何通过硬件引脚选择启动介质，知道两种启动大类（下载模式和正常启动），以及如何把固件烧写到设备上。
>
> **📌 说明**：本模块以 **K3** 为例讲解启动模式，但 Boot Strap 引脚选择启动介质的机制适用于所有 SoC 芯片。

## 一、两大启动类型

K3 支持两大类启动模式：

| 类型 | 用途 | 典型场景 |
|---|---|---|
| 下载模式（Download Mode） | 通过有线接口接收固件 | 烧录新固件、恢复砖机、调试 |
| 正常启动模式（Normal Boot） | 从存储介质加载引导程序 | 量产设备日常启动 |

**下载模式**下设备不从存储介质启动，而是等待 PC 通过 USB 或 UART 把固件传过来。两种具体方式：
- **USB Fastboot**：通过 USB 2.0 接口，使用 Fastboot 协议连接主机，速度快
- **UART Xmodem**：通过 UART 接口，使用 Xmodem 协议，速度慢，适合无 USB 的场景

## 二、Boot Strap 引脚

K3 使用 4 个 Boot Strap 引脚（GPIO_69、GPIO_68、GPIO_66、GPIO_65）决定启动模式：

| GPIO_69 | GPIO_68 | GPIO_66 | GPIO_65 | 启动模式 |
|---|---|---|---|---|
| 1 | 0 | x | x | USB Fastboot（下载模式） |
| 1 | 1 | x | x | UART Xmodem（下载模式） |
| 0 | x | 0 | 0 | SD 卡 → eMMC（正常启动） |
| 0 | x | 0 | 1 | SD 卡 → SPI NOR Flash |
| 0 | x | 1 | 0 | SD 卡 → SPI NAND Flash |
| 0 | x | 1 | 1 | SD 卡 → UFS |

> `x` 表示该引脚不影响结果。

**关键点**：K3 始终**优先尝试从 SD 卡启动**。只有检测不到 SD 卡，或 SD 卡中没有有效引导程序镜像，才切换到 Boot Strap 引脚选择的次级启动介质。

**这就是为什么开发板插上 SD 卡后自动从 SD 卡启动**——哪怕 Boot Strap 配置的是 eMMC，SD 卡的优先级更高。

## 三、固件布局

K3 以 GPT（GUID Partition Table）分区方式组织固件。以 eMMC 为例，典型分区布局：

| 分区 | 偏移/说明 | 内容 |
|---|---|---|
| bootinfo | 固定偏移 0x100000 | 固件索引，记录各固件的位置 |
| fsbl | — | 一级引导程序（FSBL） |
| opensbi | — | OpenSBI 固件 |
| uboot | — | U-Boot |
| bootfs | — | Linux 内核 + 设备树 |
| rootfs | — | 根文件系统 |

**bootinfo** 是固件布局的关键：FSBL 通过 bootinfo 找到自己和后续固件的位置，而不是硬编码地址。这让固件布局更灵活，支持 A/B 分区等更新策略。

## 四、固件刷写流程

**刷写**（也称"烧录"）是把 PC 上的固件镜像传输到设备存储介质的过程。K3 使用 Fastboot 协议完成刷写。

### 进入 U-Boot Fastboot 模式的三种方法

**方法 1：按键组合（无系统时使用）**
- 按下板子上的 FEL 键 + RESET 键，进入 BROM-Fastboot 模式
- 然后通过 PC 加载 FSBL 和 U-Boot，进入 U-Boot Fastboot 模式

**方法 2：ADB 命令（已有系统时使用）**
```
adb reboot bootloader
```

**方法 3：串口（开发调试时使用）**
- 启动时通过串口长按 `s` 键，进入 U-Boot Shell
- 执行 `fastboot 0` 进入 U-Boot Fastboot 模式

### 验证连接

进入 Fastboot 模式后，在 PC 执行：
```
fastboot devices
```
看到设备列出即表示连接成功。

### 刷写镜像

以 eMMC 为例，完整刷写顺序：
```
fastboot flash gpt partition_universal.json   # 创建 GPT 分区表
fastboot flash bootinfo factory/bootinfo_block.bin
fastboot flash fsbl factory/FSBL.bin
fastboot flash opensbi fw_dynamic.itb
fastboot flash uboot u-boot.itb
fastboot flash bootfs bootfs.img
fastboot flash rootfs rootfs.img
fastboot reboot
```

### 卡量产 vs 卡启动

| 概念 | 说明 |
|---|---|
| **卡启动** | 把镜像写入 SD 卡，直接从 SD 卡启动系统 |
| **卡量产** | 把镜像放在 SD 卡，插卡后自动刷写到 eMMC，完成后从 eMMC 启动 |

卡量产是大批量设备出厂烧录的常用方式，不需要逐台连电脑。

## 快速自测

1. K3 的 SD 卡启动优先级与其他介质相比如何？
2. 下载模式和正常启动模式各在什么场景下使用？
3. bootinfo 在固件布局中的作用是什么？

## 参考来源

| 来源 | 章节 | 内容 |
|---|---|---|
| docs-chip · key_stone/k3/k3_docs/k3_ds.md | 2.11 启动模式 | Boot Strap 引脚、下载模式、正常启动 |
| docs-buildroot · k3_buildroot/device/boot.md | 固件布局、刷机流程 | GPT 分区、bootinfo、Fastboot 刷写命令 |
