---
tags: [study, day2, spacemit, boot-chain]
domain: software
status: approved
created: 2026-11-16
updated: 2026-11-16
source: C:\Users\luojunlin\docs-buildroot\zh\k1_buildroot\device\boot.md, C:\Users\luojunlin\docs-buildroot\zh\k3_buildroot\device\boot.md
---

# Day 2 · 第一课：启动链（Boot Chain）

## 学习目标

能画出从"通电"到"程序运行"的六个层级，说出每一层的名称和职责。

---

## 完整启动链（K1 / K3 通用）

```
通电
  │
  ▼
1. BROM（Boot ROM）
   · 芯片出厂固化，不可修改
   · 任务：从存储介质找到 FSBL 并加载
  │
  ▼
2. FSBL（First Stage Boot Loader）
   · SpacemiT 自定义的第一阶段引导
   · 任务：初始化内存控制器（DRAM 在此之前不可用），加载 OpenSBI
  │
  ▼
3. OpenSBI（RISC-V Supervisor Binary Interface）
   · RISC-V 专有固件层（x86/ARM 没有这一层）
   · 任务：初始化 M-Mode 特权级，为操作系统提供底层系统调用接口
  │
  ▼
4. U-Boot
   · 通用引导加载程序（开源 + SpacemiT 适配）
   · 任务：初始化硬件外设，加载内核 + 设备树（DTB）
   · 特殊用途：进入 Fastboot 模式 → 刷机（写入新固件）
  │
  ▼
5. Linux 内核（Kernel）
   · SpacemiT 适配版本（K1 Linux 6.1，K3 Linux 6.6+）
   · 任务：初始化所有驱动，挂载根文件系统
  │
  ▼
6. Rootfs（根文件系统）
   · 在这里选择：Bianbu / Buildroot / OpenWrt / OpenHarmony
   · 任务：提供用户空间，程序从这里开始运行
  │
  ▼
你的程序跑起来了 🎉
```

---

## 每一层"是谁做的"

| 层级 | 名称 | 谁做的 | 可不可以改 |
|---|---|---|---|
| 1 | BROM | SpacemiT 芯片内固化 | ❌ 不可改 |
| 2 | FSBL | SpacemiT 开发 | ✅ 可编译替换 |
| 3 | OpenSBI | RISC-V 开源社区 | ✅ 可配置 |
| 4 | U-Boot | 开源社区 + SpacemiT 适配 | ✅ 可配置 |
| 5 | Linux 内核 | Linux 社区 + SpacemiT BSP | ✅ 可编译 |
| 6 | Rootfs | SpacemiT / 开源（4 种可选） | ✅ 可选 |

---

## 存储介质上的分区布局

每一层固件存在存储介质（eMMC / NOR Flash / SD 卡）的不同分区里。
以 NOR + SSD 组合为例：

```
NOR Flash（固件）              SSD / eMMC（系统）
┌──────────────────────┐      ┌─────────────────────┐
│ bootinfo             │  0x0 │ bootfs（内核 + DTB） │
│ FSBL.bin             │ 40KB │ rootfs（根文件系统） │
│ opensbi.itb          │  1MB └─────────────────────┘
│ u-boot.itb           │  2MB
│ env（环境变量）       │ 512KB
└──────────────────────┘
```

**重要**：`FSBL.bin`、`opensbi.itb`、`u-boot.itb` 是**分开存放的三个文件**，刷机时需要分别烧写到对应分区。

---

## K3 比 K1 多了什么

K3 的启动链可以插入一个可选的安全固件层：

```
K1：BROM → FSBL → OpenSBI → U-Boot → Kernel → Rootfs
K3：BROM → FSBL → [OP-TEE] → OpenSBI → U-Boot → Kernel → Rootfs
                   ↑
         可选安全固件（Buildroot v1.0.9 起支持）
         存放在独立 TEE 分区（GPT 介质为 4MB 分区）
```

**OP-TEE**（Trusted Execution Environment）提供硬件级安全隔离，用于安全启动、密钥保护等。

---

## 刷机 vs 启动：两件不同的事

技术写作里这对概念最容易混淆：

| 概念 | 是什么 | 什么时候发生 |
|---|---|---|
| **启动（Boot）** | 按照存储里已有的固件，一步步把系统跑起来 | 每次上电或重启 |
| **刷机（Flash）** | 把新的固件/镜像写入存储介质，替换旧的 | 升级系统 / 初始化设备时 |

**刷机的原理**：U-Boot 进入 Fastboot 模式 → PC 通过 USB 传输镜像 → 写入 eMMC/NOR 对应分区

> 官方文档原文：刷机与启动是相互关联的，如果需要对刷机做自定义配置，启动可能也需要做相关的调整，反之亦然。

---

## OpenSBI 是什么？为什么 RISC-V 需要它？

**类比**：
- x86 有 **BIOS / UEFI** 管理硬件初始化和特权级转换
- RISC-V 有 **OpenSBI** 做同样的事

**RISC-V 三个特权级**（看得懂就行，不需要记）：

```
M-Mode（机器模式） ← OpenSBI 在这里运行
S-Mode（监管模式） ← Linux 内核在这里运行
U-Mode（用户模式） ← 你的程序在这里运行
```

写文档时看到"需要 OpenSBI 支持"、"SBI 版本"，知道它是 RISC-V 特有的"固件层"就够了。

---

## 易错点

| 错误 | 正确 |
|---|---|
| "启动和刷机是同一件事" | 完全不同：启动是**读取**运行，刷机是**写入**替换 |
| "OpenSBI 和 BIOS 是不同的东西" | 功能类似，OpenSBI 是 RISC-V 专有的 BIOS/UEFI 等价物 |
| "K3 和 K1 启动链完全一样" | K3 多了可选的 **OP-TEE** 安全固件层 |
| "固件是一个大文件" | FSBL / OpenSBI / U-Boot 是**三个独立文件**，分别存在不同分区 |

---

## 关键记忆点（5 个）

1. BROM 是芯片内固化的，**不可修改**
2. FSBL 是 SpacemiT **自定义**的第一阶段引导
3. **OpenSBI 是 RISC-V 专有**的，x86/ARM 没有
4. 刷机 ≠ 启动：刷机是**写入**固件，启动是**读取**运行
5. K3 多 **OP-TEE** 安全固件层（v1.0.9 起支持）

---

## 作业

画一张从"通电"到"程序运行"的流程图，标出：
1. 六个层级的名称
2. OpenSBI 和 U-Boot 各负责什么
3. 刷机发生在哪一层
4. K3 比 K1 多的可选层
