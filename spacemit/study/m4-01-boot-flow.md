---
tags: [study, m4, boot, startup, uboot, opensbi, spacemit]
domain: chip
aliases: [启动流程, Boot Flow]
status: needs_review
created: 2026-11-23
updated: 2026-11-23
---

# 启动流程：从上电到系统运行

> **本讲目标**：理解 K3 芯片从上电到 Linux 运行的完整启动链路，知道每个阶段的名称、作用和存放位置。
>
> **📌 说明**：本模块以 **K3** 为例讲解启动流程，但多级引导的设计思路适用于所有 SoC 芯片。

## 一、为什么需要多阶段启动

[M3](./m3-03-onchip-memory-and-boot.md) 已经介绍了 Boot ROM 和 SRAM 的用途。多阶段启动有两个根本原因：

1. **容量限制**：Boot ROM 只有 128 KB，无法放下完整的操作系统
2. **初始化顺序**：LPDDR5 在上电后需要初始化，初始化代码本身需要在 SRAM 中运行

每个阶段负责初始化硬件、加载下一阶段，直到系统完全运行。

## 二、K3 的完整启动链路

```
上电复位
  ↓
① Boot ROM（片上，128 KB）
  - 出厂固化，只读
  - 检测 Boot Strap 引脚，决定从哪个介质启动
  - 加载 FSBL 到 SRAM 并执行
  ↓
② FSBL（First Stage Boot Loader，一级引导）
  - 存放在 eMMC / SD 卡 / SPI Flash / UFS
  - 在 SRAM（512 KB）中运行
  - 初始化 LPDDR5、时钟、基本外设
  - 加载 OpenSBI + U-Boot 到 LPDDR5
  ↓
③ OpenSBI（Open Supervisor Binary Interface）
  - 提供 M 模式的固件服务接口
  - U-Boot 和 Linux 内核通过 SBI 调用访问底层硬件
  ↓
④ U-Boot（Universal Boot Loader）
  - 在 LPDDR5 中运行
  - 提供交互式命令行（开发调试用）
  - 加载 Linux 内核镜像和设备树（DTB）到内存
  ↓
⑤ Linux 内核
  - 解析设备树，识别硬件
  - 挂载根文件系统
  - 启动用户空间进程（init / systemd）
  ↓
⑥ 用户空间
  - 应用程序、服务正常运行
```

## 三、每个阶段的关键点

### FSBL（一级引导）

**FSBL** 是启动链路中最关键的一环，因为它要在 LPDDR5 还没有准备好的情况下运行。

- 运行在片上 SRAM（512 KB）中
- 完成 LPDDR5 初始化（时序训练、参数配置）
- 初始化时钟树
- 通过 bootinfo 结构找到下一阶段固件的位置

**bootinfo** 是固件布局的索引结构，存放在存储介质的固定偏移位置（eMMC 为 0x100000 或 0x110000 字节处），记录了 FSBL 和后续固件的位置信息。

### OpenSBI

**OpenSBI**（RISC-V Open Supervisor Binary Interface）是 RISC-V 生态的固件层，相当于 Arm 平台的 TF-A（Trusted Firmware-A）。

- 运行在 M 模式（Machine Mode，最高特权级）
- 为 S 模式（Supervisor Mode，操作系统）提供标准服务接口（SBI）
- Linux 内核通过 SBI 调用访问定时器、IPI、电源管理等底层功能

**为什么需要 OpenSBI？**  
RISC-V 把底层固件与操作系统之间的接口标准化为 SBI，让 Linux 内核不需要针对每个芯片写不同的底层代码。

### U-Boot

**U-Boot** 是最常见的嵌入式 Bootloader，负责把内核和设备树从存储介质加载到内存。

- 支持多种文件系统（FAT、ext4）
- 支持网络启动（TFTP）
- 提供交互式命令行，方便开发调试
- 通过环境变量配置启动参数（`bootargs`）

### 设备树（DTB）

**设备树**（Device Tree Blob，DTB）描述了板子的硬件配置：有哪些外设、连接在哪个总线、使用哪个中断。

- U-Boot 把 DTB 和内核一起加载到内存
- 内核启动时读取 DTB，据此初始化驱动

**为什么需要设备树？**  
同一颗 K3 芯片可能装在不同的板子上，接口配置各不相同。设备树让内核代码保持通用，只需替换 DTB 就能支持不同的板级设计。

## 四、固件存放布局

以 eMMC 为例，K3 的固件按 GPT 分区存放：

| 分区 | 内容 | 说明 |
|---|---|---|
| bootinfo | 固件索引 | 固定偏移 0x100000，记录各固件位置 |
| fsbl | FSBL 二进制 | SRAM 中运行的一级引导 |
| opensbi | OpenSBI 固件 | M 模式固件层 |
| uboot | U-Boot 二进制 | Bootloader |
| bootfs | 内核 + DTB | Linux 内核镜像和设备树 |
| rootfs | 根文件系统 | 操作系统文件 |
| userdata | 用户数据 | 应用数据（可选） |

## 快速自测

1. FSBL 和 U-Boot 分别在哪里运行（SRAM 还是 LPDDR5）？各自做什么？
2. OpenSBI 在 RISC-V 启动链路中的作用是什么？
3. 设备树（DTB）解决了什么问题？

## 参考来源

| 来源 | 章节 | 内容 |
|---|---|---|
| docs-chip · key_stone/k3/k3_docs/k3_ds.md | 2.2.1 片上存储 | Boot ROM 128KB、SRAM 512KB |
| docs-buildroot · k3_buildroot/device/boot.md | 固件布局、刷机流程 | FSBL、bootinfo、GPT 分区、Fastboot 协议 |
