---
tags: [riscv, extensions, vocabulary]
domain: extensions
last_updated: 2026-07-22
source: https://github.com/riscv/riscv-isa-manual
---

# RISC-V Standard Extensions Vocabulary / RISC-V 标准扩展词汇表

> Covers: M/A/F/D/C/V/B/H/Zicsr and other ratified extensions
> Source: https://riscv.org/technical/specifications/ · Ratified Extensions

> 涵盖：M/A/F/D/C/V/B/H/Zicsr 等已批准扩展
> 来源：https://riscv.org/technical/specifications/ · Ratified Extensions

---

## Integer Computation Extensions / 整数计算扩展

### M Extension (Integer Multiply-Divide) / M Extension（整数乘除法）

Adds multiplication and division instructions to the base integer ISA:

| Instruction | Operation | Description |
|------|------|------|
| `MUL` | rd = rs1 × rs2 (low XLEN bits) | Signed×signed, low bits |
| `MULH` | rd = (rs1 × rs2) >> XLEN | Signed×signed, high bits |
| `MULHU` | Same, unsigned×unsigned | |
| `MULHSU` | rs1 signed × rs2 unsigned, high bits | For large integer multiplication |
| `DIV` / `REM` | Signed division/remainder | |
| `DIVU` / `REMU` | Unsigned division/remainder | |
| `MULW/DIVW/REMW` | 32-bit operations (RV64) | Result sign-extended to 64 bits |

Division result conventions: divide-by-zero → quotient all-1s, remainder=dividend; overflow (`-2^(XLEN-1) / -1`) → quotient=-2^(XLEN-1), remainder=0.

为基础整数 ISA 增加乘法和除法指令：

| 指令 | 操作 | 说明 |
|------|------|------|
| `MUL` | rd = rs1 × rs2（低XLEN位） | 有符号×有符号，取低位 |
| `MULH` | rd = (rs1 × rs2) >> XLEN | 有符号×有符号，取高位 |
| `MULHU` | 同上，无符号×无符号 | |
| `MULHSU` | rs1有符号 × rs2无符号，取高位 | 用于大整数乘法 |
| `DIV` / `REM` | 有符号除法/取余 | |
| `DIVU` / `REMU` | 无符号除法/取余 | |
| `MULW/DIVW/REMW` | 32位操作（RV64） | 结果符号扩展到64位 |

除法结果约定：除以零 → 商全1，余数=被除数；溢出（`-2^(XLEN-1) / -1`）→ 商=-2^(XLEN-1)，余数=0。

### B Extension (Bit Manipulation) / B Extension（位操作）

Combination of three sub-extensions (all individually ratified):
- **Zba**: Address generation (`sh1add`/`sh2add`/`sh3add`, shift-left 1/2/3 then add)
- **Zbb**: Basic bit manipulation (`clz`/`ctz`/`cpop`/`min`/`max`/`andn`/`orn`/`xnor`/`zext.h`/`rol`/`ror`)
- **Zbc**: Carry-less multiplication (`clmul`/`clmulh`/`clmulr`, for CRC and GCM acceleration)
- **Zbs**: Single-bit operations (`bset`/`bclr`/`binv`/`bext`, bit-indexed operations)

三个子扩展的组合（全部已单独批准）：
- **Zba**：地址生成（`sh1add`/`sh2add`/`sh3add`，左移1/2/3位后加）
- **Zbb**：基础位操作（`clz`/`ctz`/`cpop`/`min`/`max`/`andn`/`orn`/`xnor`/`zext.h`/`rol`/`ror`）
- **Zbc**：无进位乘法（`clmul`/`clmulh`/`clmulr`，用于 CRC 和 GCM 加速）
- **Zbs**：单位操作（`bset`/`bclr`/`binv`/`bext`，按位索引操作）

---

## Floating-Point Extensions / 浮点扩展

### F Extension (Single-Precision Floating-Point) / F Extension（单精度浮点）

Introduces 32 32-bit floating-point registers (f0–f31) and IEEE 754-2008 single-precision floating-point instructions. Includes: add/sub/mul/div, square root, fused multiply-add (FMA), comparison, conversion, load/store.

引入 32 个 32 位浮点寄存器（f0–f31）和 IEEE 754-2008 单精度浮点指令。包含：加减乘除、平方根、融合乘加（FMA）、比较、转换、Load/Store。

### D Extension (Double-Precision Floating-Point) / D Extension（双精度浮点）

Builds on F Extension by extending f0–f31 to 64 bits, supporting double-precision IEEE 754 operations. Includes F Extension (D Extension implementations must also implement F).

在 F 扩展基础上，将 f0–f31 扩展为 64 位，支持双精度 IEEE 754 操作。包含 F 扩展（D 扩展实现必须同时实现 F）。

### Q Extension (Quad-Precision Floating-Point) / Q Extension（四精度浮点）

128-bit quad-precision floating-point, rarely implemented, primarily for high-precision numerical computing. Includes D Extension.

128 位四精度浮点，稀少实现，主要用于高精度数值计算。包含 D 扩展。

### FMA (Fused Multiply-Add) / 融合乘加

Single instruction that completes `rd = rs1 × rs2 + rs3` with only one rounding, providing higher precision than separate multiply and add operations. RISC-V provides 4 variants: `FMADD`/`FMSUB`/`FNMADD`/`FNMSUB`.

单指令完成 `rd = rs1 × rs2 + rs3`，只做一次舍入，比分开的乘法和加法精度更高。RISC-V 提供 4 种变体：`FMADD`/`FMSUB`/`FNMADD`/`FNMSUB`。

### rounding mode / 舍入模式

Rounding method for floating-point operation results, specified via the instruction's `rm` field or the `fcsr` register's `frm` field:
- **RNE** (Round to Nearest, ties to Even): Nearest even, default
- **RTZ** (Round toward Zero): Truncate
- **RDN** (Round Down): Toward negative infinity
- **RUP** (Round Up): Toward positive infinity
- **RMM** (Round to Nearest, ties to Max Magnitude): Nearest, ties to maximum magnitude

浮点运算的结果舍入方式，通过指令的 `rm` 字段或 `fcsr` 寄存器的 `frm` 字段指定：
- **RNE**（Round to Nearest, ties to Even）：最近偶数，默认
- **RTZ**（Round toward Zero）：截断
- **RDN**（Round Down）：向负无穷
- **RUP**（Round Up）：向正无穷
- **RMM**（Round to Nearest, ties to Max Magnitude）：最近，等距取绝对值大的

### fcsr (Floating-Point Control and Status Register) / 浮点控制与状态寄存器

Contains rounding mode (frm) and exception flags (fflags, recording invalid operation/divide-by-zero/overflow/underflow/inexact).

包含舍入模式（frm）和异常标志（fflags，记录无效操作/除零/溢出/下溢/不精确）。

### Zfinx / Zdinx / 浮点使用整数寄存器

Does not introduce separate floating-point register file; floating-point operations directly use integer x registers, reducing register file area, suitable for microcontrollers. `Zfinx` for single-precision, `Zdinx` for double-precision.

不引入独立浮点寄存器堆，浮点操作直接使用整数 x 寄存器，减少寄存器文件面积，适合微控制器。`Zfinx` 对应单精度，`Zdinx` 对应双精度。

---

## Atomic Operations Extension (A Extension) / 原子操作扩展（A Extension）

### A Extension / 原子操作扩展

→ Core terms detailed in [[01-isa基础/vocabulary]] (AMO, LR/SC, acquire/release semantics)

**Instruction list:**
- `LR.W/D`, `SC.W/D`: Load-reserved/store-conditional
- `AMOSWAP`: Atomic swap
- `AMOADD`: Atomic add
- `AMOAND/AMOOR/AMOXOR`: Atomic bit operations
- `AMOMIN/AMOMAX/AMOMINU/AMOMAXU`: Atomic min/max

All AMO instructions support `.aq` (acquire), `.rl` (release), `.aqrl` (sequentially consistent) modifiers.

→ 核心词条详见 [[01-isa基础/vocabulary]]（AMO、LR/SC、acquire/release 语义）

**指令列表：**
- `LR.W/D`，`SC.W/D`：加载保留/条件存储
- `AMOSWAP`：原子交换
- `AMOADD`：原子加
- `AMOAND/AMOOR/AMOXOR`：原子位操作
- `AMOMIN/AMOMAX/AMOMINU/AMOMAXU`：原子最小/最大值

所有 AMO 指令支持 `.aq`（acquire）、`.rl`（release）、`.aqrl`（顺序一致）修饰符。

---

## Compressed Instruction Extension (C Extension) / 压缩指令扩展（C Extension）

### C Extension / 压缩指令扩展

→ Core terms detailed in [[01-isa基础/vocabulary]] (compressed instruction, CI/CR formats, etc.)

Key design: Compressed instructions use `x8–x15` (s0-s1, a0-a5) as the 8 "most commonly used registers", requiring only 3 bits for addressing in encoding. Code size reduced by approximately 25–30%, providing significant benefits for embedded systems.

→ 核心词条详见 [[01-isa基础/vocabulary]]（compressed instruction、CI/CR 等格式）

关键设计：压缩指令使用 `x8–x15`（s0-s1, a0-a5）这 8 个"最常用寄存器"，编码时只需 3 位来寻址。代码体积减少约 25–30%，对嵌入式系统有显著收益。

---

## Vector Extension (V Extension) / 向量扩展（V Extension）

### V Extension (Vector Extension 1.0) / V Extension（向量扩展 1.0）

Adds SIMD vector operation capabilities to RISC-V, a standard extension for application processor profiles. Ratified in 2021 (v1.0).

**Prerequisites**: Depends on Zvl128b (minimum VLEN=128) and Zve64d (includes F/D floating-point extensions). Supports EEW 8/16/32/64, all vector integer/floating-point/mask/permutation instructions.

为 RISC-V 添加 SIMD 向量操作能力，应用处理器配置文件的标准扩展。已于 2021 年批准（v1.0）。

**前置要求**：依赖 Zvl128b（最小 VLEN=128）和 Zve64d（含 F/D 浮点扩展）。支持 EEW 8/16/32/64，所有向量整数/浮点/mask/置换指令。

### VLEN / VLEN

Bit width of vector registers, implementation-defined (power of 2, ≥8, ≤2^16). Specification requires ELEN ≤ VLEN (future extensions may relax this). VLEN upper limit of 65536 bits ensures vl index can be represented with 16 bits (VLMAX_max = VLEN, indices 0…VLEN-1).

向量寄存器的位宽，实现定义（2 的幂，≥8，≤2^16）。规范要求 ELEN ≤ VLEN（未来扩展可能放宽）。VLEN 上限 65536 位确保 vl 索引可用 16 位表示（VLMAX_max = VLEN，索引 0…VLEN-1）。

### ELEN / ELEN

Maximum bit width of a single element in vector registers (power of 2, minimum 8 bits). V Extension requires ELEN=64.

向量寄存器中单个元素的最大位宽（2 的幂，最小 8 位）。V 扩展要求 ELEN=64。

### v0–v31 / 向量寄存器

32 vector registers, each fixed at VLEN bits. Completely separate from scalar integer/floating-point registers.

32 个向量寄存器，每个固定 VLEN 位。与标量整数/浮点寄存器完全分离。

### vtype CSR / vtype 寄存器

Vector type CSR, read-only (XLEN bits wide), can only be updated by vsetvli/vsetivli/vsetvl instructions:

| Field | Bits | Meaning |
|------|------|------|
| **vill** | XLEN-1 | =1 indicates illegal configuration |
| **vma** | 7 | Vector Mask Agnostic (mask element handling policy) |
| **vta** | 6 | Vector Tail Agnostic (tail element handling policy) |
| **vsew** | 5:3 | Selected Element Width (000=8, 001=16, 010=32, 011=64) |
| **vlmul** | 2:0 | LMUL signed encoding (2^vlmul = LMUL) |

向量类型 CSR，只读（XLEN 位宽），只能由 vsetvli/vsetivli/vsetvl 指令更新：

| 字段 | 位域 | 含义 |
|------|------|------|
| **vill** | XLEN-1 | =1 表示非法配置 |
| **vma** | 7 | Vector Mask Agnostic（掩码元素处理策略）|
| **vta** | 6 | Vector Tail Agnostic（尾部元素处理策略）|
| **vsew** | 5:3 | Selected Element Width（000=8,001=16,010=32,011=64）|
| **vlmul** | 2:0 | LMUL 有符号编码（2^vlmul = LMUL）|

### vl CSR / vl 寄存器

Read-only XLEN-bit CSR that holds the actual number of elements processed by the current vector operation. Can only be updated by vset* instructions and fault-only-first Load.

只读 XLEN 位 CSR，保存当前向量操作实际处理的元素个数。只能由 vset* 指令和 fault-only-first Load 更新。

### vlenb CSR / vlenb 寄存器

Read-only CSR, value = VLEN/8 (number of bytes in vector register). Design-time constant, avoids software manually calculating VLEN.

只读 CSR，值 = VLEN/8（向量寄存器字节数）。设计时常量，避免软件手动计算 VLEN。

### vstart CSR / vstart 寄存器

