---
tags: [tools, debug, jtag, spacemit]
domain: tools
aliases: [JTAG Debugging, JTAG调试, JTAG]
status: approved
created: 2026-08-06
updated: 2026-10-29
source: C:\Users\luojunlin\docs-buildroot\zh\k1_buildroot\device\
superseded_by:
---

# JTAG Debugging（JTAG 调试）

**中文译名**：JTAG 调试

## 定义

基于 JTAG 接口的低级调试方法，用于单步执行、断点设置与寄存器级调试，可直接访问 CPU 内核状态。

## 为什么重要

- **最底层调试**：比软件调试器更接近硬件
- **引导调试**：可调试 bootloader、内核早期启动
- **无需 OS**：裸机程序调试
- **硬件故障诊断**：CPU/内存/总线问题排查

## JTAG 功能

1. **调试功能**
   - 单步执行
   - 断点设置（硬件断点）
   - 寄存器读写
   - 内存访问
   - CPU 暂停/恢复

2. **烧录功能**
   - Flash 编程
   - Bootloader 烧录
   - eFuse 编程

3. **测试功能**
   - Boundary Scan（边界扫描）
   - 芯片连接测试

## 工具链

- **调试器硬件**：J-Link、OpenOCD 兼容调试器
- **软件工具**：OpenOCD、GDB
- **IDE 集成**：Eclipse、VS Code + Cortex-Debug

## 调试流程

```
JTAG 调试器（J-Link）→ JTAG 接口（20-pin）→ K1/K3 芯片
                    ↑
                 OpenOCD + GDB
```

## 在 SpacemiT 中

- K1/K3 提供 JTAG 接口（多路 JTAG 支持）
- 用于 SoC 引导调试、内核调试、裸机程序调试
- 常用于开发阶段的底层问题排查

## 与 Trace Tool 的区别

| 特性 | JTAG Debugging | Trace Tool |
|------|----------------|------------|
| 功能 | 暂停、断点、单步 | 记录执行流程 |
| 性能影响 | 暂停程序运行 | 不影响实时性 |
| 适用场景 | 逐行调试 | 性能分析、热点定位 |

## 相关词条

- [[Trace-Tool]] — 跟踪分析工具
- [[Flashing-Tool]] — 烧录工具
- [[Cross-Compilation-Toolchain]] — 交叉编译工具链
- [[../hardware/products/K1]] — K1 JTAG 接口
- [[../hardware/products/K3]] — K3 JTAG 接口

## 来源文档

- SpacemiT 文档中心：Tools / JTAG Debugging Tool User Guide
- K1/K3 User Manual：调试接口章节
