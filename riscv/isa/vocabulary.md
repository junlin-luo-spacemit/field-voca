---
tags: [riscv, isa, vocabulary]
domain: isa
last_updated: 2026-07-23
source: https://github.com/riscv/riscv-isa-manual
---

# RISC-V ISA Vocabulary / RISC-V 指令集架构词汇表

> Covers: Base ISA (RV32I/RV64I), instruction formats, registers, addressing modes, calling conventions
> Source: RISC-V ISA Manual (Unprivileged Specification)

> 涵盖：基础 ISA（RV32I/RV64I）、指令格式、寄存器、寻址模式、调用约定
> 来源：RISC-V ISA 手册（非特权规范）

---

## ISA Fundamentals / ISA 基础

### ISA (Instruction Set Architecture) / 指令集架构

The abstract interface between hardware and software, defining:
- Instruction encoding and semantics / 指令编码和语义
- Register set and memory model / 寄存器集和内存模型
- Exception/interrupt handling / 异常/中断处理
- Privilege modes / 特权模式

RISC-V ISA design principles: simplicity, modularity, extensibility.

RISC-V ISA 设计原则：简洁性、模块化、可扩展性。

---

### RISC (Reduced Instruction Set Computer) / 精简指令集计算机

Design philosophy emphasizing:
- Simple, regular instructions / 简单、规整的指令
- Load-store architecture / 加载-存储架构
- Large register file / 大型寄存器堆
- Fixed-length encoding (with exceptions for C extension) / 定长编码（C 扩展除外）

**RISC-V characteristics / RISC-V 特点：**
- Clean slate design (no legacy baggage) / 全新设计（无历史包袱）
- Open standard (free to implement) / 开放标准（自由实现）
- Modular extension system / 模块化扩展系统

---

### XLEN / XLEN

Native integer register width: 32-bit for RV32, 64-bit for RV64, 128-bit for RV128.

原生整数寄存器宽度：RV32 为 32 位，RV64 为 64 位，RV128 为 128 位。

**Naming convention / 命名约定：**
- `RV32I`: 32-bit base integer ISA / 32 位基础整数 ISA
- `RV64I`: 64-bit base integer ISA / 64 位基础整数 ISA
- `RV32E`: Embedded variant with 16 registers / 嵌入式变体，16 个寄存器

---

## Base Integer ISA (RVI) / 基础整数 ISA

### RV32I / RV32I

32-bit base integer instruction set, the foundation of RISC-V. Provides:

32 位基础整数指令集，RISC-V 的基础。提供：

- 32 general-purpose registers (x0-x31), each 32 bits / 32 个通用寄存器（x0-x31），每个 32 位
- ~47 instructions for computation, load/store, control flow / 约 47 条指令，用于计算、加载/存储、控制流
- PC (program counter) register / PC（程序计数器）寄存器
- Support for user and machine privilege modes / 支持用户和机器特权模式

**Key properties / 关键特性：**
- Little-endian memory / 小端内存
- Naturally aligned loads/stores / 自然对齐的加载/存储
- Two's complement integers / 二进制补码整数

---

### RV64I / RV64I

64-bit base integer ISA. Extends RV32I with:

64 位基础整数 ISA。在 RV32I 基础上扩展：

- 64-bit registers (x0-x31) / 64 位寄存器（x0-x31）
- Additional "W" suffix instructions operating on 32-bit values / 附加 "W" 后缀指令，操作 32 位值
- 64-bit addressing space / 64 位地址空间

**RV64I "W" instructions / RV64I "W" 指令：**
- `ADDW`, `SUBW`, `SLLW`, `SRLW`, `SRAW`: 32-bit operations, sign-extend result to 64 bits / 32 位操作，结果符号扩展到 64 位
- `LW`: Loads 32-bit value, sign-extends to 64 bits / 加载 32 位值，符号扩展到 64 位
- `LWU`: Loads 32-bit value, zero-extends to 64 bits / 加载 32 位值，零扩展到 64 位

---

### RV32E / RV32E

