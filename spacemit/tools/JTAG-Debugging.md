---
tags: [tools, debug, jtag, spacemit]
domain: tools
aliases: [JTAG Debugging, JTAG调试, JTAG]
status: approved
created: 2026-08-06
updated: 2026-11-16
source: C:\Users\luojunlin\docs-tool\zh\user_guide\jtag_debug_user_guide.md
superseded_by:
---

# JTAG Debugging（JTAG 调试）

**中文译名**：JTAG 调试

## 定义

基于 JTAG 接口的低级调试方法，用于单步执行、断点设置与寄存器级调试，可直接访问 CPU 内核状态。

## 为什么重要

- **最底层调试**：比软件调试器更接近硬件
- **引导调试**：可调试 bootloader、内核早期启动
- **无需 OS**：裸机程序调试
- **硬件故障诊断**：CPU/内存/总线问题排查

## JTAG 功能

1. **调试功能**
   - 单步执行
   - 断点设置（硬件断点）
   - 寄存器读写
   - 内存访问
   - CPU 暂停/恢复

2. **烧录功能**
   - Flash 编程
   - Bootloader 烧录
   - eFuse 编程

3. **测试功能**
   - Boundary Scan（边界扫描）
   - 芯片连接测试

## 工具链

- **调试器硬件**：J-Link（官方手册以 J-Link 为例）、其他 OpenOCD 兼容调试器
- **软件工具**：**SpacemiT 提供的 OpenOCD** + `riscv64-unknown-linux-gnu-gdb`
- **注意**：GDB 随**交叉编译工具链**一起提供，需单独从资源下载中心获取 OpenOCD

## 调试流程

```
J-Link 调试器 → JTAG 接口 → K1/K3 芯片
       ↑
  OpenOCD（SpacemiT 定制）
       ↕  GDB 远程连接（默认 gdb port 1024 / telnet 4444）
```

**实际操作步骤**（依据官方手册）：

1. 硬件：J-Link 拔掉跳线帽（不对外供电），连接 JTAG 信号
2. 驱动：Windows 用 Zadig 安装 WinUSB 驱动；Linux 复制 OpenOCD 的 udev 规则并 `udevadm control --reload`
3. 启动 OpenOCD：运行 `bin` 目录下的 `k1.bat/sh`（K1）
4. 调试：用 `riscv64-unknown-linux-gnu-gdb` 执行 `target remote <ip>:<port>`

> ⚠️ 不要使用 VS Code 的 **Cortex-Debug** 插件 —— 那是 ARM Cortex 专用，不适用于 RISC-V 的 SpacemiT 芯片。

## 在 SpacemiT 中

- K1/K3 板子**内置 JTAG 调试功能**，需设置硬件（J-Link 拔跳线帽）并配置 USB 驱动
- K1 引脚定义见 MUSE Pi 用户使用指南；K3 见 K3 硬件方案 FAQ 的 "开发调试" 章节
- 用于 SoC 引导调试、内核调试、裸机程序调试

> OpenOCD 脚本中 `CLUSTERS {0 1}` 与 `CLUSTER0_COREIDS {0 1 2 3}` 对应 K1 的**双簇 X60 结构**（每簇 4 核），与 X60 的双簇设计一致。

## 与 Trace Tool 的区别

| 特性 | JTAG Debugging | Trace Tool |
|------|----------------|------------|
| 功能 | 暂停、断点、单步 | 记录执行流程 |
| 性能影响 | 暂停程序运行 | 不影响实时性 |
| 适用场景 | 逐行调试 | 性能分析、热点定位 |

## 相关词条

- [[Trace-Tool]] — 跟踪分析工具
- [[Flashing-Tool]] — 烧录工具
- [[Cross-Compilation-Toolchain]] — 交叉编译工具链（GDB 随其提供）
- [[../hardware/products/K1]] — K1 JTAG 接口
- [[../hardware/products/K3]] — K3 JTAG 接口

## 来源文档

- `docs-tool/zh/user_guide/jtag_debug_user_guide.md`（官方 JTAG 调试工具使用手册）
