---
tags: [tools, toolchain, cross-compilation, gcc, llvm, spacemit]
domain: tools
aliases: [Cross Compilation Toolchain, 交叉编译工具链, RISC-V Toolchain]
status: approved
created: 2026-08-06
updated: 2026-10-29
source: C:\Users\luojunlin\docs-tool\zh\user_guide\cross_compiler_user_guide.md
superseded_by:
---

# Cross-Compilation Toolchain（交叉编译工具链）

**中文译名**：交叉编译工具链

## 定义

SpacemiT 提供的 RISC-V 交叉编译工具链，用于在 x86_64 主机上编译 RISC-V 目标代码。工具链包含 **GCC** 和 **LLVM** 两套编译器，支持 Linux 环境（glibc）和裸机环境（newlib），并针对 SpacemiT X60、X100、A100 核心进行优化。

## 为什么重要

- **交叉编译必需**：x86 主机编译 RISC-V 代码的核心工具
- **SpacemiT 优化**：针对 X60/X100/A100 核心的调度和 Fusion 优化
- **双编译器支持**：GCC 和 LLVM 任选，共用 C 库
- **完整工具链**：编译器、链接器、调试器、标准库一应俱全
- **RISC-V 扩展支持**：支持 RVV、Bitmanip、Matrix 等扩展

## 工具链类型

| 工具链类型 | C 库 | 适用场景 | 命令前缀 |
|-----------|-----|---------|----------|
| **spacemit-toolchain-linux-glibc-x86_64-xxx** | glibc | Linux 环境开发（Bianbu、Buildroot、OpenWrt 等） | `riscv64-unknown-linux-gnu-` |
| **spacemit-toolchain-elf-newlib-x86_64-xxx** | newlib | 裸机环境开发（小核固件、无 OS 管理固件） | `riscv64-unknown-elf-` |

**说明**：
- 一般发布的工具链版本同时包含 LLVM 和 GCC，共用 C 库
- 特殊版本（仅 GCC）会在下载页面标注

## 下载与安装

### 下载地址

