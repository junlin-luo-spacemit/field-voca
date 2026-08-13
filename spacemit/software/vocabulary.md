---
tags: [spacemit, software, vocabulary]
domain: software
date: 2026-08-08
source: https://www.spacemit.com/community/document?lang=zh
---

# 软件平台词汇表

> 涵盖：SDK、操作系统、开发工具链
> 来源：SpacemiT 文档中心 Software 分类

---

## SDK（Software Development Kit）

### SDK / 软件开发套件
SpacemiT 提供的软件开发工具集合，包含编译器、库文件、示例代码。详见 [[SDK]]。

### Cross-Compilation / 交叉编译
在 x86 主机上编译生成 RISC-V 目标代码的开发方式。

---

## 操作系统（Operating Systems）

### Bianbu / 边部操作系统
SpacemiT 为 K 系列芯片优化的 Linux 发行版。详见 [[Bianbu]]。

### OpenWrt / OpenWrt
开源路由器固件，支持 SpacemiT 平台。详见 [[OpenWrt]]。

### Buildroot / Buildroot
嵌入式 Linux 镜像构建工具，用于定制最小化系统。详见 [[Buildroot]]。

### OpenHarmony / 开鸿操作系统
华为主导的开源多终端操作系统，支持 SpacemiT 硬件。详见 [[OpenHarmony]]。

---

## 机器人与边缘（Robotics & Edge）

### ROS2 (Robot Operating System 2) / 机器人操作系统 2
用于机器人开发的中间件框架，SpacemiT 提供 RISC-V 移植。详见 [[ROS2]]。

### Edge Computing / 边缘计算
在靠近数据源的设备上进行计算，减少云端通信延迟。

---

## 开发工具（Development Tools）

### Compiler / 编译器
RISC-V GCC/LLVM 工具链，支持 SpacemiT 扩展指令。

### Debugger / 调试器
GDB、OpenOCD 等调试工具。

### Profiler / 性能分析器
用于分析程序性能瓶颈的工具。

---

## 系统软件（System Software）

### Bootloader / 引导加载程序
系统启动的第一级软件，负责加载操作系统内核。

### Device Driver / 设备驱动
操作系统与硬件之间的接口软件。

### Kernel Module / 内核模块
可动态加载的 Linux 内核扩展。

---

## 参考资料

- [SpacemiT 软件文档](https://www.spacemit.com/community/document?lang=zh)
- SDK 使用指南
- Bianbu 发行说明