Embedded variant with reduced register set:
- Only 16 registers (x0-x15) instead of 32 / 仅 16 个寄存器（x0-x15）而非 32 个
- Reduces register file size by 50% / 寄存器堆大小减少 50%
- Suitable for deeply embedded systems / 适合深度嵌入式系统

**Trade-offs / 权衡：**
- Smaller area and power / 更小面积和功耗
- Reduced code density due to fewer registers / 由于寄存器较少，代码密度降低
- Still compatible with standard calling convention (with restrictions) / 仍与标准调用约定兼容（有限制）

---

## Register Set / 寄存器集

### General-Purpose Registers / 通用寄存器

32 integer registers (x0-x31), each XLEN bits wide.

32 个整数寄存器（x0-x31），每个 XLEN 位宽。

**Register naming / 寄存器命名：**

| Register | ABI Name | Description / 说明 | Saver / 保存者 |
|----------|----------|-------------------|---------------|
| x0 | zero | Hard-wired zero / 硬连线零 | — |
| x1 | ra | Return address / 返回地址 | Caller |
| x2 | sp | Stack pointer / 栈指针 | Callee |
| x3 | gp | Global pointer / 全局指针 | — |
| x4 | tp | Thread pointer / 线程指针 | — |
| x5-x7 | t0-t2 | Temporaries / 临时寄存器 | Caller |
| x8 | s0/fp | Saved register / frame pointer / 保存寄存器/帧指针 | Callee |
| x9 | s1 | Saved register / 保存寄存器 | Callee |
| x10-x11 | a0-a1 | Function arguments / return values / 函数参数/返回值 | Caller |
| x12-x17 | a2-a7 | Function arguments / 函数参数 | Caller |
| x18-x27 | s2-s11 | Saved registers / 保存寄存器 | Callee |
| x28-x31 | t3-t6 | Temporaries / 临时寄存器 | Caller |

---

### x0 (zero register) / 零寄存器

Always reads as zero; writes are ignored. Enables useful idioms:

始终读取为零；写入被忽略。支持有用的惯用法：

```assembly
mv   rd, rs      # Pseudo: addi rd, rs, 0 (copy register)
li   rd, imm     # Pseudo: addi rd, x0, imm (load immediate)
nop              # Pseudo: addi x0, x0, 0 (no operation)
ret              # Pseudo: jalr x0, 0(x1) (return)
```

---

### ABI Register Names / ABI 寄存器名称

Application Binary Interface names for registers, standardizing usage across compilers:

寄存器的应用程序二进制接口名称，标准化编译器间的使用：

**Caller-saved (volatile) / 调用者保存（易失）：**
- Temporaries: t0-t6 / 临时：t0-t6
- Arguments: a0-a7 / 参数：a0-a7
- Return address: ra / 返回地址：ra

**Callee-saved (preserved) / 被调用者保存（保留）：**
- Saved registers: s0-s11 / 保存寄存器：s0-s11
- Stack pointer: sp / 栈指针：sp

---

### PC (Program Counter) / 程序计数器

Holds the address of the current instruction. Not directly accessible as a general register.

保存当前指令的地址。不能直接作为通用寄存器访问。

**PC manipulation / PC 操作：**
- JAL/JALR: Save PC+4 to rd, jump to target / 保存 PC+4 到 rd，跳转到目标
- Branch instructions: Conditionally add offset to PC / 分支指令：有条件地向 PC 添加偏移量
- AUIPC: Add 20-bit immediate to PC / 向 PC 添加 20 位立即数

---

## Instruction Formats / 指令格式

RISC-V uses six base instruction formats, all 32 bits wide (except C extension):

RISC-V 使用六种基本指令格式，全部 32 位宽（C 扩展除外）：

### R-Type (Register-Register) / R 类型（寄存器-寄存器）

```
31        25 24  20 19  15 14   12 11   7 6     0
[  funct7  ][  rs2 ][  rs1 ][ funct3][  rd  ][ opcode ]
```