访问 [SpacemiT 资源下载中心](https://www.spacemit.com/community/resources-download/Tools/Cross-compilation%20toolchain)

导航路径：`工具` → `交叉编译工具链` → 选择适合版本下载

### 安装步骤

1. **解压工具链**：
   ```bash
   tar -xf spacemit-toolchain-linux-glibc-x86_64-xxx.tar.xz
   ```

2. **添加到 PATH**：
   ```bash
   export PATH=<工具链解压路径>/bin:$PATH
   ```

3. **验证安装**：
   ```bash
   riscv64-unknown-linux-gnu-gcc --version
   ```

## GCC 工具链使用

### Linux 环境工具链

**编译命令**：
```bash
riscv64-unknown-linux-gnu-gcc -O2 \
  -mcpu=spacemit-x60 \
  -march=rv64gc_zba_zbb_zbc_zbs \
  spacemit.c -o spacemit.out
```

### 裸机环境工具链

**编译命令**：
```bash
riscv64-unknown-elf-gcc -O2 \
  -mcpu=spacemit-x60 \
  -march=rv64gc_zba_zbb_zbc_zbs \
  spacemit.c -o spacemit.out
```

### GCC 编译参数详解

**CPU 优化参数（-mcpu）**：
- **`-mcpu=spacemit-x60`** — 针对 X60 核心（K1 通用 CPU）调度和 Fusion 优化
- **`-mcpu=spacemit-x100`** — 针对 X100 核心（K3 高性能 CPU）调度和 Fusion 优化
- **`-mcpu=spacemit-a100`** — 针对 A100 核心（K3 AI 加速核）调度和 Fusion 优化

**架构扩展参数（-march）**：
- **`rv64gc`** — RISC-V 64-bit，包含 IMAFD 扩展（G = IMAFD，C = 压缩指令）
- **`_zba_zbb_zbc_zbs`** — Bitmanip 扩展（位操作加速）

**示例扩展组合**：
```bash
# 基础配置（仅 G + C）
-march=rv64gc

# 启用 Bitmanip 扩展
-march=rv64gc_zba_zbb_zbc_zbs

# 启用 Vector 扩展（RVV 1.0）
-march=rv64gcv_zba_zbb_zbc_zbs

# 启用 Vector + Matrix 扩展（K1 A60）
-march=rv64gcv_xspacemit-zfh_xspacemit-zfhmin_xspacemit-zvfh_xspacemit-zvfhmin_xspacemit-ime
```

**优化等级**：
- **`-O0`** — 无优化（调试用）
- **`-O2`** — 标准优化（推荐）
- **`-O3`** — 激进优化（可能增大代码体积）
- **`-Os`** — 优化代码大小

更多信息参考 [GCC 官方文档](https://gcc.gnu.org/onlinedocs/)。

## LLVM 工具链使用

### Linux 环境工具链

**使用 clang 命令**：
```bash
clang -O2 \
  -mcpu=spacemit-x60 \
  -march=rv64gc_zba_zbb_zbc_zbs \
  -fuse-ld=lld \
  spacemit.c -o spacemit.out
```

**避免冲突（推荐）**：
```bash
riscv64-unknown-linux-gnu-clang -O2 \
  -mcpu=spacemit-x60 \
  -march=rv64gc_zba_zbb_zbc_zbs \
  -fuse-ld=lld \
  spacemit.c -o spacemit.out
```

### 裸机环境工具链

**使用 clang 命令**：
```bash
riscv64-unknown-elf-clang -O2 \
  -mcpu=spacemit-x60 \
  -march=rv64gc_zba_zbb_zbc_zbs \
  -fuse-ld=lld \
  spacemit.c -o spacemit.out
```

### LLVM 编译参数详解

**CPU 优化参数（-mcpu）**：
- **`-mcpu=spacemit-x60`** — X60 核心调度和 Fusion 优化
- **`-mcpu=spacemit-x100`** — X100 核心调度和 Fusion 优化
- **`-mcpu=spacemit-a100`** — A100 核心调度和 Fusion 优化

**链接器参数**：
- **`-fuse-ld=lld`** — 使用 LLVM lld 链接器（推荐）

**架构扩展参数（-march）**：
- 与 GCC 相同，参见上文

更多信息参考 [LLVM 官方文档](https://llvm.org/docs/)。

## CPU 核心选择指南

| 目标平台 | CPU 核心 | -mcpu 参数 | 说明 |
|---------|---------|-----------|------|
| **K1 通用应用** | X60 | `-mcpu=spacemit-x60` | 8×X60，通用 CPU + AI 扩展（Cluster 0） |
| **K1 AI 推理** | X60 | `-mcpu=spacemit-x60` | 同上（Cluster 0 的 4 个核心带 IME 扩展） |
| **K3 通用应用** | X100 | `-mcpu=spacemit-x100` | 8×X100 @ 2.4GHz，高性能 CPU |
| **K3 AI 推理** | A100 | `-mcpu=spacemit-a100` | 8×A100，专用 AI 加速核心（60 TOPS） |

## 工具链组件

### 编译器
- **GCC** — GNU Compiler Collection
- **LLVM/Clang** — 现代编译器框架

### 链接器
- **GNU ld** — GCC 默认链接器
- **lld** — LLVM 链接器（更快）

### 调试器
- **GDB** — GNU Debugger（RISC-V 支持）

### 标准库
- **glibc** — GNU C Library（Linux 环境）
- **newlib** — 轻量级 C 库（裸机环境）

### 工具
- **objdump** — 反汇编工具
- **readelf** — ELF 文件分析工具
- **addr2line** — 地址到源代码行转换
- **size** — 代码段大小分析

## 使用场景

### Linux 应用开发
- Bianbu 桌面应用
- OpenWrt 网络应用
- Buildroot 嵌入式应用
- OpenHarmony 应用

### 裸机固件开发
- OpenSBI 固件
- U-Boot 引导程序
- 小核管理固件
- RCPU 固件（esos.elf）

### 内核与驱动开发
- Linux 内核编译
- 设备驱动开发
- BSP 适配

### AI 应用开发
- 配合 ONNXRuntime 编译
- Llama.cpp 优化编译
- 自定义 AI 算子

## 与 SpacemiT Studio 对比

| 特性 | Cross-Compilation Toolchain | SpacemiT Studio 云端编译 |
|-----|----------------------------|-------------------------|
| **安装** | 需要本地安装 | 无需安装 |
| **环境配置** | 手动配置 PATH | 自动配置 |
| **编译速度** | 依赖本地硬件 | 云端服务器快速 |
| **跨平台** | 需要不同平台工具链 | 统一 Web 界面 |
| **离线使用** | ✅ 支持 | ❌ 需要网络 |
| **高级优化** | ✅ 完全控制 | ⚠️ 预设配置 |
| **适用人群** | 资深开发者 | 快速上手者 |

**选择建议**：
- **本地开发** → 使用 **Cross-Compilation Toolchain**（完全控制）
- **快速原型** → 使用 **SpacemiT Studio**（零配置）

## 常见问题

### Q: GCC 和 LLVM 如何选择？
**A**: 
- **GCC**：成熟稳定，RISC-V 生态默认选择
- **LLVM**：现代化，更好的错误提示，编译速度更快

两者性能相近，可根据个人偏好选择。

### Q: -mcpu 和 -march 有什么区别？
**A**: 
- **-mcpu**：针对特定 CPU 核心的调度优化（推荐）
- **-march**：启用指令集扩展（功能性）

建议两者同时使用以获得最佳性能。

### Q: 如何启用 RISC-V Vector 扩展？
**A**: 添加 `v` 到 `-march` 参数：
```bash
-march=rv64gcv_zba_zbb_zbc_zbs
```

### Q: 如何启用 K1 Matrix 扩展（IME）？
**A**: 添加完整扩展参数：
```bash
-march=rv64gcv_xspacemit-zfh_xspacemit-zfhmin_xspacemit-zvfh_xspacemit-zvfhmin_xspacemit-ime
```

## 在 SpacemiT 生态中

- **Bianbu 开发**：编译桌面应用、系统工具
- **OpenWrt 开发**：编译网络应用、路由器固件
- **Buildroot 开发**：编译嵌入式应用、最小化系统
- **OpenHarmony 开发**：编译鸿蒙应用、系统组件
- **内核开发**：编译 Linux 内核、驱动模块
- **AI 开发**：编译 ONNXRuntime、Llama.cpp、自定义算子

## 相关词条

- [[SpacemiT-Studio]] — Web 工作台（集成云端编译）
- [[../software/SDK]] — 软件开发套件
- [[../atoms/X60-Core]] — K1 CPU 核心（-mcpu=spacemit-x60）
- [[../atoms/X100-Core]] — K3 CPU 核心（-mcpu=spacemit-x100）
- [[../atoms/A100-AI-Core]] — K3 AI 核心（-mcpu=spacemit-a100）
- [[../atoms/Matrix-Extension]] — RISC-V Matrix 扩展（IME）

## 来源文档

- SpacemiT 交叉编译工具链使用手册：C:\Users\luojunlin\docs-tool\zh\user_guide\cross_compiler_user_guide.md
- 资源下载中心：https://www.spacemit.com/community/resources-download/Tools/Cross-compilation%20toolchain
