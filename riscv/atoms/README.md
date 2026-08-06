# RISC-V Atomic Concepts / RISC-V 原子概念

> **注意**：原子概念文件已迁移到各自所属的分支文件夹下（如 `01-isa基础/atoms/`），本文件现在只是一个跳转索引。
> **Note**: Atom files have moved into their respective branch folders (e.g. `01-isa基础/atoms/`). This file is now just a jump index.

This page indexes standalone explanatory documents for fundamental RISC-V concepts. Each file provides an in-depth explanation of a single core concept that forms the foundation of RISC-V architecture.

本页索引 RISC-V 基础概念的独立解释文档。每个文件深入讲解一个构成 RISC-V 架构基础的核心概念。

---

## What are Atoms? / 什么是原子概念？

**Atoms** are the building blocks of RISC-V architecture—fundamental concepts that appear repeatedly across specifications and need standalone detailed explanations beyond vocabulary definitions.

**原子概念**是 RISC-V 架构的构建块——在规范中反复出现的基础概念，需要超越词汇表定义的独立详细解释。

**Atoms vs Vocabulary:**
- **Vocabulary files** (in domain folders): Brief definitions, translations, and quick references / 简短定义、翻译和快速参考
- **Atom files** (here): Detailed explanations, examples, and conceptual understanding / 详细解释、示例和概念理解

---

## Core Concepts / 核心概念

### ① ISA 基础 (Base ISA)
- **[hart.md](../01-isa基础/atoms/hart.md)** - Hardware Thread: The basic execution unit in RISC-V / 硬件线程：RISC-V 的基本执行单元
- **[XLEN.md](../01-isa基础/atoms/XLEN.md)** - Integer Register Width (32/64/128-bit) / 整数寄存器宽度（32/64/128 位）
- **[RV32I.md](../01-isa基础/atoms/RV32I.md)** - Base Integer Instruction Set (32-bit) / 基础整数指令集（32 位）

### ② 特权架构 (Privileged Architecture)
- **[privilege-mode.md](../02-特权架构/atoms/privilege-mode.md)** - Privilege Levels (M/S/U modes) / 特权级别（M/S/U 模式）
- **[CSR.md](../02-特权架构/atoms/CSR.md)** - Control and Status Registers / 控制和状态寄存器
- **[trap.md](../02-特权架构/atoms/trap.md)** - Exception and Interrupt Handling / 异常和中断处理
- **[Sv39.md](../02-特权架构/atoms/Sv39.md)** - 39-bit Virtual Addressing / 39 位虚拟地址

### ③ 标准扩展 (Standard Extensions)
- **[extension-M.md](../03-标准扩展/atoms/extension-M.md)** - Integer Multiplication and Division / 整数乘除法
- **[extension-A.md](../03-标准扩展/atoms/extension-A.md)** - Atomic Instructions / 原子指令
- **[extension-C.md](../03-标准扩展/atoms/extension-C.md)** - Compressed Instructions (16-bit) / 压缩指令（16 位）
- **[extension-V.md](../03-标准扩展/atoms/extension-V.md)** - Vector Operations / 向量操作

---

## How to Use / 如何使用

1. **Start here** if you're new to RISC-V and need to understand core concepts / 如果您是 RISC-V 新手并需要理解核心概念，请从这里开始
2. **Reference** when you encounter these terms in specifications or code / 在规范或代码中遇到这些术语时参考
3. **Link** from other documentation using relative paths / 使用相对路径从其他文档链接

**Example usage in other docs / 在其他文档中的使用示例:**
```markdown
For more details on hart, see [[01-isa基础/atoms/hart]]
关于 hart 的更多细节，请参见 [[01-isa基础/atoms/hart]]
```

---

## Suggested Reading Order / 建议阅读顺序

### For Beginners / 初学者
1. **hart.md** - Understand what executes instructions / 理解什么执行指令
2. **XLEN.md** - Understand register width variants / 理解寄存器宽度变体
3. **RV32I.md** - Learn the base instruction set / 学习基础指令集
4. **privilege-mode.md** - Understand security levels / 理解安全级别
5. **CSR.md** - Learn about system control / 学习系统控制

### For System Programmers / 系统程序员
1. **privilege-mode.md** - Essential for OS development / OS 开发必备
2. **trap.md** - Exception/interrupt handling / 异常/中断处理
3. **Sv39.md** - Virtual memory management / 虚拟内存管理
4. **CSR.md** - System register programming / 系统寄存器编程

### For Application Developers / 应用开发者
1. **hart.md** - Threading model / 线程模型
2. **extension-C.md** - Code density optimization / 代码密度优化
3. **extension-A.md** - Lock-free programming / 无锁编程
4. **extension-V.md** - SIMD performance / SIMD 性能

---

## Related Resources / 相关资源

- **概念地图**: See [[../概念地图|Concept Map]] for how these atoms fit into the five branches / 查看[[../概念地图|概念地图]]了解这些原子概念如何嵌套在五个分支中
- **Vocabulary files**: See branch folders (01-isa基础/, 02-特权架构/, etc.) for term definitions / 词汇表文件：查看分支文件夹（01-isa基础/、02-特权架构/ 等）中的术语定义
- **RISC-V ISA Manual**: https://github.com/riscv/riscv-isa-manual
- **RISC-V Privileged Spec**: https://github.com/riscv/riscv-isa-manual

---

**Last Updated / 最后更新:** 2026-08-06
