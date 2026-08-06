---
tags: [riscv, atom, extensions]
domain: extensions
aliases: [M extension, 乘除法扩展]
source: https://github.com/riscv/riscv-isa-manual
date: 2026-07-22
---

# M Extension (Integer Multiplication and Division) / extension-M（整数乘除法扩展）

**English Name:** Integer Multiplication and Division Extension  
**中文译名**：整数乘除法扩展

## Definition / 定义

**English:**  
Adds hardware multiply and divide instructions to the base integer ISA. Implementations without this extension must emulate multiplication and division through software (multiple shift/add instructions), resulting in ~10–30× performance degradation.

**中文:**  
为基础整数 ISA 增加硬件乘法和除法指令。无此扩展的实现需通过软件模拟乘除法（多条移位/加法指令），性能差约 10–30×。

## Instruction List / 指令列表

| Instruction / 指令 | Operation / 操作 | Common Use / 常见用途 |
|------|------|---------|
| `MUL rd, rs1, rs2` | rd = (rs1×rs2)[XLEN-1:0] | Low product (regular multiplication) / 低位乘积（常规乘法） |
| `MULH rd, rs1, rs2` | rd = (rs1×rs2)[2XLEN-1:XLEN] | Signed high (large integers) / 有符号高位（大整数） |
| `MULHU` | Unsigned × unsigned high / 无符号×无符号高位 | |
| `MULHSU` | Signed × unsigned high / 有符号×无符号高位 | |
| `DIV rd, rs1, rs2` | rd = rs1 / rs2 (signed) / rd = rs1 / rs2（有符号）| |
| `DIVU` | Unsigned division / 无符号除法 | |
| `REM rd, rs1, rs2` | rd = rs1 % rs2 (signed) / rd = rs1 % rs2（有符号）| |
| `REMU` | Unsigned remainder / 无符号取余 | |
| `MULW/DIVW/REMW` | 32-bit operations (RV64 only) / 32位操作（RV64 专属） | Result sign-extended to 64-bit / 结果符号扩展到64位 |

## Edge Case Conventions / 边界情况约定

**(Avoiding hardware exceptions / 避免硬件异常):**
- **Divide by zero / 除以零**: `DIV → -1` (all ones / 全1), `REM → dividend / 被除数`
- **Overflow / 溢出** (`MIN_INT / -1`): `DIV → MIN_INT`, `REM → 0`

## Related Terms / 相关词条

[[extensions/vocabulary#M Extension]], [[atoms/RV32I]]

## Source / 来源文档

RISC-V ISA Manual Volume I, Chapter 7
