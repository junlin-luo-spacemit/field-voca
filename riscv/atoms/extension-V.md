---
tags: [riscv, atom, extensions]
domain: extensions
aliases: [V extension, 向量扩展, SIMD]
source: https://github.com/riscv/riscv-v-spec
date: 2026-07-22
---

# V Extension (Vector Operations) / extension-V（向量扩展）

**English Name:** Vector Extension 1.0 (Ratified)  
**中文译名**：向量扩展 1.0（已批准）

## Definition / 定义

**English:**  
Adds SIMD vector computing capability to RISC-V. The core feature is **Variable Vector Length**—the same code can run correctly on processors with different VLEN, without needing to be rewritten for each SIMD width.

**中文:**  
为 RISC-V 添加 SIMD 向量计算能力，核心特点是**可变向量长度（Variable Vector Length）**——同一段代码可在不同 VLEN 的处理器上正确运行，无需针对每种 SIMD 宽度重写。

## Hardware Resources / 硬件资源

**English:** 32 vector registers (v0–v31), each VLEN bits wide (implementation-defined, minimum 128 bits).

**中文:** 32 个向量寄存器（v0–v31），每个宽度为 VLEN 位（实现决定，最小 128 位）。

## Three Core Programming Model Elements / 编程模型核心三要素

```
SEW  (Selected Element Width)  Current element width: 8/16/32/64
                                当前元素位宽: 8/16/32/64
LMUL (Length Multiplier)       Register group multiplier: 1/2/4/8 or 1/2/1/4/1/8
                                寄存器组倍增: 1/2/4/8 或 1/2/1/4/1/8
vl   (Vector Length)           Actual elements processed this round (returned by vsetvli)
                                本轮实际处理元素数（由 vsetvli 返回）
```

## Strip-Mining Vectorization Pattern / Strip-mining 向量化模式

```c
for (size_t i = 0; i < n; ) {
    size_t vl = vsetvli(n - i, e32, m1);  // Set SEW=32, LMUL=1 / 设置 SEW=32, LMUL=1
    vfloat32m1_t va = vle32_v_f32m1(a + i, vl);
    vfloat32m1_t vb = vle32_v_f32m1(b + i, vl);
    vfloat32m1_t vc = vfadd_vv_f32m1(va, vb, vl);
    vse32_v_f32m1(c + i, vc, vl);
    i += vl;
}
```

## Vector Operation Categories / 向量操作类别

- Integer arithmetic / 整数算术: vadd/vsub/vmul/vdiv/vshl/vshr...
- Floating-point / 浮点运算: vfadd/vfmul/vfmacc (FMA)...
- Reduction / 规约: vredsum/vredmax (reduce vector to scalar / 向量归约为标量)
- Memory / 内存: vle/vse (unit-stride / 单步), vlse/vsse (strided / 步长), vluxei/vsuxei (scatter/gather)
- Masked operations / 掩码操作: Conditional per-element execution using v0.t / 以 v0.t 为条件，逐元素选择性执行

## Related Terms / 相关词条

[[03-标准扩展/vocabulary#V Extension]], [[03-标准扩展/vocabulary#vsetvli]]

## Source / 来源文档

RISC-V V Extension Specification v1.0
