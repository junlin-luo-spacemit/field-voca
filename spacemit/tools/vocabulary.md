---
tags: [spacemit, tools, vocabulary]
domain: tools
date: 2026-08-08
source: C:\Users\luojunlin\docs-tool\zh\
---

# 开发工具词汇表

> 涵盖：工具链、调试工具、IDE
> 来源：SpacemiT 文档中心 Tools 分类

---

## 编译工具链（Compilation Toolchain）

### Cross-Compilation Toolchain / 交叉编译工具链
在 x86 主机上编译 RISC-V 目标代码的工具集合。详见 [[Cross-Compilation-Toolchain]]。

### GCC (GNU Compiler Collection) / GNU 编译器套件
支持 RISC-V 的开源编译器。

### LLVM / LLVM
现代编译器基础设施，支持 RISC-V 后端。

---

## 烧录工具（Flashing Tools）

### Flashing Tool / 刷机工具
用于将固件烧录到 SpacemiT 设备的工具。详见 [[Flashing-Tool]]。

### Bootloader Update / 引导程序更新
更新设备的启动引导程序。

---

## 调试工具（Debugging Tools）

### JTAG Debugging / JTAG 调试
通过 JTAG 接口进行芯片级调试。详见 [[JTAG-Debugging]]。

### GDB (GNU Debugger) / GNU 调试器
命令行调试工具，支持 RISC-V。

### OpenOCD / OpenOCD
开源片上调试器，支持 JTAG 和 SWD。

---

## 性能分析（Performance Analysis）

### Trace Tool / 跟踪工具
记录和分析程序执行流程的工具。详见 [[Trace-Tool]]。

### Profiler / 性能分析器
分析程序性能瓶颈，识别热点函数。

---

## 集成开发环境（IDE）

### SpacemiT Studio / SpacemiT Studio
SpacemiT 官方或推荐的集成开发环境。详见 [[SpacemiT-Studio]]。

### VS Code
支持通过插件进行 RISC-V 开发的轻量级编辑器。

---

## 参考资料

- [SpacemiT 工具文档](https://www.spacemit.com/community/document?lang=zh)
- 工具链使用指南
- 调试最佳实践