**Used for / 用于：**
- Arithmetic: ADD, SUB, AND, OR, XOR / 算术：ADD、SUB、AND、OR、XOR
- Shifts: SLL, SRL, SRA / 移位：SLL、SRL、SRA
- Comparisons: SLT, SLTU / 比较：SLT、SLTU

**Example / 示例：** `add x5, x6, x7  # x5 = x6 + x7`

---

### I-Type (Immediate) / I 类型（立即数）

```
31             20 19  15 14   12 11   7 6     0
[    imm[11:0]   ][  rs1 ][ funct3][  rd  ][ opcode ]
```

**Used for / 用于：**
- Immediate arithmetic: ADDI, SLTI, XORI, ORI, ANDI / 立即数算术：ADDI、SLTI、XORI、ORI、ANDI
- Loads: LB, LH, LW, LBU, LHU (RV32I) / 加载：LB、LH、LW、LBU、LHU（RV32I）
- JALR: Jump and link register / 跳转并链接寄存器
- System instructions: ECALL, EBREAK / 系统指令：ECALL、EBREAK

**Immediate encoding / 立即数编码：** Sign-extended 12-bit immediate / 符号扩展的 12 位立即数

**Example / 示例：** `addi x5, x6, 10  # x5 = x6 + 10`

---

### S-Type (Store) / S 类型（存储）

```
31        25 24  20 19  15 14   12 11      7 6     0
[imm[11:5]][  rs2 ][  rs1 ][ funct3][imm[4:0]][ opcode ]
```

**Used for / 用于：**
- Store instructions: SB, SH, SW / 存储指令：SB、SH、SW

**Immediate encoding / 立即数编码：** 12-bit offset split into two fields / 12 位偏移量分为两个字段

**Example / 示例：** `sw x5, 12(x6)  # mem[x6+12] = x5`

---

### B-Type (Branch) / B 类型（分支）

```
31 30      25 24  20 19  15 14   12 11      8  7 6     0
[imm[12]][imm[10:5]][  rs2 ][  rs1 ][ funct3][imm[4:1]][imm[11]][ opcode ]
```

**Used for / 用于：**
- Conditional branches: BEQ, BNE, BLT, BGE, BLTU, BGEU / 条件分支：BEQ、BNE、BLT、BGE、BLTU、BGEU

**Immediate encoding / 立即数编码：** 
- 13-bit signed offset (imm[12:0]) / 13 位有符号偏移量（imm[12:0]）
- imm[0] is implicitly 0 (2-byte aligned) / imm[0] 隐含为 0（2 字节对齐）
- Branch range: ±4 KiB / 分支范围：±4 KiB

**Example / 示例：** `beq x5, x6, label  # if (x5 == x6) goto label`

---

### U-Type (Upper Immediate) / U 类型（高位立即数）

```
31                              12 11   7 6     0
[         imm[31:12]              ][  rd  ][ opcode ]
```

**Used for / 用于：**
- LUI: Load upper immediate / 加载高位立即数
- AUIPC: Add upper immediate to PC / 向 PC 添加高位立即数

**Immediate encoding / 立即数编码：** 20-bit immediate placed in upper 20 bits of register / 20 位立即数放置在寄存器的高 20 位

**Example / 示例：** `lui x5, 0x12345  # x5 = 0x12345000`

---

### J-Type (Jump) / J 类型（跳转）

```
31 30          21 20 19            12 11   7 6     0
[imm[20]][imm[10:1]][imm[11]][imm[19:12]][  rd  ][ opcode ]
```

**Used for / 用于：**
- JAL: Jump and link / 跳转并链接

**Immediate encoding / 立即数编码：** 
- 21-bit signed offset (imm[20:0]) / 21 位有符号偏移量（imm[20:0]）
- imm[0] is implicitly 0 (2-byte aligned) / imm[0] 隐含为 0（2 字节对齐）
- Jump range: ±1 MiB / 跳转范围：±1 MiB

**Example / 示例：** `jal x1, function  # x1 = PC+4; goto function`

---

## Instruction Categories / 指令类别

