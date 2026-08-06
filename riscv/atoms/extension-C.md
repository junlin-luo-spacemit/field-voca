---
tags: [riscv, atom, isa]
domain: isa
aliases: [C extension, 压缩指令]
source: https://github.com/riscv/riscv-isa-manual
date: 2026-07-22
---

# C Extension (Compressed Instructions) / extension-C（压缩指令扩展）

**English Name:** Compressed Instructions Extension (16-bit encoding)  
**中文译名**：压缩指令扩展（16 位编码）

## Definition / 定义

**English:**  
Introduces 16-bit compressed instruction encoding, mixed with 32-bit instructions in the same program. Reduces code size by approximately 25–30%, providing significant benefits for embedded system Flash and I-cache utilization.

**中文:**  
引入 16 位压缩指令编码，与 32 位指令混合存在于同一程序中。代码体积减少约 25–30%，对嵌入式系统 Flash 和 I-cache 利用率有显著收益。

## Recognition Rule / 识别规则

**English:** If the lowest 2 bits of an instruction ≠ `11` → 16-bit compressed instruction; if = `11` → 32-bit standard instruction.

**中文:** 指令最低 2 位 ≠ `11` → 16 位压缩指令；= `11` → 32 位标准指令。

## Limitations of Compressed Instructions / 压缩指令的限制

- Most compressed instructions can only address **x8–x15** (s0-s1, a0-a5), requiring only 3-bit encoding / 大多数压缩指令只能寻址 **x8–x15**（s0-s1, a0-a5），只需 3 位编码
- Smaller immediate range / 立即数范围更小
- Not all operations have a corresponding compressed version / 不是所有操作都有对应压缩版本

## Common Compressed Instructions / 常用压缩指令

| Compressed / 压缩指令 | Equivalent 32-bit / 等价 32 位指令 |
|---------|--------------|
| `c.addi rd, imm` | `addi rd, rd, imm` |
| `c.li rd, imm` | `addi rd, x0, imm` |
| `c.lw rd', imm(rs1')` | `lw rd', imm(rs1')` |
| `c.sw rs2', imm(rs1')` | `sw rs2', imm(rs1')` |
| `c.j offset` | `jal x0, offset` |
| `c.jr rs1` | `jalr x0, rs1, 0` |
| `c.jalr rs1` | `jalr x1, rs1, 0` |
| `c.mv rd, rs2` | `add rd, x0, rs2` |

## Code Size Comparison / 代码体积对比

**(Typical Linux kernel / 典型 Linux 内核):**
- RV64G (no C extension) / RV64G（无 C 扩展）: ~25–30% larger / 大 25–30%
- RV64GC: Baseline / 基准
- Equivalent ARM64: Similar to RV64GC / 同等功能的 ARM64：类似 RV64GC

## Related Terms / 相关词条

[[01-isa基础/vocabulary#C Extension]], [[04-微架构/vocabulary#C 扩展对微架构的挑战]]

## Source / 来源文档

RISC-V ISA Manual Volume I, Chapter 16
