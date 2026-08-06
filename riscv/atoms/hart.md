---
tags: [riscv, atom, isa]
domain: isa
aliases: [hart, hardware thread]
source: https://github.com/riscv/riscv-isa-manual
date: 2026-07-22
---

# hart (Hardware Thread) / hart（硬件线程）

**English Name:** Hardware Thread  
**中文译名**：硬件线程

## Definition / 定义

**English:**  
The basic execution unit in RISC-V specification. Each hart has its own program counter (PC), general-purpose register set, and CSRs, capable of independently fetching, decoding, and executing instructions.

**中文:**  
RISC-V 规范中的基本执行单元。每个 hart 拥有独立的程序计数器（PC）、通用寄存器组和 CSR，可以独立地取指、译码、执行指令。

**Official Quote / 官方原文：**
> A RISC-V compatible core might support multiple RISC-V-compatible hardware threads, or harts, sharing a single physical processor.

## Key Features / 核心特征

- Each hart has its own PC and register set, executing independently / 每个 hart 有自己的 PC 和寄存器组，彼此独立执行
- Multiple harts can share a single physical processor core (similar to SMT hyperthreading) / 多个 hart 可以共享同一物理处理器核（类似 SMT 超线程）
- `mhartid` CSR returns the unique hart ID (starting from 0) / `mhartid` CSR 返回当前 hart 的唯一编号（从 0 开始）
- During multi-core boot, hartid=0 is typically the primary core, other harts wait for SBI HSM calls to wake up / 多核启动时，hartid=0 通常是主核，其他 hart 等待 SBI HSM 调用唤醒

## Distinction from CPU Core / 与 CPU 核心的区别

- "Core" is a physical concept (computing unit on a die) / "核心"（core）是物理概念（一个 die 上的计算单元）
- "hart" is an ISA-level abstract execution unit / "hart"是 ISA 层面的抽象执行单元
- A physical core can contain multiple harts (hardware multithreading), but embedded cores typically have 1 core = 1 hart / 一个物理核可包含多个 hart（硬件多线程），但嵌入式核通常 1 核 = 1 hart

## Related Terms / 相关词条

[[01-isa基础/vocabulary#hart]], [[02-特权架构/vocabulary#mhartid]]

## Source / 来源文档

RISC-V ISA Manual Volume I, Chapter 1
