---
tags: [riscv, atom, isa]
domain: isa
aliases: [XLEN, 寄存器宽度]
source: https://github.com/riscv/riscv-isa-manual
date: 2026-07-22
---

# XLEN (Integer Register Width) / XLEN（整数寄存器宽度）

**English Name:** Integer Register Width  
**中文译名**：整数寄存器宽度

## Definition / 定义

**English:**  
The bit width of integer registers (x0–x31) in RISC-V, which simultaneously determines the effective address bit count and address space size.

**中文:**  
RISC-V 中整数寄存器（x0–x31）的位宽，同时决定有效地址的位数和地址空间大小。

## Base ISA Variants / 基础 ISA 变体

| Base ISA / 基础 ISA | XLEN | Address Space / 地址空间 |
|----------|------|---------|
| RV32I/E | 32 | 4 GB |
| RV64I/E | 64 | 16 EB (theoretical / 理论) |
| RV128I | 128 | Draft / 草案 |

## Impact of XLEN / XLEN 的影响范围

- General-purpose register width (each of x0–x31 is XLEN bits) / 通用寄存器宽度（x0–x31 每个均为 XLEN 位）
- Virtual/physical address width (but Sv39/48 doesn't use all 64 bits) / 虚拟/物理地址宽度（但 Sv39/48 并非使用全部 64 位）
- CSR width (most CSRs are XLEN bits) / CSR 宽度（大多数 CSR 为 XLEN 位）
- Immediate sign-extension target width / 立即数符号扩展目标宽度

## Related Terms / 相关词条

[[01-isa基础/vocabulary#XLEN]], [[atoms/RV32I]]

## Source / 来源文档

RISC-V ISA Manual Volume I, Chapter 1
