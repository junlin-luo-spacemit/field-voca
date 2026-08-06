---
tags: [home, index, riscv]
domain: all
date: 2026-07-22
---

# RISC-V 词汇知识库

> 芯片行业文档工程师学习用知识库 · 来源：[riscv.org](https://riscv.org/)
> 进度追踪：[[PROGRESS]] · 规范索引：[[glossary/master_glossary]]

---

## 🌳 不知道从哪看起？先看这个

**[[概念地图]]** — 一张图讲清楚所有分支之间的关系，从最简单的心智模型开始，逐层深入到具体词条。5分钟建立全局框架。

---

## 核心原子词条

> RISC-V 最重要的基础概念，每个都有独立深度页面。

## 小白分步学习

按由浅入深的顺序学习整理好的内容：[[learning_path/README|RISC-V 分步学习路线]]

### ISA 基础
- [[atoms/RV32I]] — 32位基础整数指令集，RISC-V 最小完备子集
- [[atoms/RV64I]] — 64位基础整数指令集
- [[atoms/XLEN]] — 整数寄存器宽度，定义地址空间与操作数位宽
- [[atoms/hart]] — Hardware Thread，RISC-V 的基本执行单元

### 特权架构
- [[atoms/privilege-mode]] — M/S/U 三级特权模式
- [[atoms/CSR]] — Control and Status Register，控制与状态寄存器
- [[atoms/trap]] — 异常与中断的统一处理机制
- [[atoms/Sv39]] — 64位虚拟内存分页方案（3级页表）

### 重要扩展
- [[atoms/extension-M]] — 整数乘除法扩展
- [[atoms/extension-A]] — 原子操作扩展
- [[atoms/extension-V]] — 向量扩展（SIMD）
- [[atoms/extension-C]] — 压缩指令扩展（16位编码）

---

## 领域词汇表

| 领域 | 词汇表 | 笔记 | 词条数 | 状态 |
|------|--------|------|--------|------|
| 基础 ISA | [[isa/vocabulary\|ISA 词汇]] | [[isa/notes\|ISA 笔记]] | — | ⬜ 待学习 |
| 特权架构 | [[privileged/vocabulary\|特权架构词汇]] | [[privileged/notes\|特权架构笔记]] | — | ⬜ 待学习 |
| 标准扩展 | [[extensions/vocabulary\|扩展词汇]] | [[extensions/notes\|扩展笔记]] | — | ⬜ 待学习 |
| 微架构 | [[microarch/vocabulary\|微架构词汇]] | [[microarch/notes\|微架构笔记]] | — | ⬜ 待学习 |
| 平台规范 | [[platform/vocabulary\|平台词汇]] | [[platform/notes\|平台笔记]] | — | ⬜ 待学习 |
| 软件生态 | [[software/vocabulary\|软件词汇]] | [[software/notes\|软件笔记]] | — | ⬜ 待学习 |
| 调试与跟踪 | [[debug/vocabulary\|调试词汇]] | [[debug/notes\|调试笔记]] | — | ⬜ 待学习 |
| 全局词汇 | [[glossary/master_glossary\|Master Glossary]] | — | — | — |

---

## 学习进度

| 领域 | 文档 | 状态 | 完成日期 |
|------|------|------|----------|
| 基础 ISA | The RISC-V ISA Specification Volume I: Unprivileged | ⬜ | — |
| 特权架构 | The RISC-V ISA Specification Volume II: Privileged | ⬜ | — |
| 标准扩展 | Ratified Extensions（M/A/F/D/C/V/B/Zicsr 等） | ⬜ | — |
| 微架构 | — | ⬜ | — |
| 平台规范 | RISC-V Platform Specification + SBI Specification | ⬜ | — |
| 软件生态 | OpenSBI / GCC / LLVM / Linux / QEMU | ⬜ | — |
| 调试与跟踪 | RISC-V External Debug Specification | ⬜ | — |

---

## 快速参考

**寄存器**：32 个通用整数寄存器 x0–x31（x0 恒为 0） · ABI 名：zero/ra/sp/gp/tp/t0-t6/s0-s11/a0-a7

**基础 ISA 命名规则**：`RV[XLEN][扩展字母]`，例如 `RV64GC` = RV64I + G(=IMAFD) + C

**特权模式**：M（Machine）> S（Supervisor）> U（User） · 所有实现必须支持 M 模式

**指令格式（6种）**：R · I · S · B · U · J

**关键规范来源**：
- ISA 规范：https://riscv.org/technical/specifications/
- 技术 Wiki：https://lf-riscv.atlassian.net/wiki/spaces/HOME/
- GitHub：https://github.com/riscv
