---
tags: [riscv, atom, privileged]
domain: privileged
aliases: [trap, exception, interrupt, 陷阱, 异常, 中断]
status: approved
created: 2026-07-22
updated: 2026-07-22
source: https://github.com/riscv/riscv-isa-manual
superseded_by:
---

# trap (Trap) / trap（陷阱）

**English Name:** Trap (collective term for exceptions and interrupts)  
**中文译名**：陷阱（异常与中断的统称）

## Definition / 定义

**English:**  
RISC-V collectively refers to exceptions and interrupts as traps. Both share the same hardware handling mechanism (save context → jump to handler → restore context).

**中文:**  
RISC-V 中将异常（exception）和中断（interrupt）统称为 trap，两者共用相同的硬件处理机制（保存现场 → 跳转处理函数 → 恢复现场）。

## Exception vs Interrupt Comparison / 两类 trap 对比

| Property / 属性 | Exception / 异常 | Interrupt / 中断 |
|------|-----------------|-----------------|
| Trigger timing / 触发时机 | Synchronous, directly caused by current instruction / 同步，由当前指令直接引发 | Asynchronous, independent of instruction flow / 异步，与指令流无关 |
| mcause MSB / mcause 最高位 | 0 | 1 |
| Typical sources / 典型来源 | Illegal instruction, page fault, ecall / 非法指令、缺页、ecall | Timer, peripherals, IPI / 定时器、外设、IPI |
| Resume point / 处理后恢复点 | mepc (usually trigger instruction or next) / mepc（通常是触发指令或下一条）| mepc (interrupted instruction) / mepc（被中断的指令）|

## Trap Hardware Handling Flow / trap 硬件处理流程

**(M-mode example / 以 M 模式为例):**
```
Trap triggered / 触发 trap
  → mepc ← PC (exception: current instruction; interrupt: next instruction)
            (异常：当前指令；中断：下一条)
  → mcause ← cause code / 原因码
  → mtval ← additional info (page fault address/illegal instruction encoding)
            附加信息（缺页地址/非法指令编码）
  → mstatus.MPIE ← mstatus.MIE (save interrupt enable / 保存中断使能)
  → mstatus.MIE ← 0 (disable interrupts / 关闭中断)
  → mstatus.MPP ← current mode / 当前模式
  → PC ← mtvec (jump to handler / 跳转处理函数)
```

## Common mcause Values / 常用 mcause 值

| Value / 值 | Type / 类型 | Cause / 原因 |
|----|------|------|
| 0x0000...0002 | Exception / 异常 | Illegal instruction / 非法指令 |
| 0x0000...0008 | Exception / 异常 | Environment call from U-mode / U 模式环境调用 |
| 0x0000...000D | Exception / 异常 | Load page fault / 加载缺页 |
| 0x0000...000F | Exception / 异常 | Store page fault / 存储缺页 |
| 0x8000...0007 | Interrupt / 中断 | Machine timer interrupt / 机器定时器中断 |
| 0x8000...000B | Interrupt / 中断 | Machine external interrupt / 机器外部中断 |

## Related Terms / 相关词条

[[02-特权架构/vocabulary#trap]], [[02-特权架构/atoms/CSR]], [[02-特权架构/atoms/privilege-mode]]

## Source / 来源文档

RISC-V ISA Manual Volume II, Chapter 3