Readable/writable XLEN-bit CSR that specifies the element index where vector instruction execution begins. Only written by hardware when trap occurs (recording the trap element's index) to support resumable traps. Non-zero vstart may cause significant performance degradation; application programmers should not use it.

可读写 XLEN 位 CSR，指定向量指令开始执行的元素索引。仅在 trap 发生时由硬件写入（记录 trap 元素的索引）以支持可恢复陷阱。非零 vstart 可能导致显著性能下降，应用程序员不应使用。

### vxrm CSR / vxrm 寄存器

Vector fixed-point rounding mode register (2 bits), mirrored in vcsr[2:1]:

| Value | Abbreviation | Meaning |
|----|------|------|
| 00 | rnu | Round to nearest-up (+0.5 LSB) |
| 01 | rne | Round to nearest-even |
| 10 | rdn | Round down (truncate) |
| 11 | rod | Round to odd (OR bits into LSB) |

向量定点舍入模式寄存器（2 位），镜像在 vcsr[2:1]：

| 值 | 缩写 | 含义 |
|----|------|------|
| 00 | rnu | 向最近值舍入（+0.5 LSB）|
| 01 | rne | 向最近偶数舍入 |
| 10 | rdn | 向下舍入（截断）|
| 11 | rod | 向奇数舍入（OR bits into LSB）|

### vxsat CSR / vxsat 寄存器

Vector fixed-point saturation flag register (1 bit), mirrored in vcsr[0]. Set to 1 when fixed-point instruction saturates; software must manually clear.

向量定点饱和标志寄存器（1 位），镜像在 vcsr[0]。定点指令饱和时置 1，软件需手动清零。

### vcsr CSR / vcsr 寄存器

Vector control and status register, aggregates vxrm (bits 2:1) and vxsat (bit 0), allowing both to be set with a single csrwi instruction.

向量控制与状态寄存器，汇总 vxrm（bit 2:1）和 vxsat（bit 0），允许用单条 csrwi 指令同时设置。

---

### SEW (Selected Element Width) / 选定元素宽度

Bit width of data elements for the current operation, encoded by vtype.vsew (8/16/32/64 bits). Determines the number of elements per vector register = VLEN/SEW.

当前操作的数据元素位宽，由 vtype.vsew 编码（8/16/32/64 位）。决定每个向量寄存器包含的元素数 = VLEN/SEW。

### LMUL (Length Multiplier) / 长度倍增系数

Vector register group multiplier, encoded via vtype.vlmul signed encoding (LMUL = 2^vlmul):
- **Integer LMUL (1/2/4/8)**: Combines multiple registers into logical register groups, expanding vector width
- **Fractional LMUL (1/2, 1/4, 1/8)**: Uses only part of a register, supports mixed-width operations reducing vtype switching

**VLMAX = LMUL × VLEN / SEW**: Maximum number of elements a single vector instruction can process.

向量寄存器组倍增系数，通过 vtype.vlmul 有符号编码（LMUL = 2^vlmul）：
- **整数 LMUL（1/2/4/8）**：将多个寄存器组合为逻辑寄存器组，扩大向量宽度
- **分数 LMUL（1/2/1/4/1/8）**：仅使用寄存器的一部分，支持混合宽度操作减少 vtype 切换

**VLMAX = LMUL × VLEN / SEW**：单次向量指令最大可处理元素数。

### vector register group / 向量寄存器组

When LMUL>1, multiple consecutive vector registers are combined into a logical register group. Register numbers must be aligned (LMUL=2 requires even, LMUL=4 requires multiple of 4, LMUL=8 requires multiple of 8), otherwise reserved.

LMUL>1 时，多个连续向量寄存器组合为一个逻辑寄存器组，寄存器号必须对齐（LMUL=2 须偶数，LMUL=4 须 4 的倍数，LMUL=8 须 8 的倍数），否则 reserved。

### EEW (Effective Element Width) / 实际元素宽度

Actual element bit width of a vector operand. Most instructions have EEW=SEW, but widening/narrowing instructions have some operands with EEW≠SEW (but must satisfy EEW/EMUL = SEW/LMUL).

某个向量操作数的实际元素位宽，大多数指令 EEW=SEW，但宽化（widening）/窄化（narrowing）指令的部分操作数 EEW≠SEW（但必须满足 EEW/EMUL = SEW/LMUL）。

### EMUL (Effective LMUL) / 实际 LMUL

Actual register group size of a vector operand, EMUL = (EEW/SEW)×LMUL. Must satisfy 1/8 ≤ EMUL ≤ 8, otherwise reserved.

某个向量操作数的实际寄存器组大小，EMUL = (EEW/SEW)×LMUL，必须满足 1/8 ≤ EMUL ≤ 8 否则 reserved。

### widening operation / 宽化操作

Operations where destination register group has EEW=2×SEW, EMUL=2×LMUL (prefix `vw*`). For example, vwmul multiplies SEW-bit integers to produce 2×SEW-bit results.

目标寄存器组 EEW=2×SEW，EMUL=2×LMUL 的操作（前缀 `vw*`）。例如 vwmul 将 SEW 位整数相乘产生 2×SEW 位结果。

### narrowing operation / 窄化操作

Operations where source register group has EEW=2×SEW, destination has EEW=SEW (prefix `vn*`, source operand suffix `.w`). For example, vnsrl right-shifts 2×SEW bits then truncates to SEW bits.

源寄存器组 EEW=2×SEW，目标 EEW=SEW 的操作（前缀 `vn*`，源操作数后缀 `.w`）。例如 vnsrl 将 2×SEW 位右移后截断为 SEW 位。

---

### vta (Vector Tail Agnostic) / 向量尾部不定策略

vtype.vta field, controls behavior of tail elements (vl ≤ index < max(VLMAX, VLEN/SEW)):
- **ta (tail agnostic, vta=1)**: Implementation may leave tail elements unchanged or overwrite with all-1s (out-of-order implementations can rename registers without reading old values)
- **tu (tail undisturbed, vta=0)**: Tail elements preserve destination register's original values

**mask destination tail is always tail-agnostic, unaffected by vta.**

vtype.vta 字段，控制 tail 元素（vl ≤ index < max(VLMAX, VLEN/SEW)）的行为：
- **ta（tail agnostic，vta=1）**：实现可以将尾部元素保持不变或用全 1 覆盖（乱序实现可寄存器重命名，无需读旧值）
- **tu（tail undisturbed，vta=0）**：尾部元素保持目标寄存器原有值

**mask destination tail 始终是 tail-agnostic，不受 vta 影响。**

### vma (Vector Mask Agnostic) / 向量掩码不定策略

vtype.vma field, controls behavior of inactive elements (elements in body masked off by mask=0):
- **ma (mask agnostic, vma=1)**: inactive elements may remain unchanged or be overwritten with all-1s
- **mu (mask undisturbed, vma=0)**: inactive elements preserve destination register's original values

**Design motivation**: tail/mask agnostic policies allow out-of-order implementations to rename destination registers (without merging old physical register values into new physical registers), avoiding the overhead of reading old destination registers.

vtype.vma 字段，控制 inactive 元素（body 中被 mask=0 屏蔽的元素）的行为：
- **ma（mask agnostic，vma=1）**：inactive 元素可保持不变或用全 1 覆盖
- **mu（mask undisturbed，vma=0）**：inactive 元素保持目标寄存器原有值

**设计动机**：tail/mask agnostic 策略允许乱序实现对目标寄存器进行寄存器重命名（无需将旧物理寄存器的值合并到新物理寄存器中），避免读取旧目标寄存器的开销。

### vill (Vector Illegal) / 向量非法配置位

vtype.vill (bit XLEN-1), =1 indicates vset* wrote an unsupported vtype configuration (e.g., unsupported SEW/LMUL combination). When vill=1, executing vector instructions that depend on vtype triggers illegal instruction exception. Checking vill requires only one signed branch instruction (branch on sign bit).

vtype.vill（XLEN-1 位），=1 表示 vset* 写入了不支持的 vtype 配置（如不支持的 SEW/LMUL 组合）。vill=1 时执行依赖 vtype 的向量指令触发非法指令异常。检查 vill 只需一条带符号的分支指令（branch on sign bit）。

---

### element regions / 元素区域

During vector instruction execution, elements in destination register are divided into three disjoint regions by index:

```
prestart: 0 ≤ i < vstart   → Do not write destination, do not trigger exceptions
body:    vstart ≤ i < vl   → Divided into active (mask=1) and inactive (mask=0)
tail:      vl ≤ i < max(VLMAX, VLEN/SEW) → Follow vta policy
```

- **active** elements: May trigger exceptions, write results
- **inactive** elements: Do not trigger exceptions, follow vma policy
- **prestart** elements: Completely unaffected (supports resuming execution from vstart after precise trap)

向量指令执行期间，目标寄存器的元素按索引分为三个不相交区域：

```
prestart: 0 ≤ i < vstart   → 不写目标，不触发异常
body:    vstart ≤ i < vl   → 分为 active（mask=1）和 inactive（mask=0）
tail:      vl ≤ i < max(VLMAX, VLEN/SEW) → 遵从 vta 策略
```

- **active** 元素：可触发异常，写入结果
- **inactive** 元素：不触发异常，遵从 vma 策略
- **prestart** 元素：完全不受影响（支持精确陷阱后从 vstart 恢复执行）

---

### vsetvli / vsetivli / vsetvl / 向量配置指令

Dedicated instructions for setting vtype and vl:

```asm
vsetvli  rd, rs1, vtypei  # rd=new vl, rs1=AVL, vtypei=immediate vtype
vsetivli rd, uimm, vtypei # rd=new vl, uimm=5-bit zero-extended AVL(0-31)
vsetvl   rd, rs1, rs2     # rd=new vl, rs1=AVL, rs2=vtype register value
```

**AVL encoding rules (vsetvli/vsetvl)**:

| rd | rs1 | AVL value | vl effect |
|----|-----|-----------|----------|
| — | ≠x0 | x[rs1] | Normal strip-mining |
| ≠x0 | x0 | ~0 (max unsigned) | vl=VLMAX |
| x0 | x0 | Current vl value | Keep vl unchanged (vtype may change, but VLMAX cannot decrease) |

设置 vtype 和 vl 的专用指令：

```asm
vsetvli  rd, rs1, vtypei  # rd=新vl, rs1=AVL, vtypei=立即数vtype
vsetivli rd, uimm, vtypei # rd=新vl, uimm=5位零扩展AVL(0-31)
vsetvl   rd, rs1, rs2     # rd=新vl, rs1=AVL, rs2=vtype寄存器值
```

**AVL 编码规则（vsetvli/vsetvl）**：

| rd | rs1 | AVL 值 | vl 效果 |
|----|-----|--------|----------|
| — | ≠x0 | x[rs1] | 正常 strip-mining |
| ≠x0 | x0 | ~0（最大无符号）| vl=VLMAX |
| x0 | x0 | 当前 vl 值 | 保持 vl 不变（vtype 可变，但 VLMAX 不能减少）|

### vl setting constraints / vl 设置约束（规范精读）

`vsetvl*` first sets VLMAX, then sets vl based on AVL, following these constraints:
1. `vl = AVL` if `AVL ≤ VLMAX`
2. `⌈AVL/2⌉ ≤ vl ≤ VLMAX` if `AVL < 2×VLMAX` (implementations may use vl=⌈AVL/2⌉ to split last two iterations evenly)
3. `vl = VLMAX` if `AVL ≥ 2×VLMAX`
4. Results are deterministic for same AVL and VLMAX inputs
5. Special properties: `vl > 0` if `AVL > 0`; `vl ≤ VLMAX`; `vl ≤ AVL`

`vsetvl*` 先设置 VLMAX，再根据 AVL 设置 vl，遵从以下约束：
1. `vl = AVL` 若 `AVL ≤ VLMAX`
2. `⌈AVL/2⌉ ≤ vl ≤ VLMAX` 若 `AVL < 2×VLMAX`（实现可以 vl=⌈AVL/2⌉ 均分最后两轮迭代）
3. `vl = VLMAX` 若 `AVL ≥ 2×VLMAX`
4. 对相同 AVL 和 VLMAX 输入结果确定
5. 特殊性质：`vl > 0` 若 `AVL > 0`；`vl ≤ VLMAX`；`vl ≤ AVL`

---

### mask register / 掩码寄存器

Vector mask is fixed to be provided by v0 register, layout: mask bit for element i is at bit[i] of v0, independent of SEW and LMUL (mask register always occupies only one vector register, unaffected by LMUL).

向量 mask 固定由 v0 寄存器提供，layout 为：元素 i 的 mask 位在 v0 的 bit[i]，与 SEW 和 LMUL 无关（mask 寄存器始终只占一个向量寄存器，不受 LMUL 影响）。

### vm encoding / 掩码编码

Instruction bit[25] (vm field):
- **vm=1**: unmasked execution
- **vm=0**: masked execution with v0.mask[i]=1 as condition (`.t` suffix)

Mask inactive elements follow vma policy. **Destination register cannot overlap with v0** (except compare instructions writing mask registers and reduction instructions writing scalars).

指令 bit[25]（vm 字段）：
- **vm=1**：无掩码（unmasked）执行
- **vm=0**：以 v0.mask[i]=1 为条件的掩码执行（`.t` 后缀）

掩码 inactive 元素遵从 vma 策略。**目标寄存器不能与 v0 重叠**（比较指令写 mask 寄存器、规约指令写标量除外）。

### mask-vector logical instructions / 向量掩码逻辑指令

8 instructions that operate on entire mask registers (unaffected by LMUL, always operate on single register): `vmand`/`vmnand`/`vmandn`/`vmxor`/`vmor`/`vmnor`/`vmorn`/`vmxnor`. Can implement any of the 16 binary logic functions.

Pseudoinstructions: `vmmv.m` (copy) / `vmclr.m` (clear) / `vmset.m` (set all) / `vmnot.m` (invert).

操作整个 mask 寄存器（不受 LMUL 影响，始终操作单个寄存器）的 8 条指令：`vmand`/`vmnand`/`vmandn`/`vmxor`/`vmor`/`vmnor`/`vmorn`/`vmxnor`。可实现 16 种二元逻辑函数中的任意一种。

伪指令：`vmmv.m`（复制）/`vmclr.m`（清零）/`vmset.m`（全置位）/`vmnot.m`（取反）。

---

### fault-only-first loads / 仅首元素报错 Load

`vle8ff.v` and other ff-suffix instructions: Execute the same as normal Load, but only synchronous exceptions triggered by element 0 will cause a trap; if subsequent elements encounter exceptions, vl is shortened to the index of the first exception element (no trap).

**Use case**: Vectorize loops with data-dependent exit conditions (while loops), such as `strlen`:

```asm
strlen:
    vsetvli t1, x0, e8, m8, ta, mu  # VLMAX byte elements
loop:
    vle8ff.v v8, (a0)               # Load bytes, fault-only-first
    csrr t1, vl                     # Get actual load count
    vmseq.vi v1, v8, 0              # Find null byte
    vfirst.m a2, v1                 # Find first null
    add a0, a0, t1                  # Move pointer
    bltz a2, loop                   # Continue if not found
    add a0, a0, a2                  # Calculate null position
    sub a0, a0, t1
```

`vle8ff.v` 等 ff 后缀指令：执行与普通 Load 相同，但只有 element 0 触发的同步异常才会引发 trap，后续元素若出现异常则将 vl 缩短到首个异常元素的索引（不 trap）。

**用途**：向量化带数据相关退出条件的循环（while 循环），如 `strlen`：

```asm
strlen:
    vsetvli t1, x0, e8, m8, ta, mu  # VLMAX 字节元素
loop:
    vle8ff.v v8, (a0)               # 加载字节，fault-only-first
    csrr t1, vl                     # 获取实际加载数
    vmseq.vi v1, v8, 0              # 找 null 字节
    vfirst.m a2, v1                 # 找第一个 null
    add a0, a0, t1                  # 移动指针
    bltz a2, loop                   # 未找到继续
    add a0, a0, a2                  # 计算 null 位置
    sub a0, a0, t1
```

### segment load/store / 段 Load/Store

`vlseg*e*.v`/`vsseg*e*.v`: Unpacks tightly-packed structures (segments) from memory into multiple consecutive vector registers, or packs multiple vector registers into memory. The `nf` field (1–8) specifies the number of fields per segment (NFIELDS), with constraint EMUL×NFIELDS ≤ 8.

Typical use:
```asm
vlseg3e8.v v8, (a0)  # Unpack RGB pixels: v8=R, v9=G, v10=B
vlseg2e32.v v8, (a0) # Unpack complex numbers: v8=real, v9=imaginary
```

`vlseg*e*.v`/`vsseg*e*.v`：将内存中的紧密结构体（segment）拆包到多个连续向量寄存器，或将多个向量寄存器打包存入内存。`nf` 字段（1–8）指定每个 segment 的字段数（NFIELDS），约束 EMUL×NFIELDS ≤ 8。

典型用途：
```asm
vlseg3e8.v v8, (a0)  # 解包 RGB 像素：v8=R, v9=G, v10=B
vlseg2e32.v v8, (a0) # 解包复数：v8=实部, v9=虚部
```

### whole-register load/store / 全寄存器 Load/Store

`vl1re8.v`/`vs1r.v` etc.: Do not depend on vtype and vl, directly transfer entire vector register groups (1/2/4/8 registers, NREG must be power of 2). Mainly used for context switching, register spill/restore:

```asm
vl1r.v  v3, (a0)   # Load VLEN/8 bytes to v3 (EEW encoding as hint)
vs8r.v  v8, (a1)   # Store v8–v15
```

`vl1re8.v`/`vs1r.v` 等：不依赖 vtype 和 vl，直接传输整个向量寄存器组（1/2/4/8 个寄存器，NREG 必须是 2 的幂）。主要用于上下文切换、寄存器溢出/恢复：

```asm
vl1r.v  v3, (a0)   # 加载 VLEN/8 字节到 v3（EEW 编码作为 hint）
vs8r.v  v8, (a1)   # 存储 v8–v15
```

---

### vector arithmetic instruction formats / 向量算术指令格式

Adds OP-V (0b1010111) major opcode, funct3 distinguishes operand types:

| funct3 | Type | Scalar source |
|--------|------|---------------|
| OPIVV | vector-vector | — |
| OPFVV | FP vector-vector | — |
| OPMVV | vector-vector (special) | — |
| OPIVI | vector-immediate | 5-bit sign-extended immediate |
| OPIVX | vector-scalar | x register |
| OPFVF | FP vector-scalar | f register |
| OPMVX | vector-scalar (special) | x register |
| OPCFG | vset* configuration | x/imm |

**Operand order**: vs2 (first vector operand) comes first, rs1/vs1/imm (second operand) comes second — **opposite** of scalar instruction order.

新增 OP-V（0b1010111）主操作码，funct3 区分操作数类型：

| funct3 | 类型 | 标量来源 |
|--------|------|----------|
| OPIVV | 向量-向量 | — |
| OPFVV | FP 向量-向量 | — |
| OPMVV | 向量-向量（特殊）| — |
| OPIVI | 向量-立即数 | 5位符号扩展立即数 |
| OPIVX | 向量-标量 | x 寄存器 |
| OPFVF | FP 向量-标量 | f 寄存器 |
| OPMVX | 向量-标量（特殊）| x 寄存器 |
| OPCFG | vset* 配置 | x/imm |

**操作数顺序**：vs2（第一向量操作数）在前，rs1/vs1/imm（第二操作数）在后——与标量指令顺序**相反**。

### vector integer reduction / 向量整数规约

Reduces all active elements in vector register group with scalar input (stored in vs1[0]) using binary operation, writes result to vd[0]. Scalar input always participates in reduction (unaffected by mask). Tail follows vta policy.

Common reduction instructions: `vredsum`/`vredmaxu`/`vredmax`/`vredminu`/`vredmin`/`vredand`/`vredor`/`vredxor`.

Widening versions: `vwredsumu` (zero-extend source) / `vwredsum` (sign-extend source), result width 2×SEW.

将向量寄存器组的所有 active 元素与标量输入（存于 vs1[0]）进行二元运算，结果写回 vd[0]。scalar 输入始终参与规约（不受 mask 影响）。tail 遵从 vta 策略。

常用规约指令：`vredsum`/`vredmaxu`/`vredmax`/`vredminu`/`vredmin`/`vredand`/`vredor`/`vredxor`。

宽化版本：`vwredsumu`（零扩展源）/`vwredsum`（符号扩展源），结果宽度 2×SEW。

### vector FP reduction / 向量浮点规约

- **vfredosum** (ordered reduction): Strictly sum in element order, supports compiler auto-vectorization
- **vfredusum** (unordered reduction): Allows parallel summation in reduction tree form, better performance but may produce different results
- **vfredmax/vfredmin**: IEEE 754-2019 `maximumNumber`/`minimumNumber` reduction

- **vfredosum**（有序归约）：严格按元素顺序求和，支持编译器自动向量化
- **vfredusum**（无序归约）：允许以规约树形式并行求和，性能更好但结果可能不同
- **vfredmax/vfredmin**：IEEE 754-2019 `maximumNumber`/`minimumNumber` 规约

### vector mask instructions / 向量掩码操作指令

- **vcpop.m**: Count number of active=1 elements in mask (popcount)
- **vfirst.m**: Find index of lowest active=1 element, return -1 if none
- **vmsbf.m** (set-before-first): Write 1 at positions before first 1, write 0 after
- **vmsif.m** (set-including-first): Write 1 at all positions up to and including first 1
- **vmsof.m** (set-only-first): Write 1 only at first 1 position, write 0 elsewhere
- **viota.m**: For each element i, write count of 1s in mask[0..i-1] (parallel prefix sum)
- **vid.v**: For each element, write its index (0, 1, 2, ...)

- **vcpop.m**：计算 mask 中 active=1 的元素数（popcount）
- **vfirst.m**：找最低 active=1 元素的索引，无则返回 -1
- **vmsbf.m**（set-before-first）：在首个 1 之前的位置写 1，其后写 0
- **vmsif.m**（set-including-first）：包含首个 1 的所有前置位置写 1
- **vmsof.m**（set-only-first）：只有首个 1 位置写 1，其余写 0
- **viota.m**：对每个元素 i 写出 mask[0..i-1] 中 1 的个数（并行前缀和）
- **vid.v**：对每个元素写出其索引（0, 1, 2, ...）

### vector compress / 向量压缩指令

`vcompress.vm vd, vs2, vs1`: Packs elements from vs2 where vs1 mask=1 into consecutive positions at start of vd. Similar to scalar `pext`, used for implementing data-dependent filtering:

```asm
vcompress.vm v2, v1, v0  # Pack elements from v1 where v0=1 in order to v2
```

Reverse operation (vdecompress) implemented through viota + vrgather combination.

`vcompress.vm vd, vs2, vs1`：将 vs2 中 vs1 mask=1 的元素打包到 vd 的前连续位置。类似标量的 `pext`，用于实现数据相关过滤：

```asm
vcompress.vm v2, v1, v0  # 将 v1 中 v0=1 的元素按序打包到 v2
```

反向操作（vdecompress）通过 viota + vrgather 组合实现。

### vector permutation instructions / 向量置换指令

- **vrgather.vv**: Gather from vs2 using vs1[i] as index, return 0 for out-of-range
- **vrgatherei16.vv**: Same but index fixed to EEW=16, supports larger datasets
- **vslideup/vslidedown**: Slide elements up/down, supports register or immediate offset
- **vslide1up/vslide1down**: Slide only 1 position, can insert scalar value
- **vcompress**: → See above

- **vrgather.vv**：按 vs1[i] 作为索引从 vs2 中收集，超范围返回 0
- **vrgatherei16.vv**：同上但索引固定为 EEW=16，支持更大数据集
- **vslideup/vslidedown**：向上/向下滑动元素，支持寄存器或立即数偏移量
- **vslide1up/vslide1down**：只滑动 1 位，可插入标量值
- **vcompress**：→ 见上

### vmv instructions / 向量 move 指令

- **vmv.v.v/x/i**: Splat vector/scalar/immediate to vector (unmasked)
- **vmv.x.s/vmv.s.x**: Transfer value between scalar and vector element 0
- **vmv<nr>r.v**: Whole-register move (1/2/4/8 registers), independent of vtype/vl

- **vmv.v.v/x/i**：splat 向量/标量/立即数到向量（无掩码）
- **vmv.x.s/vmv.s.x**：标量与向量 element 0 之间传值
- **vmv<nr>r.v**：全寄存器 move（1/2/4/8 寄存器），不依赖 vtype/vl

---

### fixed-point operations / 向量定点运算

- **vsadd/vsaddu/vssub/vssubu**: Saturating add/subtract, clamp to target format boundary on overflow and set vxsat
- **vaaddu/vaadd/vasubu/vasub**: Averaging add/subtract, right shift 1 bit with rounding per vxrm
- **vsmul**: Signed fractional multiply, result right-shifted SEW-1 bits with saturation (directly analogous to Q15/Q31 format in digital signal processing)
- **vssrl/vssra**: Scaling shift right, with rounding
- **vnclipu/vnclip**: Narrowing clip, clips 2×SEW source to SEW destination, with rounding and saturation

- **vsadd/vsaddu/vssub/vssubu**：饱和加减，溢出时截断到目标格式边界并设 vxsat
- **vaaddu/vaadd/vasubu/vasub**：平均加减，右移 1 位并按 vxrm 舍入
- **vsmul**：有符号分数乘，结果右移 SEW-1 位并饱和（直接类比数字信号处理中的 Q15/Q31 格式）
- **vssrl/vssra**：缩放右移（scaling shift right），带舍入
- **vnclipu/vnclip**：窄化裁剪，将 2×SEW 源裁剪到 SEW 目标，带舍入和饱和

### element group instructions / 元素组指令

Some vector instructions (such as SHA-256 instructions in cryptographic extension Zvkn) treat operands as fixed-size element groups (EGS, must be power of 2). Requires vl to be integer multiple of EGS (otherwise reserved), VLMAX ≥ EGS (otherwise reserved).

部分向量指令（如密码学扩展 Zvkn 的 SHA-256 指令）将操作数视为固定大小的元素组（EGS，必须为 2 的幂）。要求 vl 必须是 EGS 的整数倍（否则 reserved），VLMAX ≥ EGS（否则 reserved）。

---

### Zvl* (minimum vector length) extensions / 向量长度扩展

Extension family that standardizes minimum VLEN requirements: Zvl32b/Zvl64b/Zvl128b/Zvl256b/Zvl512b/Zvl1024b. V Extension depends on Zvl128b (minimum VLEN=128).

标准化最小 VLEN 要求的扩展族：Zvl32b/Zvl64b/Zvl128b/Zvl256b/Zvl512b/Zvl1024b。V 扩展依赖 Zvl128b（最小 VLEN=128）。

### Zve* (embedded vector) extensions / 嵌入式向量扩展

Vector extension subsets for embedded scenarios:
- **Zve32x**: ELEN=32, integer only, VLEN≥32
- **Zve32f**: Zve32x + FP32
- **Zve64x**: ELEN=64, integer only, VLEN≥64
- **Zve64f**: Zve64x + FP32
- **Zve64d**: Zve64f + FP64 (V Extension dependency)

面向嵌入式场景的向量扩展子集：
- **Zve32x**：ELEN=32，仅整数，VLEN≥32
- **Zve32f**：Zve32x + FP32
- **Zve64x**：ELEN=64，仅整数，VLEN≥64
- **Zve64f**：Zve64x + FP32
- **Zve64d**：Zve64f + FP64（V 扩展的依赖项）

### Zvbb / Zvbc / 向量位操作扩展

- **Zvbb**: Vector bit manipulation (brev8/roll/ror/wshfl/andn, etc.)
- **Zvbc**: Vector carry-less multiplication (clmul/clmulh, for GCM cipher acceleration)

- **Zvbb**：向量位操作（brev8/roll/ror/wshfl/andn 等）
- **Zvbc**：向量无进位乘法（clmul/clmulh，用于 GCM 密码加速）

---

## Virtualization Extension (H Extension) / 虚拟化扩展（H Extension）

### H Extension / Hypervisor 扩展

Elevates S-mode to HS-mode (Hypervisor-extended Supervisor mode), supporting efficient execution of Guest OS (VS-mode) on HS-mode. Key design goal from specification: both type-1/type-2 hypervisors can be efficiently implemented, and can be emulated on platforms without H Extension support via S-mode + M-mode.

**Prerequisites**:
- Must use RV32I/RV64I (RV32E/RV64E not supported)
- `mtval` must not be read-only zero
- Must support standard paging (RV32 Sv32, RV64 at least Sv39)

**Enabled via `misa` bit 7 (H bit), recommended not hardwired, allowing runtime disable.**

将 S-mode 提升为 HS-mode（Hypervisor-extended Supervisor mode），支持在 HS-mode 上高效运行 Guest OS（VS-mode）。规范原文的关键设计目标：type-1/type-2 虚拟机监控程序均可高效实现，且可在不支持 H 扩展的平台上通过 S-mode + M-mode 模拟。

**前置要求**：
- 必须使用 RV32I/RV64I（不支持 RV32E/RV64E）
- `mtval` 不得为只读零
- 必须支持标准分页（RV32 Sv32，RV64 至少 Sv39）

**通过 `misa` bit 7（H 位）启用，建议不硬连线，允许运行时禁用。**

---

### 特权模式与虚拟化模式 V

| V 位 | 名义特权 | 缩写 | 名称 | 两级翻译 |
|------|---------|------|------|---------|
| 0 | U | U-mode | 用户模式 | 关闭 |
| 0 | S | HS-mode | Hypervisor 扩展监管者模式 | 关闭 |
| 0 | M | M-mode | 机器模式 | 关闭 |
| 1 | U | VU-mode | 虚拟用户模式 | 开启 |
| 1 | S | VS-mode | 虚拟监管者模式 | 开启 |

**V=1 时两级地址翻译自动生效**；V=0 时与标准 S/U-mode 行为相同。

### HS-mode (Hypervisor-extended Supervisor mode) / HS 模式

Privilege level where Hypervisor or host OS runs. Same functionality as S-mode, but additionally holds CSRs for managing two-stage address translation and VS-mode Guests (hstatus, hgatp, hedeleg/hideleg, etc.). Normal S-mode OS can run unmodified in HS-mode.

Hypervisor 或宿主 OS 运行的特权级。功能与 S-mode 相同，但额外持有管理两级地址翻译和 VS-mode Guest 的 CSR（hstatus、hgatp、hedeleg/hideleg 等）。普通 S-mode OS 可在 HS-mode 中不加修改地运行。

### VS-mode (Virtual Supervisor mode) / 虚拟监管者模式

Privilege level where Guest OS runs (V=1, S nominal). VS-mode's S-level CSR operations actually access corresponding VS CSRs (vsstatus, vsatp, vstvec, etc.), not directly accessing HS-level CSRs.

Guest OS 运行的特权级（V=1，S 名义）。VS-mode 的 S 级 CSR 操作实际访问对应的 VS CSR（vsstatus、vsatp、vstvec 等），不直接访问 HS-level CSR。

### VU-mode (Virtual User mode) / 虚拟用户模式

Privilege level where Guest user programs run (V=1, U nominal).

Guest 用户程序运行的特权级（V=1，U 名义）。

---

### VS CSR (Virtual Supervisor CSRs) / VS CSR（虚拟监管者 CSR）

The H extension provides VS CSRs as shadow copies of S-level CSRs for a guest OS.

H 扩展提供一套 VS CSR，作为 Guest OS 所见 S 级 CSR 的影子副本：

| HS-level CSR | Corresponding VS CSR / 对应 VS CSR | Description / 说明 |
|-------------|-------------------------------------|--------------------|
| `sstatus` | `vsstatus` | Status register / 状态寄存器 |
| `stvec` | `vstvec` | Trap vector base address / trap 向量基地址 |
| `sscratch` | `vsscratch` | Scratch register / scratch 寄存器 |
| `sepc` | `vsepc` | Exception program counter / 异常程序计数器 |
| `scause` | `vscause` | Trap cause / trap 原因 |
| `stval` | `vstval` | Trap auxiliary value / trap 附加信息 |
| `sip` / `sie` | `vsip` / `vsie` | Interrupt pending/enable / 中断挂起/使能 |
| `satp` | `vsatp` | Address translation (VS stage) / 地址翻译（VS-stage） |
| `stimecmp` | `vstimecmp` | S-mode timer comparison (Sstc) / S-mode 定时器比较（Sstc） |

**Rule / 规则：** When V=1, instructions that access an S-level CSR are redirected to its corresponding VS CSR. Direct access using a VS CSR's own address raises a virtual-instruction exception.

V=1 时，访问 S 级 CSR 的指令自动重定向到对应 VS CSR；直接按 VS CSR 自身地址访问时触发 virtual-instruction exception。

### vsstatus Special Semantics / vsstatus 特殊语义

When V=1, both the HS-level `sstatus`.FS and `vsstatus`.FS fields are effective. If either is Off, floating-point instructions raise an illegal-instruction exception; changing floating-point state marks both fields Dirty. This lets the hypervisor maintain its own FS state without always saving and restoring it during a VM switch. The vector VS fields follow the same rule.

V=1 时，**HS-level `sstatus`.FS 和 `vsstatus`.FS 同时有效**：任一为 Off 则浮点指令触发非法指令异常；修改浮点状态时两者均被置为 Dirty。这使 Hypervisor 可以独立维护自己的 FS 状态，无需在 VM 切换时总是保存/恢复浮点状态。向量 VS 字段同理。

---

### Two-Stage Address Translation / 两级地址翻译

When V=1, two-stage translation is always active, although each stage can be disabled by setting `vsatp` or `hgatp` to Bare:

**V=1 时始终有效**，虽然可通过令 vsatp 或 hgatp 为 Bare 来逐级禁用：

```
GVA（Guest Virtual Address）
  ↓ VS-stage 翻译（vsatp 控制，Sv39/48/57）
GPA（Guest Physical Address）
  ↓ G-stage 翻译（hgatp 控制，Sv39x4/48x4/57x4）
SPA（Supervisor Physical Address）
  ↓ PMP 检查
机器物理地址
```

**G-stage special rules / G-stage 特殊规则：**
- All accesses, including implicit accesses during VS-stage page-table walks, are treated as U-mode accesses / 所有访问（包括 VS-stage 页表遍历的隐式访问）均被视为 U 级访问。
- Faults are guest-page faults (causes 20/21/23), not ordinary page faults (12/13/15) / 触发的是客户页错误（cause 20/21/23），而非普通页错误（12/13/15）。
- `htval` records the faulting GPA, encoded after a two-bit right shift / `htval` 记录引发异常的 GPA，以右移 2 位编码。
- `stval` records the faulting GVA / `stval` 记录引发异常的 GVA。

### GVA (Guest Virtual Address) / 客户虚拟地址
Virtual address used by a guest OS; it is translated by the VS stage into a GPA.

Guest OS 使用的虚拟地址，经 VS-stage 翻译为 GPA。

### GPA (Guest Physical Address) / 客户物理地址
Physical address as perceived by a guest OS. It is an intermediate address space managed by the hypervisor and translated by the G stage into a supervisor physical address.

Guest OS "认为"的物理地址，实际上是 Hypervisor 管理的中间地址空间，需经 G-stage 翻译为真实的 Supervisor 物理地址。

### G-stage address translation / G-stage 地址翻译（第二阶段翻译）
Translates a GPA into an SPA under `hgatp`. Sv32x4/Sv39x4/Sv48x4/Sv57x4 use the corresponding Sv format but enlarge the root page table fourfold (16 KiB and 16 KiB aligned), adding two GPA address bits. For example, Sv39x4 supports a 41-bit GPA space.

将 GPA 翻译为 SPA，由 `hgatp` 控制。地址方案为 Sv32x4/Sv39x4/Sv48x4/Sv57x4——与对应 Sv 方案相同，但根页表扩大为 4 倍（16 KiB，16 KiB 对齐），虚拟地址宽度额外增加 2 位（如 Sv39x4 支持 41 位 GPA 空间）。

**Why two extra bits / 为什么额外 2 位：** This lets a hypervisor support a guest physical address space wider than its own virtual address space, such as a 34-bit physical-address RV32 guest.

**为什么额外 2 位**：允许 Hypervisor 在虚拟机中支持比自身 VA 空间更宽的物理地址（如运行 34 位物理地址的 RV32 Guest）。

---

### hgatp / hgatp
G-stage address-translation control register. It has the same basic structure as `satp`, but contains a VMID instead of an ASID:

G-stage 地址翻译控制寄存器，与 `satp` 结构相同，但含 VMID 而非 ASID：

- **MODE**: Bare / Sv32x4 / Sv39x4 / Sv48x4 / Sv57x4 / 模式：Bare / Sv32x4 / Sv39x4 / Sv48x4 / Sv57x4
- **VMID** (Virtual Machine Identifier): Tags a TLB entry's VM, with up to 7 bits on RV32 and 14 bits on RV64 / 标记 TLB 条目所属虚拟机，RV32 最多 7 位、RV64 最多 14 位。
- **PPN**: Physical page number of the G-stage root page table, 16 KiB aligned with PPN[1:0] always zero / G-stage 根页表的物理页号，16 KiB 对齐且 PPN[1:0] 始终为 0。

**注意**：写 `hgatp` 不触发隐式 TLB 刷新，需手动执行 HFENCE.GVMA。

**Note / 注意：** Writing `hgatp` does not implicitly flush the TLB; software must execute `HFENCE.GVMA`.

### VMID (Virtual Machine Identifier) / 虚拟机标识符
Like an ASID, a VMID tags TLB entries with their virtual machine, avoiding a full G-stage TLB flush on fast VM switches. VMIDs are local to each physical hart.

类似 ASID，标记 TLB 条目所属的虚拟机，支持多 VM 快速切换时不必全量刷新 G-stage TLB。VMID 对每个物理 hart 是本地的（不同 hart 的 VMID 独立）。

---

### hstatus / hstatus
Core H-extension status register, analogous to `mstatus` for M-mode. Its key fields are:

H 扩展的核心状态寄存器，类比 `mstatus` 对于 M-mode 的作用。关键字段：

| Field / 字段 | Meaning / 含义 |
|------|------|
| **VSXL** | VS-mode XLEN setting / VS-mode 的 XLEN 设置 |
| **VTVM** | When 1, VS-mode access to `satp` or `sfence.vma` raises a virtual-instruction exception / =1 时 VS-mode 访问 satp/sfence.vma 触发 virtual-instruction exception |
| **VTW** | When 1, a VS-mode WFI timeout raises a virtual-instruction exception / =1 时 VS-mode 执行 WFI 超时触发 virtual-instruction exception |
| **VTSR** | When 1, VS-mode `SRET` raises a virtual-instruction exception / =1 时 VS-mode 执行 SRET 触发 virtual-instruction exception |
| **VGEIN** | Selects an `hgeip` guest external interrupt as the VS external interrupt / 选择 hgeip 中的哪个 guest 外部中断作为 VS 级外部中断 |
| **HU** | When 1, permits U-mode execution of HLV/HSV instructions / =1 时允许 U-mode 执行 HLV/HSV 指令 |
| **SPV** | V bit saved on a trap into HS-mode (Previous V) / trap 进入 HS-mode 时保存的 V 位（Previous V） |
| **SPVP** | Nominal U=0/S=1 privilege saved on an HS-mode trap, used by HLV/HSV / trap 进入 HS-mode 时的名义特权（U=0/S=1），供 HLV/HSV 使用 |
| **GVA** | Indicates whether `stval` contains a guest virtual address on an HS-mode trap / trap 进入 HS-mode 时，stval 是否写入了 Guest Virtual Address |
| **VSBE** | VS-mode byte order, 0=little endian and 1=big endian / VS-mode 的字节序（0=小端，1=大端） |

### VTVM / VTW / VTSR / 虚拟化陷阱控制字段
Correspond to `mstatus`.TVM/TW/TSR but apply to VS-mode. They raise a **virtual-instruction exception** (cause 22), rather than an illegal-instruction exception, allowing a hypervisor to intercept guest privileged operations.

对应 `mstatus` 的 TVM/TW/TSR，但作用于 VS-mode，触发 **virtual-instruction exception**（cause=22）而非 illegal-instruction exception。Hypervisor 通过这三个字段拦截 Guest OS 的特权操作。

---

### Hypervisor Delegation Registers / Hypervisor 委托寄存器

### hedeleg / hedeleg
H-extension exception delegation register. It delegates exceptions already delegated to HS-mode further to VS-mode; after delegation, an exception occurring with V=1 enters VS-mode directly rather than HS-mode.

H 扩展异常委托寄存器，将已委托给 HS-mode 的异常进一步委托给 VS-mode。委托后，V=1 时发生的该异常直接进入 VS-mode（而不是 HS-mode）。

### hideleg / hideleg
H-extension interrupt delegation register. It delegates interrupts already delegated to HS-mode further to VS-mode. VS-level interrupts (codes 2/6/10) are translated to S-level interrupt numbers when delegated to VS-mode.

H 扩展中断委托寄存器，将已委托给 HS-mode 的中断进一步委托给 VS-mode。特殊规则：VS-level 中断（code 2/6/10）在委托给 VS-mode 时会自动翻译（虚拟 S-level 中断映射为 S-level 中断编号）。

**Trap routing rules / Trap 路由规则：**
- A trap in HS/U-mode enters M-mode unless `medeleg`/`mideleg` delegates it to HS-mode / HS/U-mode 中的 trap 进入 M-mode，除非 `medeleg`/`mideleg` 委托给 HS-mode。
- A trap in VS/VU-mode enters M-mode unless it is delegated first to HS-mode by `medeleg`/`mideleg`, then to VS-mode by `hedeleg`/`hideleg` / VS/VU-mode 中的 trap 进入 M-mode，除非先由 `medeleg`/`mideleg` 委托给 HS-mode，再由 `hedeleg`/`hideleg` 进一步委托给 VS-mode。

---

### Hypervisor Interrupt Registers / Hypervisor 中断寄存器

### hvip / hvip
Hypervisor virtual-interrupt pending register. HS-mode writes it to inject virtual interrupts into a guest:

Hypervisor 虚拟中断挂起寄存器，HS-mode 写入以向 Guest 注入虚拟中断：
- `VSSIP` (bit 2): Injects a VS-level software interrupt / 注入 VS 级软件中断
- `VSTIP` (bit 6): Injects a VS-level timer interrupt when Sstc is unavailable / 注入 VS 级定时器中断（Sstc 未实现时使用）
- `VSEIP` (bit 10): Injects a VS-level external interrupt / 注入 VS 级外部中断

### hip / hie / 超管者中断寄存器
`hip` (pending) and `hie` (enable) contain HS-mode-specific interrupt bits:

`hip`（pending）和 `hie`（enable）包含 HS-mode 专有的中断位：
- **SGEIP/SGEIE**: Supervisor-level guest external interrupt from `hgeip AND hgeie` / 监管者级客户外部中断（来自 `hgeip AND hgeie`）
- **VSEIP/VSEIE**: VS-level external interrupt, the OR of `hvip.VSEIP`, `hgeip[VGEIN]`, and a platform signal / VS 级外部中断（`hvip.VSEIP`、`hgeip[VGEIN]` 和平台信号的 OR）
- **VSTIP/VSTIE**: VS-level timer interrupt, the OR of `hvip.VSTIP` and the `vstimecmp` comparison result / VS 级定时器中断（`hvip.VSTIP` 和 `vstimecmp` 比较结果的 OR）
- **VSSIP/VSSIE**: VS-level software interrupt, an alias of `hvip.VSSIP` / VS 级软件中断（`hvip.VSSIP` 的别名）

### hgeip / hgeie / guest 外部中断寄存器
Each bit represents the external-interrupt set of one guest, as collected by the external interrupt controller. At most GEILEN guests are available (31 on RV32 and 63 on RV64). `hstatus.VGEIN` selects which guest external interrupt is routed to the VS external interrupt.

每个 bit 对应一个虚拟机（Guest）的中断集合，由外部中断控制器汇总填写。最多 GEILEN 个 guest（RV32 最大 31，RV64 最大 63）。`hstatus.VGEIN` 选择哪个 guest 的外部中断路由到 VS 级外部中断。

### pass-through / direct assignment / 直通（直接分配）
Mechanism that routes a physical device interrupt directly to a particular guest OS. It requires an interrupt controller that can collect interrupts by guest. Direct assignment is limited by GEILEN, while a hypervisor can emulate an unlimited number of devices in software.

将物理设备中断直接路由给特定 Guest OS 的机制，需要外部中断控制器支持按 Guest 分类收集中断。受 GEILEN 限制（每 hart 最多 GEILEN 个 Guest 可直接接收外部中断）。Hypervisor 可以无限制地**模拟**设备（软件模拟无 GEILEN 限制）。

---

### Hypervisor-Specific CSRs / Hypervisor 专用 CSR

### henvcfg / henvcfg
Hypervisor environment configuration register (64-bit). Like `menvcfg`/`senvcfg`, it controls execution-environment features for V=1, including FIOM, CBZE, CBCFE, CBIE, PBMTE, ADUE, and STCE.

Hypervisor 环境配置寄存器（64 位），与 `menvcfg`/`senvcfg` 类似，控制 V=1 时的执行环境特性（FIOM/CBZE/CBCFE/CBIE/PBMTE/ADUE/STCE 等）。

### hcounteren / hcounteren
Controls counter access while V=1, analogous to `mcounteren`/`scounteren`. A zero bit makes VS-mode access to the corresponding counter raise a virtual-instruction exception.

控制 V=1 时计数器的访问权限（类比 mcounteren/scounteren）。bit 为 0 时 VS-mode 访问对应计数器触发 virtual-instruction exception。

### htimedelta / htimedelta
64-bit register defining the clock offset visible to a guest: VS/VU-mode reads of the `time` CSR return `time + htimedelta`. It provides each VM with an independent relative time; a large unsigned value represents a negative offset through wraparound.

64 位寄存器，定义 Guest 看到的时钟偏移量：VS/VU-mode 读 `time` CSR 实际返回 `time + htimedelta`。支持为每个虚拟机提供独立的相对时间（大值可表示负偏移，通过溢出实现）。

### htval / htval
HS-mode trap auxiliary-value register, analogous to `stval`. For a guest-page fault, it holds the faulting GPA shifted right by two bits to accommodate an insufficient XLEN; other traps write zero.

HS-mode trap 附加寄存器（类比 stval），专用于 guest-page-fault：写入引发异常的 GPA（右移 2 位，以适应 XLEN 不够宽的场景）。其他 trap 写零。

### htinst / htinst
HS-mode trap instruction-information register. It supplies the trapping instruction, or a transformed form, to the hypervisor trap handler to accelerate emulation. It supplies special pseudoinstruction values for guest-page faults caused by implicit VS-stage memory accesses.

HS-mode trap 指令信息寄存器，提供引发 trap 的指令（或其变换形式）给 Hypervisor 的 trap 处理程序，加速指令模拟。对 VS-stage 隐式内存访问引发的 guest-page-fault 提供特殊伪指令值。

### vsatp / vsatp
VS-mode address-translation control register for the VS stage. With V=1, accesses to `satp` actually access `vsatp`.

VS-mode 的地址翻译控制寄存器，控制 VS-stage（第一阶段）翻译。当 V=1 时，访问 `satp` 实际访问 `vsatp`。

### vstimecmp / vstimecmp
VS-mode timer comparison register, provided by the cooperation of Sstc and H: a VSTIP occurs when `time + htimedelta >= vstimecmp`. A hypervisor can emulate a guest timer by setting `vstimecmp`, without routing through SBI.

VS-mode 的定时器比较寄存器（Sstc 扩展 + H 扩展协作）：`time + htimedelta >= vstimecmp` 时触发 VSTIP。Hypervisor 通过设置 `vstimecmp` 为 Guest OS 模拟定时器，无需通过 SBI 路由。

---

### H-Extension Instructions / H 扩展专用指令

### HLV / HSV / HLVX / 虚拟机 Load/Store 指令
HS-mode-only memory instructions, also permitted in U-mode when HU=1. They perform two-stage translation with VS/VU privilege selected by `hstatus.SPVP`:

HS-mode（或 HU=1 时的 U-mode）专用的访存指令，以 VS/VU 模式权限（由 `hstatus.SPVP` 决定）执行两级地址翻译：
- **HLV.B/H/W/D etc.**: VM loads corresponding to LB/LH/LW/LD / 对应 LB/LH/LW/LD 的 VM Load
- **HSV.B/H/W/D etc.**: VM stores corresponding to SB/SH/SW/SD / 对应 SB/SH/SW/SD 的 VM Store
- **HLVX.HU / HLVX.WU**: Use **execute** permission instead of read permission to emulate instruction fetch; R is not required but X is / 以**执行**权限替代读权限，用于模拟指令取指；不需要 R 位，但需要 X 位

**Execute-permission rule / 执行权限规则：** HLVX uses execute permission, rather than read permission, for its implicit memory access and two-stage translation. HS-level `sstatus`.MXR applies to both stages, whereas `vsstatus`.MXR applies only to the VS stage.

**V=1 时执行 HLV/HSV/HLVX → virtual-instruction exception；U-mode 且 HU=0 → illegal-instruction exception。**

### HFENCE.VVMA / HFENCE.VVMA
Flushes VS-stage TLB entries for the VM selected by the current `hgatp.VMID`, analogous to `SFENCE.VMA` at HS level. `rs1` is a virtual address (`x0` means all addresses) and `rs2` is an ASID (`x0` means all ASIDs). It affects only the current VMID.

刷新指定 VM（由当前 hgatp.VMID 标识）的 VS-stage TLB 缓存，类比 SFENCE.VMA 对 HS-level。参数：rs1=虚拟地址（x0 表示全部），rs2=ASID（x0 表示全部）。**HFENCE.VVMA 只对当前 VMID 有效**。

### HFENCE.GVMA / HFENCE.GVMA
Flushes G-stage TLB entries. `rs1` is a GPA shifted right by two bits (`x0` means all GPAs), and `rs2` is a VMID (`x0` means all VMIDs). It may execute in HS-mode when `mstatus.TVM=0`, or in M-mode.

刷新 G-stage TLB 缓存，参数：rs1=GPA（右移 2 位，x0 表示全部），rs2=VMID（x0 表示全部）。执行条件：HS-mode（mstatus.TVM=0）或 M-mode。

**Correct TLB handling during a VM switch / VM 切换的正确 TLB 处理：**

```asm
# Correct VM-switch sequence; prevents VS-stage entries being cached under the wrong VMID.
# 切换到新 VM 的正确步骤；防止 VS-stage 条目被错误 VMID 缓存。
csrw vsatp, x0         # Clear vsatp / 先清零 vsatp
csrw hgatp, new_hgatp  # Switch hgatp with the new VMID / 再切换含新 VMID 的 hgatp
csrw vsatp, new_vsatp  # Install the new vsatp / 最后写入新 vsatp
```

---

### H-Extension Trap Causes / H 扩展新增 Trap Cause

The H extension adds these trap cause values:

H 扩展增加的 trap cause 值：

| Interrupt/Exception / 中断或异常 | Code / 编号 | Name / 名称 |
|--------------------|------|------|
| Interrupt / 中断 | 2 | Virtual supervisor software interrupt / 虚拟监管者软件中断 |
| Interrupt / 中断 | 6 | Virtual supervisor timer interrupt / 虚拟监管者定时器中断 |
| Interrupt / 中断 | 10 | Virtual supervisor external interrupt / 虚拟监管者外部中断 |
| Interrupt / 中断 | 12 | Supervisor guest external interrupt (SGEIP) / 监管者客户外部中断（SGEIP） |
| Exception / 异常 | 10 | Environment call from VS-mode / 来自 VS-mode 的环境调用 |
| Exception / 异常 | 20 | Instruction guest-page fault / 指令客户页错误 |
| Exception / 异常 | 21 | Load guest-page fault / Load 客户页错误 |
| Exception / 异常 | 22 | Virtual instruction exception / 虚拟指令异常 |
| Exception / 异常 | 23 | Store/AMO guest-page fault / Store/AMO 客户页错误 |

### virtual-instruction exception / 虚拟指令异常（cause=22）
Raised, rather than an illegal-instruction exception, when V=1 and software attempts an instruction that is legal in HS-mode but intercepted in VS-mode. Typical cases include:

V=1 时，尝试执行在 HS-mode 合法但在 VS-mode 被拦截的指令时触发，而非 illegal-instruction exception。典型场景：

- Accessing a Hypervisor CSR or a VS CSR by its direct address / 访问 Hypervisor CSR 或 VS CSR（直接地址）
- Executing SFENCE.VMA/SINVAL.VMA when VTVM=1 / 执行 SFENCE.VMA/SINVAL.VMA 且 VTVM=1
- Executing WFI until a timeout when VTW=1 / 执行 WFI 且 VTW=1 超时
- Executing SRET when VTSR=1 / 执行 SRET 且 VTSR=1
- Executing HFENCE.VVMA/GVMA/HLV/HSV / 执行 HFENCE.VVMA/GVMA/HLV/HSV

HS-qualified 指令规则：若某指令在 HS-mode（假设 TVM=0）下合法，则在 VS/VU-mode 下无权限时触发 virtual-instruction（而非 illegal-instruction），便于 Hypervisor 快速拦截和模拟。

**HS-qualified instruction rule / HS-qualified 指令规则：** If an instruction is legal in HS-mode (assuming TVM=0), insufficient privilege in VS/VU-mode raises virtual-instruction, rather than illegal-instruction, so the hypervisor can intercept and emulate it quickly.

### guest-page-fault / 客户页错误（cause 20/21/23）
Raised when G-stage translation fails for an instruction, load, or store. During handling:

G-stage 地址翻译失败时触发（对应 instruction/load/store）。处理时：
- `stval`: Faulting GVA / 引发异常的 GVA
- `htval`: Faulting GPA shifted right by two bits, possibly zero / 引发异常的 GPA（右移 2 位，可能为 0）
- `htinst`: Transformed faulting instruction or a pseudoinstruction for implicit VS-stage access / 引发异常的指令变换，或用于 VS-stage 隐式访问的伪指令

**guest-page-fault 可从 M-mode 委托到 HS-mode，但不能进一步委托到 VS-mode。**

**A guest-page fault can be delegated from M-mode to HS-mode, but cannot be delegated further to VS-mode.**

---

### H-Extension `mstatus` Fields / `mstatus` H 扩展新字段

### MPV (Machine Previous Virtualization mode) / 机器前虚拟化模式
Stores the V bit when a trap enters M-mode; `mret` restores it.

trap 进入 M-mode 时保存的 V 位（`mstatus.MPV`），`mret` 时恢复。

### MPRV 与 H 扩展的交互
When `mstatus.MPRV=1`, explicit memory accesses perform address translation with the (V, privilege) pair selected by MPV+MPP:

当 `mstatus.MPRV=1` 时，显式内存访问以 MPV+MPP 指定的（V, 特权）组合执行地址翻译：
- MPV=1, MPP=U: Perform two-stage translation in VU-mode / 以 VU 模式执行两级翻译
- MPV=1, MPP=S: Perform two-stage translation in VS-mode / 以 VS 模式执行两级翻译
- MPV=0: Use ordinary MPRV semantics with HS-level translation only / 与普通 MPRV 语义相同（仅 HS-level 翻译）

### mtval2 / mtval2
Second M-mode trap-value register added by H. When a guest-page fault enters M-mode, it holds the faulting GPA shifted right by two bits. It is analogous to `htval`, but applies to M-mode traps.

M-mode 第二 trap 值寄存器（H 扩展新增）：guest-page-fault 进入 M-mode 时，写入引发异常的 GPA（右移 2 位）。与 `htval` 类似，但适用于 M-mode trap。

### mtinst / mtinst
M-mode trap instruction-information register added by H; it corresponds to HS-mode `htinst`.

M-mode trap 指令信息寄存器（H 扩展新增），对应 HS-mode 的 `htinst`。

---

### G-Stage PTE Special Rules / G-stage PTE 特殊规则

**G-stage PTE special rules / G-stage PTE 特殊规则：**

- **G bit**：G-stage PTE 中 G 位当前未使用，软件应清零
- **U bit**：G-stage PTE 使用相同格式（含 U bit），但 G-stage 翻译时所有访问均被视为 U 级，故 U=1 的页对 G-stage 总是可访问
- **A/D bit 更新**：G-stage PTE 的 A/D bit 更新规则与 VS-stage 相同，但更新时机不同（D 位必须在 VS-stage 页表的 G-stage 隐式写访问期间精确更新）

- **G bit:** Currently unused in a G-stage PTE; software must clear it.
- **U bit:** G-stage PTEs use the same format, including U. Every G-stage access is treated as U-mode, so pages with U=1 are always accessible to G-stage translation.
- **A/D-bit updates:** Follow the VS-stage rules, but their timing differs: D must be updated precisely during an implicit G-stage write caused by a VS-stage page-table walk.

---

## Other Important Z Extensions / 其他重要 Z 扩展

### Zicsr / Zicsr
CSR access instruction extension defining `csrrw`/`csrrs`/`csrrc` and their immediate variants. It was separated from the base ISA in RISC-V Specification 2.2, although implementations supporting the privileged architecture must implement it in practice.

CSR 访问指令扩展，定义了 `csrrw`/`csrrs`/`csrrc` 及其立即数变体。从 RISC-V 2.2 规范起从基础 ISA 中分离出来作为独立扩展（但实际上所有支持特权架构的实现都必须实现）。

### Zifencei / Zifencei
Instruction-cache synchronization extension providing `FENCE.I`. It guarantees that newly written executable memory is correctly fetched as instructions; JIT compilers and dynamic linkers must execute it after generating code.

指令缓存同步屏障扩展，提供 `FENCE.I` 指令，确保在修改代码（写 I-cache 可见内存）后能正确执行新指令。JIT 编译器、动态链接器必须在生成代码后调用。

### Zicond / Zicond
Ratified conditional-operations extension providing `czero.eqz` (clear when equal to zero) and `czero.nez` (clear when not equal to zero). It can replace a conditional branch and reduce branch-prediction pressure:

条件操作扩展（已批准），提供 `czero.eqz`（等于零时置零）和 `czero.nez`（不等于零时置零），可替代条件分支，减少分支预测压力：
```
# 等价于：rd = (rs2 != 0) ? rs1 : 0
czero.eqz rd, rs1, rs2
```

### Zba / Zbb / Zbc / Zbs
See the B Extension entry above.

→ 详见 B Extension 条目

### Smstateen / Smstateen
Machine-mode State Enable extension. M-mode controls which CSRs and features are visible to S/U-mode, enabling fine-grained feature-access control.

Machine-mode State Enable 扩展，M 模式可控制哪些 CSR/功能对 S/U 模式可见，实现特性的细粒度访问控制。

### Ssaia (Advanced Interrupt Architecture for Supervisor) / 高级中断架构
Provides stronger interrupt-control capability for S-mode through APLIC and IMSIC, supporting MSI interrupts and virtualization scenarios. It is paired with the AIA Non-ISA specification.

为 S 模式提供更强大的中断控制能力（APLIC + IMSIC），支持 MSI 中断和虚拟化场景。与 AIA Non-ISA 规范配套。

---

## Cache Management Extensions (Zicbom / Zicboz / Zicbop) / 缓存管理扩展（Zicbom / Zicboz / Zicbop）

### Zicbom (Cache Block Management Operations) / 缓存块管理操作
Provides three cache-block management instructions, operating on one cache block (usually 64 bytes):
- `cbo.clean addr`: Write back a dirty cache line without invalidating it.
- `cbo.flush addr`: Write back and invalidate a cache line, used for DMA coherency.
- `cbo.inval addr`: Invalidate a cache line without writing it back, used for DMA input buffers.

提供 3 条缓存块管理指令，操作单位为 cache block（通常 64 字节）：
- `cbo.clean addr`：将脏缓存行写回内存（不使其无效）
- `cbo.flush addr`：写回并使缓存行无效（DMA 一致性场景）
- `cbo.inval addr`：直接使缓存行无效（不写回，用于 DMA 输入缓冲）

**典型用法**：DMA 传输前调用 `cbo.flush` 确保数据已写回内存；DMA 传输完成后调用 `cbo.inval` 确保 CPU 重新从内存读取新数据。

**Typical use / 典型用法：** Execute `cbo.flush` before DMA so data reaches memory; execute `cbo.inval` after DMA so the CPU reloads the updated data from memory.

### Zicboz (Cache Block Zero) / 缓存块清零
Provides `cbo.zero addr`, which clears one cache block. It can be more efficient than byte-wise `memset` because hardware may create an all-zero cache line without reading memory first.

提供 `cbo.zero addr` 指令，将一个缓存块大小的内存区域清零，比逐字节写零的 `memset` 更高效（允许硬件在不读取内存的情况下直接在缓存中创建全零行）。

### Zicbop (Cache Block Prefetch) / 缓存块预取
Provides `prefetch.r`/`prefetch.w`/`prefetch.i` to hint that an address will soon be read, written, or fetched as instructions, respectively. They are analogous to x86 `PREFETCHT0/T1/T2` hints.

提供 `prefetch.r`/`prefetch.w`/`prefetch.i` 指令（分别预取供读/写/指令使用），向硬件提示即将访问的内存地址，类似 x86 的 `PREFETCHT0/T1/T2`。

### CBOM_BLOCK_SIZE / CBOM_BLOCK_SIZE
System cache-block size in bytes. Software obtains it from `/sys/bus/cpu/devices/cpu*/cache_info` or the `riscv,cbom-block-size` device-tree property; Zicbom and Zicboz operations must use correctly aligned addresses.

系统缓存块大小（字节），通过设备树 `/sys/bus/cpu/devices/cpu*/cache_info` 或 `riscv,cbom-block-size` DT 属性获取，Zicbom/Zicboz 指令必须知道此值才能正确对齐操作地址。

---

## Atomic Operation Enhancement Extensions / 原子操作增强扩展

### Zacas (Atomic Compare-and-Swap) / 原子比较并交换
Provides CAS (Compare-And-Swap), filling the gap left by the base A extension's LR/SC-only primitive:
- `amocas.w/d/q rd, rs2, (rs1)`: If `*rs1 == rd`, store `rs2` to `*rs1`, then write the old value to `rd`; otherwise, only write the old value to `rd`.
- Supports 128-bit doubleword CAS (`.q` suffix, RV64) for lock-free data structures, including updates that avoid the ABA problem.

提供 CAS（Compare-And-Swap）原子指令，弥补基础 A 扩展只有 LR/SC 的不足：
- `amocas.w/d/q rd, rs2, (rs1)`：若 `*rs1 == rd`，则 `*rs1 ← rs2`，并将旧值写回 `rd`；否则只将旧值写回 `rd`
- 支持 128 位双字 CAS（`.q` 后缀，RV64），用于无锁数据结构（如免 ABA 问题的双字原子更新）

### Zabha (Byte and Halfword Atomics) / 字节和半字原子操作
Extends A-extension AMOs to 8-bit bytes and 16-bit halfwords for fine-grained shared data structures, such as byte-level reference counts.

将 A 扩展的 AMO 操作扩展到 8 位（byte）和 16 位（halfword）粒度，用于细粒度共享数据结构（如字节级引用计数）。

### Zalrsc / Zalrsc
Separates LR/SC from the A extension as an independent extension, alongside Zaamo in the atomic architecture reorganization. It suits minimal implementations that need LR/SC but not AMOs.

将 A 扩展的 LR/SC 独立化为单独扩展（原子架构重组的一部分，与 Zaamo 一起拆分自原 A 扩展），便于只需 LR/SC 而不需 AMO 的极简实现。

---

## Low-Power and Synchronization Extensions / 低功耗与同步扩展

### Zawrs (Wait on Reservation Set) / 等待保留集
Provides `wrs.nto` (wait without timeout) and `wrs.sto` (wait with timeout). After LR establishes a reservation, the hart can enter low-power wait until another hart writes the reservation set, rather than polling in a spinlock loop.

提供 `wrs.nto`（Wait for Reservation Set, No Time Out）和 `wrs.sto`（有超时版本）指令。在 LR 建立保留后调用，hart 进入低功耗等待直到保留集被其他 hart 写入（即监视地址有变化）唤醒，比轮询 + WFI 的自旋锁更节能。

### Zihintpause (Pause Hint) / 暂停提示
The `pause` instruction, encoded as a specific `FENCE` variant, tells hardware that code is spinning. The processor may reduce frequency or voltage, or yield a simultaneous-multithreading slot, to reduce wasted power. It is analogous to x86 `PAUSE` and is used in Linux spinlocks.

`pause` 指令（编码为特定 `FENCE` 变体），向硬件提示当前代码处于自旋等待循环中，允许处理器降低频率/电压或让出超线程槽位，减少无效功耗。类似 x86 的 `PAUSE` 指令，Linux 内核在自旋锁中使用。

### Zihintntl (Non-Temporal Locality Hints) / 非时间局部性提示
Provides non-temporal locality hints, indicating that an accessed memory location will not be reused soon. Hardware may bypass cache, which is useful for streaming large writes.

提供非时间局部性提示，告诉硬件该内存访问不会很快被重复使用，可绕过缓存（类似 x86 的 non-temporal store），用于流式大数据写入场景。

---

## 内存模型扩展

### Ztso (Total Store Ordering) / 全存储序

Elevates hart's memory model from default RVWMO (weak ordering) to TSO (x86-like total store ordering), allowing software ported from x86 to reduce memory barrier usage. Implementation optional, identified by `misa.Ztso`.

将 hart 的内存模型从默认的 RVWMO（弱序）提升为 TSO（类 x86 全存储序），允许从 x86 移植的软件减少内存屏障使用。实现可选，通过 `misa.Ztso` 标识。

### Zam (Misaligned Atomics) / 非对齐原子操作

Allows AMO and LR/SC operations on misaligned addresses (base A Extension requires alignment), simplifying certain protocol stacks and network data processing code.

允许 AMO 和 LR/SC 在非对齐地址上操作（基础 A 扩展要求对齐），简化某些协议栈和网络数据处理代码。

---

## Timer Extensions / 定时器扩展

### Sstc (Supervisor-mode Timer Comparison) / 监管者模式定时器比较

Adds `stimecmp` CSR (and `stimecmph` for RV32) to S-mode, allowing OS kernel to directly set timer trigger point without calling into M-mode via SBI `ecall` each time. Supported in Linux 6.x, significantly reduces timer interrupt latency.

为 S 模式新增 `stimecmp` CSR（及 RV32 的 `stimecmph`），允许 OS 内核直接设置定时器触发点，无需每次通过 SBI `ecall` 进入 M 模式。Linux 6.x 已支持，定时器中断延迟显著降低。

### Smcntrpmf (Cycle and Instret Privilege Mode Filtering) / 计数器特权模式过滤

Allows configuring filter conditions for `cycle` and `instret` counters (count only in certain privilege modes), enabling precise user-mode/kernel-mode separated performance measurement, unaffected by noise from system calls, interrupts, etc.

允许对 `cycle` 和 `instret` 计数器配置过滤条件（仅在某些特权模式下计数），实现精确的用户态/内核态分离性能测量，不受系统调用、中断等噪声影响。

---

## Security Extensions / 安全扩展

### Scalar Cryptography Extensions / 标量密码学扩展

RISC-V Scalar Cryptography Extensions provide hardware acceleration for cryptographic algorithms without requiring large lookup tables (avoiding cache timing side-channels) and ensuring constant-time execution (preventing timing attacks).

RISC-V 标量密码学扩展为密码学算法提供硬件加速，无需大型查找表（避免缓存定时侧信道），并确保常量时间执行（防止定时攻击）。

**Extension Hierarchy / 扩展层次：**
- **Zk** = Zkn + Zkr + Zkt (complete scalar cryptography suite / 完整标量密码学套件)
- **Zkn** = Zkne + Zknd + Zknh + Zbkb + Zbkc + Zbkx (NIST algorithm group / NIST 算法组)
- **Zks** = Zksed + Zksh + Zbkb + Zbkc + Zbkx (ShangMi algorithm group / ShangMi 国密算法组)

---

### Zkt (Data Independent Execution Latency) / Zkt（数据无关执行延迟）

**Purpose / 目的：** Guarantees that certain instructions have execution time independent of operand data values, preventing timing side-channel attacks.

**Covered Instructions / 覆盖指令：**
- All AES instructions (Zkne/Zknd) / 所有 AES 指令（Zkne/Zknd）
- All SM4 instructions (Zksed) / 所有 SM4 指令（Zksed）
- All SHA-2 instructions (Zknh) / 所有 SHA-2 指令（Zknh）
- All SM3 instructions (Zksh) / 所有 SM3 指令（Zksh）
- Bit-manipulation instructions used in cryptography (Zbkb/Zbkc/Zbkx) / 密码学使用的位操作指令（Zbkb/Zbkc/Zbkx）
- Division/remainder operations (`DIV`, `DIVU`, `REM`, `REMU`, `DIVW`, etc.) / 除法和取余操作（`DIV`、`DIVU`、`REM`、`REMU`、`DIVW` 等）

**Why Important / 重要性：** Without Zkt, attackers can measure instruction execution time to infer secret key bits. Cryptographic libraries (OpenSSL, libsodium) require this guarantee for secure implementations.

**标记某些指令的执行时间与操作数数据无关，防止定时侧信道攻击（timing side-channel）。密码学库（如 OpenSSL）依赖此保证。**

---

### Zbkb / Zbkc / Zbkx (Bit Manipulation for Cryptography) / 密码学用位操作

**Zbkb** (Bitmanip Instructions for Cryptography / 密码学位操作指令):
- `andn`, `orn`, `xnor` — Logical operations with NOT / 带 NOT 的逻辑操作
- `rol`, `ror`, `rori` — Rotate left/right, critical for SHA-2 and AES / 左右循环移位，对 SHA-2 和 AES 至关重要
- `rev8` — Byte-order reversal for endianness conversion / 字节序反转，用于端序转换
- `pack`, `packh` — Pack low bytes from two sources / 打包两个源操作数的低字节
- `brev8` — Bit reversal within each byte / 每个字节内的比特反转
- `zip`, `unzip` (RV32 only) — Bit interleaving / 位交错（仅 RV32）

**Zbkc** (Carryless Multiplication for Cryptography / 密码学无进位乘法):
- `clmul`, `clmulh` — Carryless multiplication for GCM and CRC / 用于 GCM 和 CRC 的无进位乘法

**Zbkx** (Crossbar Permutations / 交叉开关置换):
- `xperm.n`, `xperm.b` — Nibble/byte permutations for AES MixColumns / 用于 AES MixColumns 的半字节/字节置换

**用途：** 这些位操作指令是实现 AES、SHA-2、GCM 等算法的基础，Zkn 和 Zks 都依赖它们。

---

### Zkne (AES Encryption) / Zkne（AES 加密）

**RV32 Instructions / RV32 指令：**

| Instruction / 指令 | Encoding / 编码 | Operation / 操作 | Description / 说明 |
|-------------|----------|-----------|-------------|
| `aes32esmi rd, rs1, rs2, bs` | `opcode=0110011`<br>`funct7=0011010`<br>`bs[1:0]=imm[31:30]` | AES-128/192/256 middle-round encryption / AES-128/192/256 中间轮加密 | Performs SubBytes + ShiftRows + MixColumns on one `rs2` byte selected by `bs`, then XORs with `rs1` / 对 `bs` 选择的一个 `rs2` 字节执行 SubBytes + ShiftRows + MixColumns，再与 `rs1` 异或 |
| `aes32esi rd, rs1, rs2, bs` | `funct7=0010010` | AES final-round encryption / AES 最终轮加密 | SubBytes + ShiftRows only, without MixColumns / 仅执行 SubBytes + ShiftRows，不执行 MixColumns |
| `aes32dsmi rd, rs1, rs2, bs` | `funct7=0011110` | AES middle-round decryption / AES 中间轮解密 | InvSubBytes + InvShiftRows + InvMixColumns / 逆 SubBytes + 逆 ShiftRows + 逆 MixColumns |
| `aes32dsi rd, rs1, rs2, bs` | `funct7=0010110` | AES final-round decryption / AES 最终轮解密 | InvSubBytes + InvShiftRows only / 仅执行逆 SubBytes + 逆 ShiftRows |

**RV64 Instructions / RV64 指令：**

| Instruction / 指令 | Encoding / 编码 | Operation / 操作 | Description / 说明 |
|-------------|----------|-----------|-------------|
| `aes64es rd, rs1, rs2` | `opcode=0110011`<br>`funct7=0011001`<br>`rs2=any`<br>`funct3=000` | AES-128/192/256 middle round (64-bit) / AES-128/192/256 中间轮（64 位） | Applies SubBytes + ShiftRows to two columns (64 bits) / 对两列（64 位）执行 SubBytes + ShiftRows |
| `aes64esm rd, rs1, rs2` | `funct7=0011011` | AES middle round with MixColumns / 含 MixColumns 的 AES 中间轮 | Adds MixColumns for the forward cipher / 为正向密码加入 MixColumns |
| `aes64ds rd, rs1, rs2` | `funct7=0011101` | AES middle-round decryption / AES 中间轮解密 | InvSubBytes + InvShiftRows on 64-bit state data / 对 64 位状态数据执行逆 SubBytes + 逆 ShiftRows |
| `aes64dsm rd, rs1, rs2` | `funct7=0011111` | AES middle decryption with InvMixColumns / 含 InvMixColumns 的 AES 中间轮解密 | Adds InvMixColumns for the inverse cipher / 为逆向密码加入 InvMixColumns |
| `aes64im rd, rs1` | `funct7=0011000`<br>`rs2=00000` | InvMixColumns only / 仅 InvMixColumns | Used to prepare decryption round keys / 用于准备解密轮密钥 |
| `aes64ks1i rd, rs1, rnum` | `funct7=0011000`<br>`rnum[3:0]=imm[23:20]` | AES key-schedule round / AES 密钥扩展轮 | Generates a partial round key for round `rnum` (0-10) / 为轮次 `rnum`（0-10）生成部分轮密钥 |
| `aes64ks2 rd, rs1, rs2` | `funct7=0111111` | AES key-schedule completion / AES 密钥扩展完成 | Combines two sources to complete key expansion / 组合两个源操作数以完成密钥扩展 |

**RV32 指令说明：**
- `bs` (byte select) selects byte 0-3 in `rs2` as the input.
- Each instruction processes one byte; four instructions complete one 32-bit column.
- This design suits small embedded processors with lower hardware cost.

- `bs` (byte select) 字段选择 rs2 中的哪个字节（0–3）作为输入
- 每条指令处理 1 个字节，需要 4 条指令完成一列（32 位）
- 适合小型嵌入式处理器（低硬件成本）

**RV64 指令说明：**
- Each instruction processes two columns (64 bits), twice the RV32 data width.
- The `rnum` immediate of `aes64ks1i` specifies a key-schedule round (0-10 for AES-128/192/256).

- 每条指令处理 2 列（64 位），效率是 RV32 的 2 倍
- `aes64ks1i` 的 `rnum` 立即数指定轮次（0–10 for AES-128/192/256）

**Key Expansion Process / 密钥扩展过程：**
1. Use `aes64ks1i` with a round number to compute a partial round key / 使用带轮次号的 `aes64ks1i` 计算部分轮密钥
2. Use `aes64ks2` to combine results and complete the round key / 使用 `aes64ks2` 组合结果并完成轮密钥
3. For decryption, apply `aes64im` to intermediate round keys, excluding the first and last / 解密时对首尾轮密钥之外的中间轮密钥应用 `aes64im`

**编码示例（AES-128 加密一轮，RV64）：**
```assembly
# Input / 输入: state in a0,a1 (128 bits = 2×64 bits)
# Round key / 轮密钥: t0,t1
aes64esm a0, a0, a1    # First 64 bits / 前 64 位：含 MixColumns 的加密轮
aes64esm a1, a1, a0    # Second 64 bits / 后 64 位
xor      a0, a0, t0    # AddRoundKey / 加轮密钥
xor      a1, a1, t1
```

---

### Zknd (AES Decryption) / Zknd（AES 解密）

Contains the inverse operations for AES decryption. On RV32, it uses `aes32dsmi`/`aes32dsi`; on RV64, it uses `aes64ds`/`aes64dsm`/`aes64im`.

包含 AES 解密的逆向操作。在 RV32 上使用 `aes32dsmi`/`aes32dsi`，在 RV64 上使用 `aes64ds`/`aes64dsm`/`aes64im`。

**Note / 注意：** Zknd and Zkne are separate extensions because some use cases only need encryption (e.g., CTR mode, GCM mode use only AES encryption even for "decryption").

Zknd 和 Zkne 是独立扩展，因为某些场景只需要加密（如 CTR 模式、GCM 模式即使"解密"也只用 AES 加密方向）。

---

### Zknh (SHA-2 Hash Functions) / Zknh（SHA-2 哈希函数）

Accelerates SHA-256 and SHA-512 by implementing the sigma (Σ) and Ch/Maj functions in hardware.

通过硬件实现 sigma (Σ) 和 Ch/Maj 函数来加速 SHA-256 和 SHA-512。

**SHA-256 Instructions (RV32 & RV64) / SHA-256 指令（RV32 和 RV64）：**

| Instruction / 指令 | Encoding / 编码 | Operation / 操作 | Description / 说明 |
|-------------|----------|-----------|-------------|
| `sha256sig0 rd, rs1` | `opcode=0010011`<br>`funct7=0001000`<br>`rs2=00010`<br>`funct3=001` | σ₀(x) = ROTR(x,7) ⊕ ROTR(x,18) ⊕ SHR(x,3) | SHA-256 small sigma 0 for the message schedule / SHA-256 消息调度小 sigma 0 |
| `sha256sig1 rd, rs1` | `rs2=00011` | σ₁(x) = ROTR(x,17) ⊕ ROTR(x,19) ⊕ SHR(x,10) | SHA-256 small sigma 1 for the message schedule / SHA-256 消息调度小 sigma 1 |
| `sha256sum0 rd, rs1` | `rs2=00000` | Σ₀(x) = ROTR(x,2) ⊕ ROTR(x,13) ⊕ ROTR(x,22) | SHA-256 big sigma 0 for compression / SHA-256 压缩函数大 sigma 0 |
| `sha256sum1 rd, rs1` | `rs2=00001` | Σ₁(x) = ROTR(x,6) ⊕ ROTR(x,11) ⊕ ROTR(x,25) | SHA-256 big sigma 1 for compression / SHA-256 压缩函数大 sigma 1 |

**SHA-512 Instructions (RV64 only) / SHA-512 指令（仅 RV64）：**

| Instruction / 指令 | Encoding / 编码 | Operation / 操作 | Description / 说明 |
|-------------|----------|-----------|-------------|
| `sha512sig0 rd, rs1` | `funct7=0001000`<br>`rs2=00110` | σ₀(x) = ROTR(x,1) ⊕ ROTR(x,8) ⊕ SHR(x,7) | SHA-512 small sigma 0 (64-bit) / SHA-512 小 sigma 0（64 位） |
| `sha512sig1 rd, rs1` | `rs2=00111` | σ₁(x) = ROTR(x,19) ⊕ ROTR(x,61) ⊕ SHR(x,6) | SHA-512 small sigma 1 (64-bit) / SHA-512 小 sigma 1（64 位） |
| `sha512sum0 rd, rs1` | `rs2=00100` | Σ₀(x) = ROTR(x,28) ⊕ ROTR(x,34) ⊕ ROTR(x,39) | SHA-512 big sigma 0 (64-bit) / SHA-512 大 sigma 0（64 位） |
| `sha512sum1 rd, rs1` | `rs2=00101` | Σ₁(x) = ROTR(x,14) ⊕ ROTR(x,18) ⊕ ROTR(x,41) | SHA-512 big sigma 1 (64-bit) / SHA-512 大 sigma 1（64 位） |

**SHA-512 RV32 Instructions / SHA-512 RV32 指令：**

| Instruction / 指令 | Encoding / 编码 | Operation / 操作 | Description / 说明 |
|-------------|----------|-----------|-------------|
| `sha512sig0l rd, rs1, rs2` | `funct7=0101010` | Low 32 bits of σ₀ / σ₀ 的低 32 位 | Requires a paired instruction for the full 64-bit result / 需配对指令完成完整 64 位结果 |
| `sha512sig0h rd, rs1, rs2` | `funct7=0101110` | High 32 bits of σ₀ / σ₀ 的高 32 位 | Combines with `sig0l` for the full result / 与 `sig0l` 合并得到完整结果 |
| `sha512sig1l rd, rs1, rs2` | `funct7=0101011` | Low 32 bits of σ₁ / σ₁ 的低 32 位 | Low half of the 64-bit result / 64 位结果的低半部分 |
| `sha512sig1h rd, rs1, rs2` | `funct7=0101111` | High 32 bits of σ₁ / σ₁ 的高 32 位 | High half of the 64-bit result / 64 位结果的高半部分 |
| `sha512sum0r rd, rs1, rs2` | `funct7=0101000` | Σ₀ with register pair / 使用寄存器对的 Σ₀ | RV32 uses register pairs for 64-bit operations / RV32 用寄存器对实现 64 位操作 |
| `sha512sum1r rd, rs1, rs2` | `funct7=0101001` | Σ₁ with register pair / 使用寄存器对的 Σ₁ | RV32 uses register pairs for 64-bit operations / RV32 用寄存器对实现 64 位操作 |

**Performance Gain / 性能提升：**
- Without instructions, σ₀ requires about six instructions (two rotates, one shift, and three XORs) / 无指令加速时，σ₀ 需要约 6 条指令（两次循环移位、一次移位和三次异或）
- `sha256sig0` performs it in one instruction / `sha256sig0` 用单条指令完成
- **Speedup: 5-6x for the message schedule and 2-3x for the compression function / 加速比：消息调度 5-6 倍，压缩函数 2-3 倍**

**SHA-256 Implementation Example / SHA-256 实现示例：**
```assembly
# Message schedule / 消息调度: W[i] = σ₁(W[i-2]) + W[i-7] + σ₀(W[i-15]) + W[i-16]
lw       t0, -64(sp)   # W[i-16] / 第 i-16 个字
lw       t1, -60(sp)   # W[i-15] / 第 i-15 个字
sha256sig0 t1, t1      # σ₀(W[i-15]) / 单条指令
lw       t2, -28(sp)   # W[i-7] / 第 i-7 个字
add      t0, t0, t1
lw       t3, -8(sp)    # W[i-2] / 第 i-2 个字
sha256sig1 t3, t3      # σ₁(W[i-2]) / 单条指令
add      t0, t0, t2
add      t0, t0, t3    # Final W[i] / 最终 W[i]
```

---

### Zksed (SM4 Block Cipher) / Zksed（SM4 分组密码）

SM4 is China's national block cipher standard (similar to AES). Zksed provides hardware acceleration for SM4 encryption/decryption.

SM4 是中国国家分组密码标准（类似 AES）。Zksed 为 SM4 加解密提供硬件加速。

**Instructions / 指令：**

| Instruction / 指令 | Encoding / 编码 | Operation / 操作 | Description / 说明 |
|-------------|----------|-----------|-------------|
| `sm4ed rd, rs1, rs2, bs` | `opcode=0110011`<br>`funct7=0011000`<br>`bs[1:0]=funct3[2:1]` | SM4 encryption/decryption round / SM4 加解密轮 | Performs S-box + L transformation on a `bs`-selected byte / 对 `bs` 选择的字节执行 S-box + L 变换 |
| `sm4ks rd, rs1, rs2, bs` | `funct7=0011010` | SM4 key schedule / SM4 密钥扩展 | Generates a round key using S-box + L' transformation / 用 S-box + L' 变换生成轮密钥 |

**Encoding Details / 编码细节：**
- `bs` (byte select): 2-bit field selecting byte 0-3 from `rs2` / `bs`（字节选择）：从 `rs2` 中选择字节 0-3 的 2 位字段
- Both RV32 and RV64 use the same instructions, operating on 32-bit words / RV32 和 RV64 使用相同指令，操作 32 位字

**SM4 vs AES Comparison / SM4 与 AES 对比：**
| Feature / 特性 | SM4 | AES |
|---------|-----|-----|
| Block size / 分组大小 | 128 bits / 128 位 | 128 bits / 128 位 |
| Key size / 密钥长度 | 128 bits only / 仅 128 位 | 128/192/256 bits / 128/192/256 位 |
| Rounds / 轮数 | 32 | 10/12/14 |
| Structure / 结构 | Feistel-like / 类 Feistel | SPN / 代换-置换网络 |
| S-box size / S-box 大小 | 8-bit -> 8-bit / 8 位 -> 8 位 | 8-bit -> 8-bit / 8 位 -> 8 位 |

---

### Zksh (SM3 Hash Function) / Zksh（SM3 哈希函数）

SM3 is China's national cryptographic hash standard (similar to SHA-256, produces 256-bit digest).

SM3 是中国国家密码哈希标准（类似 SHA-256，产生 256 位摘要）。

**Instructions / 指令：**

| Instruction / 指令 | Encoding / 编码 | Operation / 操作 | Description / 说明 |
|-------------|----------|-----------|-------------|
| `sm3p0 rd, rs1` | `opcode=0010011`<br>`funct7=0001000`<br>`rs2=01000`<br>`funct3=001` | P₀(X) = X ⊕ ROL(X,9) ⊕ ROL(X,17) | SM3 permutation P₀ / SM3 置换 P₀ |
| `sm3p1 rd, rs1` | `rs2=01001` | P₁(X) = X ⊕ ROL(X,15) ⊕ ROL(X,23) | SM3 permutation P₁ / SM3 置换 P₁ |

**Performance / 性能：**
- Each P₀/P₁ replaces five instructions (two rotates, two XORs, and result handling) / 每个 P₀/P₁ 替代 5 条指令（两次循环移位、两次异或和结果处理）

---

### Zkr (Entropy Source) / Zkr（熵源）

Provides access to hardware random number generator for cryptographic key generation and nonce creation.

为密码学密钥生成和随机数创建提供硬件随机数生成器访问。

**Instructions / 指令：**

| Instruction / 指令 | Encoding / 编码 | Operation / 操作 | Description / 说明 |
|-------------|----------|-----------|-------------|
| `seed rd` | Custom CSR read via `csrrs rd, seed, x0` / 通过 `csrrs rd, seed, x0` 读取自定义 CSR | Read entropy from the hardware RNG / 从硬件 RNG 读取熵 | Returns entropy data plus a status code / 返回熵数据及状态码 |

**Return Values / 返回值：**
- `rd[31:16]`: Status: BIST=running, WAIT=retry, ES16=16-bit entropy valid, DEAD=failed / 状态：BIST=运行中，WAIT=重试，ES16=16 位熵有效，DEAD=失败
- `rd[15:0]`: 16-bit entropy when status is ES16 / 当状态为 ES16 时的 16 位熵

**Usage / 用法：**
```assembly
retry:
    seed    a0          # Poll entropy source / 轮询熵源
    srli    a1, a0, 16  # Extract status / 提取状态
    li      a2, 0x0002  # ES16 status code / ES16 状态码
    bne     a1, a2, retry
    andi    a0, a0, 0xFFFF  # Extract 16-bit entropy / 提取 16 位熵
```

**Quality Requirements / 质量要求：**
- Must pass continuous health tests under NIST SP 800-90B / 必须通过 NIST SP 800-90B 规定的连续健康测试
- Provides physical entropy, not PRNG output / 提供物理熵，非伪随机数生成器输出

---

### Summary: Cryptography Extensions Performance Impact / 总结：密码学扩展性能影响

| Algorithm / 算法 | Without Extensions / 无扩展 | With Extensions / 有扩展 | Speedup / 加速比 |
|-----------|-------------------|-----------------|---------|
| AES-128 encrypt / AES-128 加密 | ~200 cycles/block / ~200 周期每块 | ~50 cycles/block / ~50 周期每块 | **4x** |
| AES-128 decrypt / AES-128 解密 | ~240 cycles/block / ~240 周期每块 | ~60 cycles/block / ~60 周期每块 | **4x** |
| SHA-256 | ~25 cycles/byte / ~25 周期每字节 | ~10 cycles/byte / ~10 周期每字节 | **2.5x** |
| SHA-512 | ~40 cycles/byte (RV64) / ~40 周期每字节（RV64） | ~15 cycles/byte / ~15 周期每字节 | **2.7x** |
| SM4 | ~180 cycles/block / ~180 周期每块 | ~55 cycles/block / ~55 周期每块 | **3.3x** |
| SM3 | ~28 cycles/byte / ~28 周期每字节 | ~12 cycles/byte / ~12 周期每字节 | **2.3x** |

**Security Benefits / 安全收益：**
- Constant-time execution (Zkt) prevents timing attacks / 常量时间执行（Zkt）防止定时攻击
- No lookup tables eliminates cache side channels / 无查找表消除缓存侧信道
- Hardware entropy (Zkr) supports secure key generation / 硬件熵（Zkr）支持安全密钥生成
- Single-instruction operations reduce software complexity and defects / 单指令操作减少软件复杂性和缺陷

---

## Vector Cryptography Extensions / 向量密码学扩展

### Zvkn / Zvks / Zvk / 向量密码学套件

Vector cryptography extensions that accelerate cryptographic algorithms using vector operations, achieving higher throughput than scalar versions.

使用向量操作加速密码学算法的向量密码学扩展，实现比标量版本更高的吞吐量。

**Extension Hierarchy / 扩展层次：**
- **Zvk** = Zvkn + Zvksc + Zvksg + Zvkb + Zvkt (complete vector crypto / 完整向量密码学)
- **Zvkn** = Zvkned + Zvknhb + Zvkb + Zvkt (NIST algorithms / NIST 算法)
- **Zvks** = Zvksed + Zvksh + Zvkb + Zvkt (ShangMi algorithms / 国密算法)

**Key Vector Crypto Extensions / 关键向量密码扩展：**
- **Zvkned**: Vector AES encryption/decryption / 向量 AES 加解密
- **Zvknhb**: Vector SHA-256 / 向量 SHA-256
- **Zvknhg**: Vector SHA-512 / 向量 SHA-512
- **Zvksed**: Vector SM4 / 向量 SM4
- **Zvksh**: Vector SM3 / 向量 SM3
- **Zvbc**: Vector carryless multiplication for GCM / 向量无进位乘法用于 GCM
- **Zvkb**: Vector bit manipulation for crypto / 向量位操作用于密码学
- **Zvkt**: Vector data-independent execution latency / 向量数据无关执行延迟

**Performance Benefits / 性能优势：**
- Process multiple blocks in parallel / 并行处理多个块
- 4-8x throughput improvement over scalar crypto / 比标量密码学提高 4-8 倍吞吐量
- Suitable for high-throughput applications (VPNs, TLS servers) / 适合高吞吐量应用（VPN、TLS 服务器）

---

## Pointer Masking Extensions / 指针掩码扩展

### Ssnpm / Smnpm / Smmpm / Supm / Sspm (Pointer Masking) / 指针掩码

Ratified October 2024. Provides hardware support for memory tagging and capability-based security by masking high bits of pointers before use in address calculations.

2024年10月批准。通过在地址计算前掩码指针高位，为内存标记和基于能力的安全提供硬件支持。

**Extension Variants / 扩展变体：**

1. **Ssnpm** (Supervisor Naturally-aligned PM): S-mode naturally-aligned power-of-2 masking
2. **Smnpm** (Machine Naturally-aligned PM): M-mode naturally-aligned power-of-2 masking  
3. **Smmpm** (Machine Selectable PM): M-mode with arbitrary mask patterns
4. **Supm** (User Pointer Masking): U-mode pointer masking
5. **Sspm** (Supervisor Selectable PM): S-mode with arbitrary mask patterns

**Mechanism / 机制：**

When pointer masking is enabled:
- High bits of addresses (e.g., bits [63:57] on Sv57) are masked before memory access
- Masked bits can store metadata (tags, capabilities, color bits)
- Hardware automatically applies mask, software sees full pointer value
- Per-privilege-mode enable and configuration

启用指针掩码时：
- 地址高位（例如Sv57上的[63:57]位）在内存访问前被掩码
- 掩码位可存储元数据（标签、能力、颜色位）
- 硬件自动应用掩码，软件看到完整指针值
- 按特权模式启用和配置

**Use Cases / 使用场景：**

1. **Memory Tagging / 内存标记：**
   - Store allocation tags in high pointer bits
   - Hardware checks tag on each access (with additional hardware)
   - Detect use-after-free, buffer overflows
   
   在指针高位存储分配标签
   - 硬件在每次访问时检查标签（需额外硬件）
   - 检测释放后使用、缓冲区溢出

2. **Capability-Based Security / 基于能力的安全：**
   - Encode permissions/bounds in pointer bits (CHERI-like)
   - Compressed capabilities with reduced overhead
   - Fine-grained memory protection
   
   在指针位编码权限/边界（类CHERI）
   - 减少开销的压缩能力
   - 细粒度内存保护

3. **Colored Pointers / 彩色指针：**
   - Store object metadata for GC or memory management
   - Pointer coloring for lock-free data structures
   - Low-fat pointers with metadata
   
   存储对象元数据用于GC或内存管理
   - 无锁数据结构的指针着色
   - 带元数据的低脂指针

4. **Address Sanitization / 地址消毒：**
   - Compiler instrumentation for debugging
   - AddressSanitizer-like functionality
   - Reduced performance overhead vs software-only
   
   编译器插桩用于调试
   - 类AddressSanitizer功能
   - 相比纯软件减少性能开销

**Configuration / 配置：**
- CSRs control mask pattern per privilege mode
- Naturally-aligned variants (npm): Mask is power-of-2 aligned
- Selectable variants (mpm/spm): Arbitrary bit patterns
- Can be enabled/disabled per-mode

CSR按特权模式控制掩码模式
- 自然对齐变体（npm）：掩码是2的幂对齐
- 可选择变体（mpm/spm）：任意位模式
- 可按模式启用/禁用

**Example / 示例：**
```c
// Pointer with metadata in high bits
void* tagged_ptr = (void*)((uintptr_t)ptr | (tag << 57));

// Hardware automatically masks high bits during load/store
int value = *tagged_ptr;  // tag bits ignored by hardware
```

---

## Page-Based Memory Types (Svpbmt) / 基于页的内存类型

### Svpbmt (Page-Based Memory Types) / 基于页的内存类型

Extends page table entries with memory type attributes (similar to x86 PAT or ARM MAIR).

使用内存类型属性扩展页表项（类似 x86 PAT 或 ARM MAIR）。

**Memory Types / 内存类型：**
- **PMA (Normal Cacheable)**: Default / 默认
- **NC (Non-Cacheable)**: Bypass cache / 绕过缓存
- **IO (I/O)**: Strongly ordered, no speculation / 强顺序，无推测

**PTE Encoding / PTE 编码：**
- Uses bits 62:61 in Sv39/Sv48/Sv57 PTEs / 在 Sv39/Sv48/Sv57 PTE 中使用 62:61 位
- `00` = PMA, `01` = NC, `10` = IO, `11` = Reserved / `00` = PMA，`01` = NC，`10` = IO，`11` = 保留

**Applications / 应用：**
- Device MMIO regions (IO type) / 设备 MMIO 区域（IO 类型）
- DMA buffers (NC type) / DMA 缓冲区（NC 类型）
- Performance optimization for specific workloads / 特定工作负载的性能优化

---

## Hypervisor Extensions / 虚拟化扩展

### Svinval (Virtual Memory Invalidation) / 虚拟内存失效

Provides fine-grained TLB invalidation operations, more efficient than SFENCE.VMA for batch invalidations.

提供细粒度 TLB 失效操作，对于批量失效比 SFENCE.VMA 更高效。

**Instructions / 指令：**
- `SINVAL.VMA rs1, rs2`: Invalidate single TLB entry / 使单个 TLB 条目失效
- `SFENCE.W.INVAL`: Ordering barrier before invalidations / 失效前的排序屏障
- `SFENCE.INVAL.IR`: Ordering barrier after invalidations / 失效后的排序屏障

**Benefit / 优势：** Allows batching multiple invalidations without flushing the entire TLB after each one.

允许批量多个失效操作，无需在每次操作后刷新整个 TLB。

---

### Sstc (Supervisor-mode Timer Compare) / 监管者模式定时器比较

Already covered above. Key benefit: Direct S-mode timer access without SBI overhead.

上文已涵盖。关键优势：无需 SBI 开销即可直接访问 S 模式定时器。

---

## Privileged Extensions Summary / 特权扩展摘要

### Smstateen (State Enable) / 状态使能

M-mode controls which state is visible to lower privilege levels, enabling fine-grained feature isolation.

M 模式控制哪些状态对较低特权级可见，实现细粒度特性隔离。

### Smcntrpmf (Counter Privilege Mode Filtering) / 计数器特权模式过滤

Enables privilege-mode filtering for performance counters (count only in specific modes).

为性能计数器启用特权模式过滤（仅在特定模式下计数）。

### Ssccfg (Supervisor Counter Configuration) / 监管者计数器配置

S-mode configuration for counter overflow interrupts and filtering.

S 模式配置计数器溢出中断和过滤。

---

## Debug Extensions / 调试扩展

### Sdtrig (Debug Triggers) / 调试触发器

Already detailed in [[05-调试与工具链/debug/vocabulary]]. Provides hardware breakpoints and watchpoints.

[[05-调试与工具链/debug/vocabulary]] 中已详细说明。提供硬件断点和观察点。

### Sdext (External Debug) / 外部调试

Defines the Debug Module interface for external debuggers (GDB, OpenOCD).

为外部调试器（GDB、OpenOCD）定义调试模块接口。

---

## Control Flow Integrity Extensions / 控制流完整性扩展

### Zicfiss (Shadow Stack) / 影子栈

**Status / 状态：** Ratified 2024 / 2024 年批准

Provides hardware shadow stack for return address protection, defending against ROP (Return-Oriented Programming) attacks.

为返回地址保护提供硬件影子栈，防御 ROP（面向返回编程）攻击。

**Key Concepts / 关键概念：**
- Separate shadow stack stores return addresses / 独立影子栈存储返回地址
- `SSPUSH`/`SSPOP` manage shadow stack / `SSPUSH`/`SSPOP` 管理影子栈
- Return address mismatch triggers exception / 返回地址不匹配触发异常

### Zicfilp (Landing Pad) / 着陆垫

**Status / 状态：** Ratified 2024 / 2024 年批准

Provides forward-edge CFI (Control Flow Integrity) using landing pad instructions.

使用着陆垫指令提供前向边缘 CFI（控制流完整性）。

**Key Instructions / 关键指令：**
- `lpad N`: Landing pad with label N / 标签为 N 的着陆垫
- Indirect jumps must target valid landing pads / 间接跳转必须以有效着陆垫为目标
- Prevents JOP (Jump-Oriented Programming) attacks / 防止 JOP（面向跳转编程）攻击

**Combined Protection / 组合保护：**
- Zicfiss (shadow stack) + Zicfilp (landing pad) = Complete CFI / Zicfiss（影子栈）+ Zicfilp（着陆垫）= 完整 CFI
- Defends against both ROP and JOP attacks / 防御 ROP 和 JOP 攻击

---

## Extension Naming Convention / 扩展命名约定

RISC-V extension names follow a structured pattern:

RISC-V 扩展名遵循结构化模式：

**Prefixes / 前缀：**
- **Z**: ISA extensions (Zicsr, Zifencei, Zba, etc.) / ISA 扩展（Zicsr、Zifencei、Zba 等）
- **S**: Supervisor-level / 监管者级
  - **Ss**: Supervisor extensions / 监管者扩展
  - **Sv**: Virtual memory / 虚拟内存
- **Sm**: Machine-level / 机器级
- **H**: Hypervisor / 虚拟化监控程序
- **X**: Non-standard (vendor-specific) / 非标准（供应商特定）

**Suffixes / 后缀：**
- **32/64**: Applies to RV32/RV64 only / 仅适用于 RV32/RV64
- **e/d/f**: Requires D/F extension / 需要 D/F 扩展

**Examples / 示例：**
- `Zicsr`: Z (ISA), icsr (CSR instructions) / Z（ISA），icsr（CSR 指令）
- `Svpbmt`: Sv (virtual memory), pbmt (page-based memory types) / Sv（虚拟内存），pbmt（基于页的内存类型）
- `Smstateen`: Sm (machine-level), stateen (state enable) / Sm（机器级），stateen（状态使能）

---

## Extension Dependencies / 扩展依赖

Understanding extension dependencies is critical for implementation:

理解扩展依赖对实现至关重要：

**Base Dependencies / 基础依赖：**
- F → Zicsr (floating-point requires CSR access) / F → Zicsr（浮点需要 CSR 访问）
- D → F (double-precision requires single-precision) / D → F（双精度需要单精度）
- V → D + Zve64d + Zvl128b (vector requires FP and min VLEN) / V → D + Zve64d + Zvl128b（向量需要浮点和最小 VLEN）
- H → S + Sv39/48/57 (hypervisor requires S-mode and paging) / H → S + Sv39/48/57（虚拟化监控程序需要 S 模式和分页）

**Crypto Dependencies / 密码学依赖：**
- Zkn → Zkne + Zknd + Zknh + Zbkb + Zbkc + Zbkx
- Zks → Zksed + Zksh + Zbkb + Zbkc + Zbkx
- Zk → Zkn + Zkr + Zkt

**Vector Dependencies / 向量依赖：**
- Zvl*b: Zvl64b → Zvl32b, Zvl128b → Zvl64b (hierarchy / 层次结构)
- Zve64d → Zve64f → Zve32f → Zve32x

---

## Extension Ratification Status / 扩展批准状态

**Ratified (Stable) / 已批准（稳定）：**
- M, A, F, D, C — Base extensions / 基础扩展
- V 1.0 — Vector / 向量
- Zba, Zbb, Zbc, Zbs — Bit manipulation / 位操作
- H — Hypervisor / 虚拟化监控程序
- Zk, Zkn, Zks — Scalar cryptography / 标量密码学
- Sstc, Svpbmt, Svinval — Privileged / 特权
- Zicfiss, Zicfilp — Control flow integrity / 控制流完整性

**Frozen (Near Ratification) / 冻结（接近批准）：**
- Some vector crypto extensions / 一些向量密码扩展
- Additional memory model extensions / 附加内存模型扩展

**Under Development / 开发中：**
- Advanced virtualization features / 高级虚拟化特性
- New bit manipulation extensions / 新位操作扩展
- Hardware transactional memory / 硬件事务内存

**Check official status / 检查官方状态：** https://riscv.org/technical/specifications/

---

## Best Practices for Extension Selection / 扩展选择最佳实践

**For Embedded Systems / 嵌入式系统：**
- Minimum: RV32IMC or RV32IMAC / 最小：RV32IMC 或 RV32IMAC
- Add F/D if floating-point needed / 如需浮点则添加 F/D
- Consider Zve32x for simple vector operations / 考虑 Zve32x 进行简单向量操作
- Use Zicsr + Zifencei for bare-metal / 裸机使用 Zicsr + Zifencei

**For Application Processors / 应用处理器：**
- Minimum: RV64GC (RV64IMAFD_Zicsr_Zifencei) / 最小：RV64GC
- Add V for SIMD workloads / 为 SIMD 工作负载添加 V
- Add H for virtualization / 为虚拟化添加 H
- Consider Zba/Zbb for general performance / 考虑 Zba/Zbb 提高通用性能

**For Security-Critical Systems / 安全关键系统：**
- Add Zk (scalar crypto) or Zvk (vector crypto) / 添加 Zk（标量密码）或 Zvk（向量密码）
- Add Zicfiss + Zicfilp for CFI / 为 CFI 添加 Zicfiss + Zicfilp
- Enable Zkt for constant-time crypto / 为常量时间密码启用 Zkt
- Use Zkr for hardware RNG / 为硬件 RNG 使用 Zkr

**For Servers / 服务器：**
- Minimum: RV64GCV + H / 最小：RV64GCV + H
- Add Zk/Zvk for TLS acceleration / 为 TLS 加速添加 Zk/Zvk
- Add Sstc for low-latency timers / 为低延迟定时器添加 Sstc
- Consider Svpbmt for I/O performance / 考虑 Svpbmt 提高 I/O 性能

---

**Last Updated / 最后更新：** 2026-07-23

---

## Recent Ratified Extensions (2024-2026) / 近期批准扩展

### Zalasr (Atomic Load-Acquire and Store-Release) / 原子加载-获取与存储-释放

Ratified October 2025. Adds acquire/release memory ordering semantics to atomic operations, providing more efficient synchronization than full fences.

2025年10月批准。为原子操作添加获取/释放内存序语义，提供比完全屏障更高效的同步。

**Instructions / 指令：**
- `amoadd.aq` / `amoadd.rl` / `amoadd.aqrl` - Atomic add with acquire/release
- `amoswap.aq` / `amoswap.rl` / `amoswap.aqrl` - Atomic swap with acquire/release
- Similar for all AMO operations (amoand, amoor, amoxor, etc.)
- `lr.aq` / `sc.rl` / `lr.aqrl` / `sc.aqrl` - LR/SC with acquire/release

**Semantics / 语义：**
- `.aq` (acquire): Subsequent memory operations cannot be reordered before this operation
- `.rl` (release): Previous memory operations cannot be reordered after this operation
- `.aqrl`: Both acquire and release semantics

`.aq`（获取）：后续内存操作不能重排到此操作之前
`.rl`（释放）：之前内存操作不能重排到此操作之后
`.aqrl`：同时具有获取和释放语义

**Use Cases / 使用场景：**
- Lock acquisition (acquire semantics)
- Lock release (release semantics)
- Producer-consumer synchronization
- More efficient than FENCE instructions for simple synchronization

锁获取（获取语义）、锁释放（释放语义）、生产者-消费者同步、比FENCE指令更高效用于简单同步

### Zimop / Zcmop (May-Be-Operations) / 可能操作扩展

Ratified March 2024. Reserves instruction encoding space for future custom extensions that may modify architectural state.

2024年3月批准。为未来自定义扩展保留指令编码空间，这些扩展可能修改架构状态。

**Zimop**: Integer May-Be-Operations
**Zcmop**: Compressed May-Be-Operations

**Purpose / 目的：**
- Reserve opcodes that HINT to "may modify state"
- Allows custom extensions without breaking base ISA compatibility
- Standard software treats them as NOPs
- Custom implementations can assign specific behavior

保留"可能修改状态"的HINT操作码
- 允许自定义扩展而不破坏基础ISA兼容性
- 标准软件将其视为NOP
- 自定义实现可分配特定行为

### Zilsd / Zclsd (Load/Store Pair for RV32) / RV32加载/存储对扩展

Ratified February 2025. Adds load/store pair instructions for RV32 to improve code density and performance.

2025年2月批准。为RV32添加加载/存储对指令以提升代码密度和性能。

**Instructions / 指令：**
- `ldd rd1, rd2, offset(rs1)` - Load double (two 32-bit words)
- `sdd rs2, rs3, offset(rs1)` - Store double (two 32-bit words)
- Compressed versions: `c.ldd`, `c.sdd`

**Benefits / 优势：**
- Reduces instruction count for 64-bit data on RV32
- Improves memory bandwidth utilization
- Better code density than separate loads/stores

减少RV32上64位数据的指令数、提升内存带宽利用率、比分离加载/存储更好的代码密度

### Ziccid (Instruction/Data Coherence and Consistency) / 指令/数据一致性扩展

Ratified June 2026. Formalizes instruction and data coherence guarantees for self-modifying code and JIT compilers.

2026年6月批准。正式化自修改代码和JIT编译器的指令和数据一致性保证。

**Key Guarantees / 关键保证：**
- After storing instructions and executing `FENCE.I`, instruction fetch observes the new instructions
- Defines precise ordering between data stores and instruction fetches
- Clarifies interaction with instruction caches and prefetchers

存储指令并执行`FENCE.I`后，指令取指观察到新指令
- 定义数据存储和指令取指之间的精确顺序
- 阐明与指令缓存和预取器的交互

**Impact / 影响：**
- Critical for JIT compilers (JavaScript, Python, Java)
- Enables safe self-modifying code
- Formalizes previously implicit behavior

对JIT编译器关键（JavaScript、Python、Java）、启用安全的自修改代码、正式化之前隐含的行为

