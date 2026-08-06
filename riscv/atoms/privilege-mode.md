---
tags: [riscv, atom, privileged]
domain: privileged
aliases: [privilege mode, M-mode, S-mode, U-mode]
source: https://github.com/riscv/riscv-isa-manual
date: 2026-07-22
---

# Privilege Mode / privilege-mode（特权模式）

**English Name:** Privilege Mode  
**中文译名**：特权模式

## Definition / 定义

**English:**  
A RISC-V processor runs at a specific privilege level at any given time, determining which instructions, CSRs, and memory regions can be accessed. Accessing higher privilege resources from a lower privilege level triggers an illegal instruction exception.

**中文:**  
RISC-V 处理器在任意时刻运行于某一特权级别，决定哪些指令、CSR、内存区域可以访问。越低特权级访问越高特权级的资源会触发非法指令异常。

## Privilege Modes / 特权模式

| Mode / 模式 | Encoding / 编码 | Runs / 运行内容 | Required / 必须支持 |
|------|------|---------|---------|
| M (Machine) | 11 | Firmware (OpenSBI) / 固件（OpenSBI） | ✅ All implementations / 所有实现 |
| S (Supervisor) | 01 | OS kernel (Linux) / OS 内核（Linux） | Optional / 可选 |
| U (User) | 00 | User applications / 用户应用 | Optional / 可选 |
| H (Hypervisor) | — | VMM (extension) / VMM（扩展） | Optional / 可选 |

## Typical Implementation Combinations / 典型实现组合

- **M only**: Simple MCU, firmware directly runs user code / 简单 MCU，固件直接运行用户代码
- **M + U**: Embedded systems with basic user/privilege isolation (no OS virtual memory) / 嵌入式系统，有基本用户/特权隔离（无 OS 虚拟内存）
- **M + S + U**: Application processors running Linux (most common) / 运行 Linux 的应用处理器（最常见）
- **M + H + S + U**: Server SoCs with virtualization support / 支持虚拟化的服务器 SoC

## Related Terms / 相关词条

[[02-特权架构/vocabulary#privilege mode]], [[atoms/CSR]], [[atoms/trap]]

## Source / 来源文档

RISC-V ISA Manual Volume II, Chapter 1
