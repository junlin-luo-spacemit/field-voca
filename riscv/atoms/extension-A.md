---
tags: [riscv, atom, extensions]
domain: extensions
aliases: [A extension, 原子操作扩展]
source: https://github.com/riscv/riscv-isa-manual
date: 2026-07-22
---

# A Extension (Atomic Instructions) / extension-A（原子操作扩展）

**English Name:** Atomic Instructions Extension  
**中文译名**：原子操作扩展

## Definition / 定义

**English:**  
Provides atomic instructions needed to implement mutexes, lock-free data structures, and memory order synchronization—the foundation of multi-core programming.

**中文:**  
提供实现互斥锁、无锁数据结构、内存序同步所需的原子指令，是多核编程的基础。

## Two Atomic Mechanisms / 两类原子机制

### 1. LR/SC (Load-Reserved / Store-Conditional)

**English:** Optimistic locking primitive that implements complex atomic operations like CAS (Compare-And-Swap):

**中文:** 乐观锁原语，实现 CAS（Compare-And-Swap）等复杂原子操作：

```asm
# Implement atomic compare-and-swap / 实现 atomic compare-and-swap
cas:
  lr.w    t0, (a0)        # Load and set reservation / 加载并设置保留
  bne     t0, a1, fail    # Exit if mismatch / 不匹配则退出
  sc.w    t1, a2, (a0)    # Conditional write / 条件写入
  bnez    t1, cas         # Retry on failure / 失败则重试
fail:
  mv      a0, t0
  ret
```

### 2. AMO (Atomic Memory Operations)

**English:** Single instruction completes read-modify-write, suitable for counters and bitmap updates:

**中文:** 单指令完成读-改-写，适合计数器、位图更新：

```asm
amoadd.w  t0, a1, (a0)   # t0←old value, *a0 += a1 (atomic) / t0←旧值，*a0 += a1（原子）
amoswap.w t0, a1, (a0)   # t0←old value, *a0 = a1 (atomic swap) / t0←旧值，*a0 = a1（原子交换）
```

## Memory Order Modifiers / 内存序修饰符

- `.aq` (acquire): Reads/writes after this operation cannot be reordered before it / 此操作之后的读写不能被重排到此操作之前
- `.rl` (release): Reads/writes before this operation cannot be reordered after it / 此操作之前的读写不能被重排到此操作之后
- `.aqrl`: Sequentially consistent (strongest) / 顺序一致（最强）

## Related Terms / 相关词条

[[isa/vocabulary#LR/SC]], [[isa/vocabulary#AMO]], [[extensions/vocabulary#A Extension]]

## Source / 来源文档

RISC-V ISA Manual Volume I, Chapter 8