### Integer Computational Instructions / 整数计算指令

**Arithmetic / 算术：**
- `ADD`, `SUB`: Addition and subtraction / 加法和减法
- `ADDI`: Add immediate / 加立即数
- `LUI`: Load upper immediate (imm << 12) / 加载高位立即数（imm << 12）
- `AUIPC`: Add upper immediate to PC / 向 PC 加高位立即数

**Logical / 逻辑：**
- `AND`, `OR`, `XOR`: Bitwise operations / 按位操作
- `ANDI`, `ORI`, `XORI`: Immediate variants / 立即数变体

**Shift / 移位：**
- `SLL`, `SRL`, `SRA`: Shift left logical, right logical, right arithmetic / 逻辑左移、逻辑右移、算术右移
- `SLLI`, `SRLI`, `SRAI`: Immediate variants / 立即数变体

**Comparison / 比较：**
- `SLT`, `SLTU`: Set less than (signed/unsigned) / 小于则置位（有符号/无符号）
- `SLTI`, `SLTIU`: Immediate variants / 立即数变体

---

### Load and Store Instructions / 加载和存储指令

**Load instructions / 加载指令：**
```
LB   rd, offset(rs1)  # Load byte (sign-extend)
LH   rd, offset(rs1)  # Load halfword (sign-extend)
LW   rd, offset(rs1)  # Load word
LBU  rd, offset(rs1)  # Load byte (zero-extend)
LHU  rd, offset(rs1)  # Load halfword (zero-extend)
```

**Store instructions / 存储指令：**
```
SB  rs2, offset(rs1)  # Store byte
SH  rs2, offset(rs1)  # Store halfword
SW  rs2, offset(rs1)  # Store word
```

**Addressing / 寻址：**
- Base + offset: `effective_address = rs1 + sign_extend(offset)` / 基址 + 偏移量
- Offset range: -2048 to +2047 (12-bit signed) / 偏移量范围：-2048 到 +2047（12 位有符号）

**Alignment requirements / 对齐要求：**
- LH/SH: 2-byte aligned / 2 字节对齐
- LW/SW: 4-byte aligned / 4 字节对齐
- Misaligned access triggers exception (unless Zicsr) / 非对齐访问触发异常（除非 Zicsr）

---

### Control Transfer Instructions / 控制转移指令

**Unconditional jumps / 无条件跳转：**
```
JAL   rd, offset      # rd = PC+4; PC += sign_extend(offset)
JALR  rd, offset(rs1) # rd = PC+4; PC = (rs1 + sign_extend(offset)) & ~1
```

**Conditional branches / 条件分支：**
```
BEQ   rs1, rs2, offset  # if (rs1 == rs2) PC += sign_extend(offset)
BNE   rs1, rs2, offset  # if (rs1 != rs2) PC += sign_extend(offset)
BLT   rs1, rs2, offset  # if (rs1 < rs2) PC += sign_extend(offset) [signed]
BGE   rs1, rs2, offset  # if (rs1 >= rs2) PC += sign_extend(offset) [signed]
BLTU  rs1, rs2, offset  # if (rs1 < rs2) PC += sign_extend(offset) [unsigned]
BGEU  rs1, rs2, offset  # if (rs1 >= rs2) PC += sign_extend(offset) [unsigned]
```

**Return / 返回：**
```
RET  # Pseudo: jalr x0, 0(ra)  # PC = ra
```

---

### System Instructions / 系统指令

**Environment calls / 环境调用：**
```
ECALL   # Environment call (system call or SBI call)
EBREAK  # Debugger breakpoint
```

**Memory ordering / 内存顺序：**
```
FENCE   predecessor, successor  # Memory fence
FENCE.I  # Instruction fence (sync I-cache)
```

---

## Pseudo-Instructions / 伪指令

Pseudo-instructions are assembler conveniences that expand to actual instructions:

伪指令是汇编器便利功能，扩展为实际指令：

