---
tags: [riscv, atom, isa]
domain: isa
aliases: [RV32I, 基础整数指令集]
source: https://github.com/riscv/riscv-isa-manual
date: 2026-07-22
---

# RV32I / RV64I (Base Integer Instruction Set) / RV32I / RV64I（基础整数指令集）

**English Name:** 32/64-bit Base Integer Instruction Set  
**中文译名**：32/64 位基础整数指令集

## Definition / 定义

**English:**  
The minimal yet complete instruction subset of RISC-V. RV32I contains 47 base instructions, sufficient to run a complete software stack (compiler, OS, applications). RV64I adds 64-bit operation instructions on this foundation.

**中文:**  
RISC-V 最小且完备的指令子集。RV32I 包含 47 条基础指令，足以运行完整软件栈（编译器、OS、应用程序）。RV64I 在此基础上增加 64 位操作指令。

## RV32I Instruction Categories / RV32I 包含的指令类别

| Category / 类别 | Count / 指令数 | Typical Instructions / 典型指令 |
|------|--------|---------|
| Integer arithmetic (immediate) / 整数计算（立即数） | 9 | ADDI, SLTI, ANDI, ORI, XORI, SLLI, SRLI, SRAI, LUI, AUIPC |
| Integer arithmetic (register) / 整数计算（寄存器） | 10 | ADD, SUB, SLT, AND, OR, XOR, SLL, SRL, SRA |
| Load / Store | 8 | LB, LH, LW, LBU, LHU, SB, SH, SW |
| Conditional branches / 条件分支 | 6 | BEQ, BNE, BLT, BGE, BLTU, BGEU |
| Unconditional jumps / 无条件跳转 | 2 | JAL, JALR |
| Memory ordering / 内存序 | 1 | FENCE |
| System calls / 系统调用 | 2 | ECALL, EBREAK |
| CSR access / CSR 访问 | 6 | CSRRW, CSRRS, CSRRC, CSRRWI, CSRRSI, CSRRCI (Zicsr) |

## RV64I Additions / RV64I 新增

LD/SD (64-bit Load/Store), ADDW/SUBW/SLLW/SRLW/SRAW (32-bit operations with sign extension), LWU (zero-extended 32-bit Load)

LD/SD（64位Load/Store）、ADDW/SUBW/SLLW/SRLW/SRAW（32位操作并符号扩展）、LWU（零扩展32位Load）

## Comparison with x86/ARM / 与 x86/ARM 对比

- **x86 base ISA**: Hundreds of instructions, complex variable-length encoding / 数百条指令，复杂变长编码
- **ARM Thumb-2**: Mixed 16/32-bit, much historical baggage / 混合 16/32 位，历史包袱多
- **RV32I**: 47 instructions, fixed 32-bit encoding, clean design suitable for teaching and new designs / 47 条、固定 32 位编码、设计整洁，适合教学和新设计

## Related Terms / 相关词条

[[01-isa基础/vocabulary#RV32I]], [[01-isa基础/atoms/XLEN]]

## Source / 来源文档

RISC-V ISA Manual Volume I, Chapter 2
