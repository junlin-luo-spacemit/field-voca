---
tags: [riscv, atom, privileged]
domain: privileged
aliases: [CSR, Control and Status Register, 控制与状态寄存器]
source: https://github.com/riscv/riscv-isa-manual
date: 2026-07-22
---

# CSR (Control and Status Register) / CSR（控制与状态寄存器）

**English Name:** Control and Status Register  
**中文译名**：控制与状态寄存器

## Definition / 定义

**English:**  
Special register space accessed through dedicated `csr*` instructions (12-bit address, up to 4096 registers), used to control processor behavior and read processor status.

**中文:**  
通过专用 `csr*` 指令访问的特殊寄存器空间（12 位地址，最多 4096 个），用于控制处理器行为和读取处理器状态。

## Access Instructions / 访问指令

| Instruction / 指令 | Semantics / 语义 |
|------|------|
| `csrrw rd, csr, rs1` | rd←old value, csr←rs1 / rd←旧值，csr←rs1 |
| `csrrs rd, csr, rs1` | rd←old value, csr\|=rs1 (set bits) / rd←旧值，csr\|=rs1（置位） |
| `csrrc rd, csr, rs1` | rd←old value, csr&=~rs1 (clear bits) / rd←旧值，csr&=~rs1（清位） |
| `csrr rd, csr` | Read-only (pseudo-instruction, rs1=x0) / 只读（伪指令，rs1=x0） |
| `csrw csr, rs1` | Write-only (pseudo-instruction, rd=x0) / 只写（伪指令，rd=x0） |

## Key CSRs Quick Reference / 关键 CSR 速查

| CSR | Privilege / 特权级 | Function / 作用 |
|-----|--------|------|
| `mstatus` | M | Global interrupt enable, privilege mode status / 全局中断使能、特权模式状态 |
| `mtvec` | M | Trap entry address / trap 入口地址 |
| `mepc` | M | Exception return PC / 异常返回 PC |
| `mcause` | M | Trap cause / trap 原因 |
| `mhartid` | M | Current hart ID / 当前 hart ID |
| `satp` | S | Virtual memory config (page table base) / 虚拟内存配置（页表基地址） |
| `stvec` | S | S-mode trap vector / S 模式 trap 向量 |
| `sepc` | S | S-mode exception return PC / S 模式异常返回 PC |
| `scause` | S | S-mode trap cause / S 模式 trap 原因 |
| `cycle` | U/S/M | Processor cycle count (read-only) / 处理器时钟周期计数（只读） |
| `time` | U/S/M | Real-time clock (read-only) / 实时时钟（只读） |
| `instret` | U/S/M | Instructions retired count (read-only) / 已退休指令计数（只读）|

## Related Terms / 相关词条

[[privileged/vocabulary#CSR]], [[atoms/privilege-mode]]

## Source / 来源文档

RISC-V ISA Manual Volume II, Chapter 2
