---
tags: [study, m3, sram, boot-rom, startup, spacemit]
domain: chip
aliases: [片上存储与启动, On-Chip Memory and Boot]
status: needs_review
created: 2026-11-19
updated: 2026-11-19
---

# 片上存储与启动流程

> **本讲目标**：理解 K3 片上存储（Boot ROM、SRAM）的用途，理解芯片上电后如何从 Boot ROM 一步步加载操作系统，以及各种存储接口在启动过程中扮演的角色。
>
> **📌 说明**：本模块以 **K3** 为例讲解启动流程，但片上存储的层次结构和多级引导的设计思路适用于所有 SoC 芯片。

## 一、K3 的片上存储

K3 集成了两种片上存储：

| 类型 | 容量 | 用途 |
|---|---|---|
| Boot ROM | 128 KB | 一级引导代码，只读，出厂固化 |
| SRAM | 512 KB | 主 CPU 和 RCPU 共享，掉电数据丢失 |

**Boot ROM（启动只读存储器）**：
- 芯片出厂时固化，不可修改
- 存放最初的引导代码（一级引导）
- 支持从多种外部介质启动（eMMC、UFS、SD、QSPI NOR Flash、USB、UART）
- 支持基于 eFuse 的安全启动

**SRAM（静态随机存取存储器）**：
- 芯片内部的高速静态内存
- 由主 CPU（X100/A100）和实时核（RCPU）共享使用
- 上电后立即可用，不需要初始化（不像 LPDDR5 需要训练）
- 容量小，主要用于启动早期的临时数据和代码

## 二、为什么需要多级引导？

芯片上电后无法直接运行完整的 Linux 内核。原因：

1. **内存未就绪**：LPDDR5 需要训练（Training）才能使用，而训练代码本身需要地方运行
2. **容量限制**：Boot ROM 只有 128 KB，放不下完整的 Bootloader
3. **安全要求**：需要逐步验证每个阶段的完整性

**解决方案**：多级引导（Multi-stage Boot）

```
Boot ROM（128 KB，片上）
    ↓ 加载
一级引导代码（SPL/FSBL，存在 QSPI/eMMC）
    ↓ 初始化 LPDDR5，加载
U-Boot（存在 eMMC/UFS）
    ↓ 加载内核和设备树
Linux 内核（存在 eMMC/UFS）
    ↓ 挂载
根文件系统（存在 eMMC/UFS）
```

## 三、各存储接口在启动中的角色

| 阶段 | 执行位置 | 代码存放在哪里 |
|---|---|---|
| 上电复位 | Boot ROM（片上，128 KB） | 出厂固化 |
| 一级引导（SPL） | SRAM（片上，512 KB） | QSPI NOR Flash 或 eMMC 启动分区 |
| U-Boot | LPDDR5（已初始化） | eMMC 或 UFS |
| Linux 内核 | LPDDR5 | eMMC 或 UFS |
| 根文件系统 | LPDDR5（挂载） | eMMC、UFS 或 SD 卡 |

**关键点**：每个阶段的代码都比上一阶段大，存放位置也从片上存储逐步移到外部存储。

## 四、启动模式选择

K3 支持多种启动介质，通过硬件引脚（Boot Mode Pins）在上电时选择：

| 启动模式 | 存储介质 | 典型用途 |
|---|---|---|
| eMMC 启动 | eMMC | 量产设备，系统固化在 eMMC |
| UFS 启动 | UFS | 高性能设备 |
| SD 卡启动 | SD/MMC | 开发调试，方便换系统镜像 |
| QSPI 启动 | QSPI NOR Flash | 存放 SPL，快速启动 |
| USB 下载 | USB | 烧录固件，恢复砖机 |
| UART 下载 | UART | 调试用，速度慢 |

**为什么开发板通常支持 SD 卡启动？**  
SD 卡可以热插拔，更换系统镜像只需换一张卡。量产设备通常固定用 eMMC 启动，SD 卡槽只作为数据扩展用。

## 五、安全启动

**安全启动（Secure Boot）** 基于 eFuse（一次性可编程熔断丝）：

1. **eFuse** 中存储了芯片厂商或设备厂商的公钥哈希值，出厂后无法修改
2. 每个引导阶段都会**验证下一阶段的签名**，确保代码没有被篡改
3. 如果验证失败，启动停止，防止恶意代码运行

**数据流**：
```
Boot ROM 读取 eFuse 公钥哈希
    ↓ 验证
SPL 的数字签名 → 验证通过 → 执行 SPL
    ↓ 验证
U-Boot 的数字签名 → 验证通过 → 执行 U-Boot
    ↓ 验证
内核和设备树的签名 → 验证通过 → 启动系统
```

## 快速自测

1. Boot ROM 和 SRAM 各有什么用途？
2. 为什么需要多级引导，而不是直接从 Boot ROM 加载 Linux？
3. 安全启动如何保证每个引导阶段不被篡改？

## 参考来源

| 来源 | 章节 | 内容 |
|---|---|---|
| docs-chip · key_stone/k3/k3_docs/k3_ds.md | 2.2.1 片上存储 | Boot ROM 128KB、SRAM 512KB 的配置 |
| docs-chip · key_stone/k3/k3_docs/k3_ds.md | 启动模式 | 多种启动介质支持、安全启动 |
