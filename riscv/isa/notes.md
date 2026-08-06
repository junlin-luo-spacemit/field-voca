# RISC-V 基础 ISA 学习笔记

> 文档入口：
> - ISA Manual Vol.I (Unprivileged): https://github.com/riscv/riscv-isa-manual
> - psABI (调用约定): https://github.com/riscv-non-isa/riscv-elf-psabi-doc

---

## 为什么 RISC-V 的设计哲学与众不同

RISC-V 最核心的设计决策是**模块化而非大一统**。x86/ARM 为了向后兼容积累了大量历史包袱，而 RISC-V 从一开始就区分"基础 ISA"（稳定不变的最小核心）和"扩展"（可选功能模块）。

这意味着：
- 微控制器实现 `RV32EC`（仅 16 寄存器 + 压缩指令），代码体积极小
- 服务器实现 `RV64GCV`（通用 + 向量），性能最大化
- 两者使用**同一工具链**，软件可按需裁剪

---

## 指令格式的设计逻辑

RISC-V 所有 32 位指令的最低 2 位为 `11`（C 扩展 16 位指令最低 2 位 ≠ `11`，硬件可快速区分），最低 7 位为 opcode。

```
31          25 24      20 19    15 14  12 11       7 6      0
[ funct7      ][  rs2   ][  rs1  ][f3 ][    rd    ][ opcode ]  ← R 格式
[    imm[11:0]          ][  rs1  ][f3 ][    rd    ][ opcode ]  ← I 格式
[ imm[11:5]   ][  rs2   ][  rs1  ][f3 ][ imm[4:0] ][ opcode ]  ← S 格式
```

**符号位始终在 bit[31]**：这是刻意设计——硬件对所有格式做符号扩展时只需看最高位，不需要知道格式类型，节省关键路径延迟。

---

## 内存访问与对齐

RISC-V 基础 ISA 要求自然对齐访问（Load/Store 地址必须是操作数大小的整数倍）。非对齐访问会触发 misaligned 异常，由软件处理（某些实现可选择硬件支持非对齐）。

**Load/Store 宽度指令对照：**
| 指令 | 宽度 | 符号扩展 |
|------|------|---------|
| LB/SB | 8位 | LB符号扩展，LBU零扩展 |
| LH/SH | 16位 | LH符号扩展，LHU零扩展 |
| LW/SW | 32位 | RV64中LW符号扩展，LWU零扩展 |
| LD/SD | 64位 | RV64专属 |

---

## RISC-V 函数调用约定速记

```
调用前：a0–a7 装参数（超出的入栈），fa0–fa7 装浮点参数
调用时：JAL ra, func      # ra = 返回地址
返回时：a0/a1 = 返回值，fa0/fa1 = 浮点返回值
        JALR zero, ra, 0  # 即 ret 伪指令

调用者保存（可随意用）：ra, t0–t6, a0–a7, fa0–fa7, ft0–ft11
被调用者保存（用前压栈）：sp, s0–s11, fs0–fs11
```

---

## 常用伪指令（Pseudo Instructions）

| 伪指令 | 展开为 | 说明 |
|--------|--------|------|
| `nop` | `addi x0, x0, 0` | 空操作 |
| `ret` | `jalr zero, ra, 0` | 函数返回 |
| `call label` | `auipc ra, hi20` + `jalr ra, lo12(ra)` | 远函数调用 |
| `tail label` | `auipc t1, hi20` + `jalr zero, lo12(t1)` | 尾调用（不保存 ra） |
| `la rd, symbol` | `auipc` + `addi`（PIC）或 `lui` + `addi` | 加载符号地址 |
| `li rd, imm` | 根据立即数大小展开为 1–8 条指令 | 加载任意立即数 |
| `mv rd, rs` | `addi rd, rs, 0` | 寄存器复制 |
| `beqz rs, label` | `beq rs, zero, label` | 等于零时跳转 |

---

## 待深入阅读

- [ ] RISC-V ISA Manual Vol.I 全文（指令编码细节）
- [ ] psABI 文档（栈帧布局、TLS、PLT/GOT）
- [ ] RISC-V 压缩指令 C 扩展详细编码规则
- [ ] RVWMO 内存模型形式化规范