| Pseudo-instruction / 伪指令 | Base instruction(s) / 基础指令 | Meaning / 含义 |
|------------------|----------------------|---------------|
| `nop` | `addi x0, x0, 0` | No operation / 无操作 |
| `li rd, imm` | `lui` + `addi` | Load immediate / 加载立即数 |
| `mv rd, rs` | `addi rd, rs, 0` | Copy register / 复制寄存器 |
| `not rd, rs` | `xori rd, rs, -1` | Bitwise NOT / 按位非 |
| `neg rd, rs` | `sub rd, x0, rs` | Negate / 取反 |
| `seqz rd, rs` | `sltiu rd, rs, 1` | Set if equal to zero / 等于零则置位 |
| `snez rd, rs` | `sltu rd, x0, rs` | Set if not equal to zero / 不等于零则置位 |
| `sltz rd, rs` | `slt rd, rs, x0` | Set if less than zero / 小于零则置位 |
| `sgtz rd, rs` | `slt rd, x0, rs` | Set if greater than zero / 大于零则置位 |
| `beqz rs, offset` | `beq rs, x0, offset` | Branch if equal to zero / 等于零则分支 |
| `bnez rs, offset` | `bne rs, x0, offset` | Branch if not equal to zero / 不等于零则分支 |
| `j offset` | `jal x0, offset` | Jump / 跳转 |
| `jr rs` | `jalr x0, 0(rs)` | Jump register / 跳转寄存器 |
| `ret` | `jalr x0, 0(ra)` | Return from subroutine / 从子程序返回 |
| `call offset` | `auipc` + `jalr` | Call far function / 调用远函数 |
| `tail offset` | `auipc` + `jalr x0` | Tail call / 尾调用 |

---

## Addressing Modes / 寻址模式

RISC-V is a load-store architecture with simple addressing modes:

RISC-V 是加载-存储架构，具有简单的寻址模式：

**Register / 寄存器：**
```
add x5, x6, x7  # x5 = x6 + x7
```

**Immediate / 立即数：**
```
addi x5, x6, 100  # x5 = x6 + 100
```

**Base + Displacement / 基址 + 偏移：**
```
lw x5, 12(x6)  # x5 = mem[x6 + 12]
sw x5, 8(x6)   # mem[x6 + 8] = x5
```

**PC-relative / PC 相对：**
```
auipc x5, 0x1000  # x5 = PC + (0x1000 << 12)
jal x1, label     # x1 = PC+4; PC += offset
```

**No complex addressing modes / 无复杂寻址模式：**
- No auto-increment/decrement / 无自动增减
- No register indirect with index / 无寄存器间接加索引
- No memory-to-memory operations / 无内存到内存操作

---

## Calling Convention / 调用约定

RISC-V standard calling convention (defined in psABI):

RISC-V 标准调用约定（在 psABI 中定义）：

### Function Arguments / 函数参数

**Integer arguments / 整数参数：**
- First 8 arguments: a0-a7 (x10-x17) / 前 8 个参数：a0-a7（x10-x17）
- Additional arguments: passed on stack / 额外参数：通过栈传递

**Return values / 返回值：**
- a0-a1 (x10-x11) / a0-a1（x10-x11）

**Floating-point arguments / 浮点参数：**
- First 8 FP arguments: fa0-fa7 (f10-f17) / 前 8 个浮点参数：fa0-fa7（f10-f17）
- Additional: passed on stack / 额外：通过栈传递

---

### Register Preservation / 寄存器保存

**Caller-saved (volatile) / 调用者保存（易失）：**
- Temporaries: t0-t6 / 临时：t0-t6
- Arguments: a0-a7 / 参数：a0-a7
- Return address: ra / 返回地址：ra

**Callee-saved (non-volatile) / 被调用者保存（非易失）：**
- Saved registers: s0-s11 / 保存寄存器：s0-s11
- Stack pointer: sp / 栈指针：sp

**Function prologue/epilogue / 函数序言/尾声：**
```assembly
function:
    # Prologue
    addi sp, sp, -16    # Allocate stack frame
    sw   ra, 12(sp)     # Save return address
    sw   s0, 8(sp)      # Save s0
    
    # Function body
    ...
    
    # Epilogue
    lw   s0, 8(sp)      # Restore s0
    lw   ra, 12(sp)     # Restore return address
    addi sp, sp, 16     # Deallocate stack frame
    ret                 # Return
```

---

## Memory Model / 内存模型

### RVWMO (RISC-V Weak Memory Ordering) / RISC-V 弱内存顺序

RISC-V's default memory consistency model:

RISC-V 的默认内存一致性模型：

**Key properties / 关键特性：**
- Load-load, load-store, store-store reordering allowed / 允许加载-加载、加载-存储、存储-存储重排序
- Stores are not atomic by default / 存储默认不是原子的
- Each hart observes its own memory ops in program order / 每个 hart 以程序顺序观察其自己的内存操作
- Ordering between harts requires explicit fences or atomics / hart 间的顺序需要显式屏障或原子操作

**FENCE instruction / FENCE 指令：**
```
FENCE pred, succ
```
- `pred`: Predecessor operations (r/w/rw) / 前序操作（r/w/rw）
- `succ`: Successor operations (r/w/rw) / 后续操作（r/w/rw）

**Example / 示例：**
```assembly
FENCE rw, rw  # Full memory barrier
FENCE w, rw   # Store-load barrier
FENCE r, r    # Load-load barrier
```

---

### Little-Endian / 小端

RISC-V uses little-endian byte ordering:
- Least significant byte at lowest address / 最低有效字节在最低地址
- Multi-byte values stored with LSB first / 多字节值 LSB 优先存储

**Example / 示例：**
```
32-bit value 0x12345678 at address 0x100:
Address:  0x100  0x101  0x102  0x103
Value:    0x78   0x56   0x34   0x12
```

---

## Immediate Encoding / 立即数编码

RISC-V's immediate encoding design principles:

RISC-V 的立即数编码设计原则：

**Sign bit always at bit 31 / 符号位始终在位 31：**
- Simplifies sign-extension logic / 简化符号扩展逻辑
- Consistent across all formats / 所有格式一致

**Immediate bits gathered from fixed positions / 立即数位从固定位置收集：**
- Reduces critical path in decode stage / 减少译码阶段的关键路径
- rd and rs fields always at same positions / rd 和 rs 字段始终在相同位置

**Building large constants / 构建大常量：**
```assembly
# Load 32-bit constant 0x12345678
lui  t0, 0x12346      # t0 = 0x12346000
addi t0, t0, 0x678    # t0 = 0x12345678

# Or using pseudo-instruction
li   t0, 0x12345678   # Assembler expands to lui + addi
```

---

## Opcode Map / 操作码映射

RISC-V base opcode space (bits 6:0):

RISC-V 基础操作码空间（位 6:0）：

| Opcode | Instruction Type / 指令类型 |
|--------|---------------------------|
| 0000011 | LOAD (LB, LH, LW, LBU, LHU) |
| 0001111 | MISC-MEM (FENCE, FENCE.I) |
| 0010011 | OP-IMM (ADDI, SLTI, ANDI, etc.) |
| 0010111 | AUIPC |
| 0100011 | STORE (SB, SH, SW) |
| 0110011 | OP (ADD, SUB, SLT, AND, OR, XOR, SLL, SRL, SRA) |
| 0110111 | LUI |
| 1100011 | BRANCH (BEQ, BNE, BLT, BGE, BLTU, BGEU) |
| 1100111 | JALR |
| 1101111 | JAL |
| 1110011 | SYSTEM (ECALL, EBREAK, CSR ops) |

**Extension opcodes / 扩展操作码：**
- 0101111: AMO (A extension atomics) / AMO（A 扩展原子操作）
- 0000111, 0100111: FP loads/stores (F/D extensions) / 浮点加载/存储（F/D 扩展）
- 1010111: OP-V (V extension vector ops) / OP-V（V 扩展向量操作）

---

**Last Updated / 最后更新：** 2026-07-23
