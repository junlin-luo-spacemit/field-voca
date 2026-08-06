---
tags: [riscv, software, vocabulary]
domain: software
last_updated: 2026-07-22
source: https://riscv.org/
---

# RISC-V Software Ecosystem Vocabulary / RISC-V 软件生态词汇表

> Covers: Toolchains, Operating Systems, Firmware, Emulators, Compiler Backends
> Sources: https://riscv.org/developers/ · https://github.com/riscv

> 涵盖：工具链、操作系统、固件、仿真器、编译器后端
> 来源：https://riscv.org/developers/ · https://github.com/riscv

---

## Toolchains / 工具链

### GNU toolchain / GNU 工具链

The combination of `binutils + GCC + glibc/newlib` is the most mature RISC-V toolchain. Two primary variants exist: `riscv64-unknown-elf-gcc` for bare-metal/embedded development (using newlib, no OS), and `riscv64-linux-gnu-gcc` for Linux userspace applications (using glibc).

`binutils + GCC + glibc/newlib` 的组合，是 RISC-V 最成熟的工具链。存在两个主要变体：`riscv64-unknown-elf-gcc` 用于裸机/嵌入式开发（使用 newlib，无操作系统），`riscv64-linux-gnu-gcc` 用于 Linux 用户态应用程序（使用 glibc）。

### LLVM / Clang RISC-V backend / LLVM / Clang RISC-V 后端

LLVM has supported RISC-V since version 7.0, with Clang serving as an alternative to GCC. LLVM sometimes provides faster support for new extensions compared to GCC, and offers comprehensive intrinsics support for the Vector extension (V).

LLVM 从 7.0 版本起支持 RISC-V，Clang 可作为 GCC 的替代编译器。LLVM 对新扩展的支持有时比 GCC 更快，向量扩展（V）的 intrinsics 支持较完善。

### binutils / binutils

A collection of binary utilities including the assembler (`as`), linker (`ld`), and object file tools (`objdump`/`objcopy`/`readelf`). The RISC-V assembler supports all standard extensions and pseudo-instructions.

包含汇编器（`as`）、链接器（`ld`）、目标文件工具（`objdump`/`objcopy`/`readelf`）的二进制工具集合。RISC-V 汇编器支持所有标准扩展和伪指令。

### riscv-gnu-toolchain / riscv-gnu-toolchain

Official GNU toolchain build repository (https://github.com/riscv-collab/riscv-gnu-toolchain) that enables one-command compilation of newlib or linux toolchains.

官方 GNU 工具链构建仓库（https://github.com/riscv-collab/riscv-gnu-toolchain），一键编译 newlib 或 linux 工具链。

### psABI (Processor-Specific ABI) / 处理器特定 ABI

The RISC-V ELF ABI specification document defining data type sizes and alignment, calling conventions, stack frame layout, TLS models, and PLT/GOT formats. Repository: https://github.com/riscv-non-isa/riscv-elf-psabi-doc

RISC-V ELF ABI 规范文档，定义数据类型大小对齐、调用约定、栈帧布局、TLS 模型、PLT/GOT 格式。仓库：https://github.com/riscv-non-isa/riscv-elf-psabi-doc

### ABI variants / ABI 变体

| ABI Name | Integer Width | Float Passing |
|----------|---------------|---------------|
| `ilp32` | 32-bit | Soft-float |
| `ilp32f` | 32-bit | f registers (single-precision) |
| `ilp32d` | 32-bit | f registers (double-precision) |
| `lp64` | 64-bit | Soft-float |
| `lp64f` | 64-bit | f registers (single-precision) |
| `lp64d` | 64-bit | f registers (double-precision) **← Linux default** |

| ABI 名 | 整数宽度 | 浮点传参 |
|--------|---------|---------|
| `ilp32` | 32位 | 软浮点 |
| `ilp32f` | 32位 | f 寄存器（单精度） |
| `ilp32d` | 32位 | f 寄存器（双精度） |
| `lp64` | 64位 | 软浮点 |
| `lp64f` | 64位 | f 寄存器（单精度） |
| `lp64d` | 64位 | f 寄存器（双精度）**← Linux 默认** |

### march / mabi / mtune / 编译器目标选项

Compiler target options: `-march=rv64gc` specifies the target instruction set (which instructions to generate), `-mabi=lp64d` specifies the ABI (how parameters are passed), and `-mtune=sifive-u74` specifies the tuning target (instruction scheduling strategy) without affecting correctness.

编译器目标选项：`-march=rv64gc` 指定目标指令集（生成哪些指令），`-mabi=lp64d` 指定 ABI（参数如何传递），`-mtune=sifive-u74` 指定调优目标（指令调度策略），不影响正确性。

---

## Operating Systems / 操作系统

### Linux RISC-V / Linux on RISC-V

Mainline Linux officially supports RISC-V (RV64GC) starting from version 4.15. The primary architecture code resides in `arch/riscv/`, with support for Sv39/Sv48 virtual memory, SBI interface, and multiple interrupt controllers (PLIC/ACLINT/AIA).

Linux 主线从 4.15 版本开始正式支持 RISC-V（RV64GC）。主要架构代码在 `arch/riscv/`，支持 Sv39/Sv48 虚拟内存、SBI 接口、多种中断控制器（PLIC/ACLINT/AIA）。

### OpenSBI / OpenSBI

See [[05-调试与工具链/platform/vocabulary]] for details.

详见 [[05-调试与工具链/platform/vocabulary]]。

### RTEMS / RTEMS

A real-time operating system supporting RISC-V, designed for embedded and aerospace applications.

实时操作系统，已支持 RISC-V，用于嵌入式/航空航天场景。

### FreeRTOS RISC-V port / FreeRTOS RISC-V 移植

A lightweight RTOS supporting RV32/RV64 that runs directly in M-mode (without OS kernel), suitable for resource-constrained embedded systems. Context switching is implemented via `mret`.

轻量级 RTOS，支持 RV32/RV64，直接运行在 M 模式（无需操作系统内核），适合资源受限嵌入式系统。上下文切换通过 `mret` 实现。

### Zephyr RTOS / Zephyr RTOS

A Linux Foundation embedded RTOS with official support for multiple RISC-V development boards (SiFive, ESP32-C3, etc.).

Linux 基金会的嵌入式 RTOS，官方支持多种 RISC-V 开发板（SiFive、ESP32-C3 等）。

---

## Emulation and Virtualization / 仿真与虚拟化

### QEMU RISC-V / QEMU RISC-V

QEMU supports multiple RISC-V machine models: `qemu-system-riscv64 -machine virt` for generic virtual machines (with PLIC and VirtIO support), `qemu-system-riscv32 -machine sifive_e` for SiFive E-series MCU emulation, and `qemu-riscv64` for user-mode emulation (running RISC-V ELF binaries without a full system).

QEMU 支持多种 RISC-V 机器模型：`qemu-system-riscv64 -machine virt` 用于通用虚拟机（支持 PLIC、VirtIO），`qemu-system-riscv32 -machine sifive_e` 用于 SiFive E 系列 MCU 仿真，`qemu-riscv64` 用于用户态仿真（运行 RISC-V ELF 二进制文件不需完整系统）。

### spike / spike

The official RISC-V ISA reference simulator with precise behavioral modeling, supporting all standard extensions. Works with pk (Proxy Kernel) to run bare-metal programs.

RISC-V 官方 ISA 参考模拟器，行为建模精确，支持所有标准扩展。与 pk（Proxy Kernel）配合可运行裸机程序。

### pk (Proxy Kernel) / 代理内核

A lightweight runtime that operates on top of spike, providing minimal system calls for applications (file/console I/O) without requiring a full Linux installation. Ideal for quickly testing bare-metal programs.

轻量级运行时，运行在 spike 之上，为应用程序提供最小化系统调用（文件/控制台 I/O），无需完整 Linux。适合快速测试 bare-metal 程序。

### Verilator / Verilator

A tool that converts Verilog/SystemVerilog RTL into cycle-accurate C++ simulators, widely used in RISC-V processor development (Rocket Chip, CVA6, PicoRV32).

将 Verilog/SystemVerilog RTL 转换为 C++ 周期精确模拟器的工具，RISC-V 处理器（Rocket Chip、CVA6、PicoRV32）开发中广泛使用。

---

## 处理器实现生态

### Rocket Chip / Rocket Chip
UC Berkeley 开源的 RISC-V 生成器框架（Chisel 语言），参数化生成不同规格的处理器，是学术研究和工业应用的重要参考实现。

### BOOM (Berkeley Out-of-Order Machine) / BOOM
基于 Rocket Chip 框架的乱序超标量 RISC-V 处理器开源实现，用于研究和高性能场景。

### CVA6 (Ariane) / CVA6
OpenHW Group 维护的开源 RISC-V 应用处理器核（RV64GC），支持 Linux，已流片，用于嵌入式到汽车场景。

### PicoRV32 / PicoRV32
极小面积的开源 RV32IMC 软核，适合 FPGA 嵌入和面积极度受限的 ASIC 场景。

### SiFive / SiFive
第一家基于 RISC-V 的商业芯片公司（RISC-V 发明者创办），提供 E/S/U/P 系列处理器 IP 核，以及 HiFive 开发板。

### T-Head (平头哥) / T-Head（平头哥半导体）
阿里巴巴旗下芯片设计公司，推出 Xuantie（玄铁）系列 RISC-V 处理器（C906/C910/C920），已被多个 SoC 厂商采用，并贡献 Linux 主线支持。

---

## 软件生态工具

### riscv-tests / riscv-tests
→ 详见 [[05-调试与工具链/debug/vocabulary]]

### riscv-arch-test / 架构一致性测试
RISC-V International 维护的合规性测试框架，验证处理器实现是否符合规范，通过即可申请 RISC-V 兼容性认证。

### sail-riscv / sail-riscv
用 Sail（形式化规范语言）编写的 RISC-V ISA 形式化模型，是 RISC-V 规范的机器可读版本，可生成模拟器并用于形式化验证。

### GDB multiarch / GDB multiarch
支持多架构的 GDB 版本，可在 x86 主机上调试 RISC-V 目标（通过 OpenOCD 远程连接或 QEMU GDB stub）。

### objdump / objdump
反汇编和目标文件分析工具，`riscv64-unknown-elf-objdump -d` 可反汇编 RISC-V ELF 文件，显示每条指令的助记符。

---

## C Standard Libraries / C 标准库

### glibc (GNU C Library) / GNU C 库

The standard C library for Linux systems, provides POSIX-compliant system call wrappers, pthread threading, mathematical functions, and locale support. RISC-V glibc support added in version 2.27.

Linux 系统的标准 C 库，提供 POSIX 兼容的系统调用包装、pthread 线程、数学函数、locale 支持。RISC-V glibc 支持在 2.27 版本加入。

**Key Components / 关键组件：**
- **libc.so**: Core C library / 核心 C 库
- **libm.so**: Math library / 数学库
- **libpthread.so**: Threading library / 线程库
- **libdl.so**: Dynamic loading / 动态加载
- **librt.so**: Real-time extensions / 实时扩展

**Dynamic Linker / 动态链接器：**
- `/lib/ld-linux-riscv64-lp64d.so.1`: Dynamic linker for RV64 / RV64 的动态链接器
- `/lib/ld-linux-riscv32-ilp32d.so.1`: Dynamic linker for RV32 / RV32 的动态链接器

---

### newlib / newlib

Lightweight C library designed for embedded systems and bare-metal environments, smaller footprint than glibc but with reduced functionality.

为嵌入式系统和裸机环境设计的轻量级 C 库，比 glibc 占用空间小但功能减少。

**Features / 特性：**
- No OS dependencies / 无操作系统依赖
- Configurable features / 可配置功能
- Reentrant implementations / 可重入实现
- Minimal system call requirements / 最小系统调用要求

**Use Cases / 使用场景：**
- Bare-metal applications / 裸机应用
- RTOS environments / RTOS 环境
- Bootloaders and firmware / Bootloader 和固件

---

### musl libc / musl C 库

Lightweight, fast, and simple C standard library alternative to glibc, popular in embedded Linux and Alpine Linux.

轻量、快速、简单的 C 标准库，glibc 的替代品，在嵌入式 Linux 和 Alpine Linux 中流行。

**Advantages / 优势：**
- Smaller binary size / 更小的二进制大小
- Faster compilation / 更快的编译
- Better code quality / 更好的代码质量
- Static linking friendly / 静态链接友好

**RISC-V Support / RISC-V 支持：**
- Full RV32/RV64 support / 完整 RV32/RV64 支持
- Both hard and soft float ABIs / 硬浮点和软浮点 ABI

---

### picolibc / picolibc

Modern embedded C library combining newlib and avr-libc features, optimized for small footprint.

现代嵌入式 C 库，结合 newlib 和 avr-libc 特性，针对小占用空间优化。

**Features / 特性：**
- Smaller than newlib / 比 newlib 更小
- Modern C standards (C11/C17) / 现代 C 标准（C11/C17）
- Better embedded support / 更好的嵌入式支持
- TLS (Thread Local Storage) support / TLS（线程本地存储）支持

---

## Compiler Optimization / 编译器优化

### ISA String Canonicalization / ISA 字符串规范化

RISC-V ISA strings follow a canonical order: base (I/E) → standard single-letter extensions (M, A, F, D, C) → Z* extensions → S* extensions → X* extensions. Example: `rv64imafdcv` → `rv64imafdc_zve64x_zvl128b`.

RISC-V ISA 字符串遵循规范顺序：基础（I/E）→ 标准单字母扩展（M、A、F、D、C）→ Z* 扩展 → S* 扩展 → X* 扩展。示例：`rv64imafdcv` → `rv64imafdc_zve64x_zvl128b`。

---

### Code Model / 代码模型

Determines how code and data are addressed in memory, affects instruction sequences for global variable access.

决定代码和数据在内存中的寻址方式，影响全局变量访问的指令序列。

**GCC Code Models / GCC 代码模型：**

| Model / 模型 | Address Range / 地址范围 | Description / 说明 |
|-------------|------------------------|-------------------|
| `medlow` | ±2 GiB | Default, uses `lui` + `addi` for addresses / 默认，使用 `lui` + `addi` 寻址 |
| `medany` | ±2 GiB | PC-relative addressing / PC 相对寻址 |
| `large` | Full 64-bit | Multi-instruction sequences / 多指令序列 |

**Example / 示例：**
```bash
# Compile with medany code model
riscv64-unknown-elf-gcc -mcmodel=medany -o app app.c
```

---

### Link-Time Optimization (LTO) / 链接时优化

Enables whole-program optimization by deferring optimization until link time, allowing cross-module inlining and dead code elimination.

通过将优化推迟到链接时实现全程序优化，允许跨模块内联和死代码消除。

**Usage / 使用：**
```bash
# Compile with LTO
riscv64-linux-gnu-gcc -flto -O3 -o app file1.c file2.c

# Separate compilation
riscv64-linux-gnu-gcc -flto -c file1.c -o file1.o
riscv64-linux-gnu-gcc -flto -c file2.c -o file2.o
riscv64-linux-gnu-gcc -flto -o app file1.o file2.o
```

**Benefits / 优势：**
- Smaller binary size / 更小的二进制大小
- Better performance / 更好的性能
- More aggressive inlining / 更激进的内联

---

### Function Multi-Versioning / 函数多版本

Technique for generating multiple versions of a function optimized for different ISA extensions, with runtime selection.

为不同 ISA 扩展生成函数的多个优化版本，运行时选择的技术。

**GCC Attribute Example / GCC 属性示例：**
```c
__attribute__((target("arch=rv64gc")))
void process_data_scalar(float *data, int n) { /* ... */ }

__attribute__((target("arch=rv64gcv")))
void process_data_vector(float *data, int n) { /* ... */ }

// Runtime dispatcher
void process_data(float *data, int n) {
    if (has_vector_extension())
        process_data_vector(data, n);
    else
        process_data_scalar(data, n);
}
```

---

### RISC-V Vector Intrinsics / RISC-V 向量内联函数

Compiler built-in functions providing direct access to RISC-V Vector (RVV) instructions without assembly.

编译器内置函数，提供对 RISC-V 向量（RVV）指令的直接访问，无需汇编。

**Example / 示例：**
```c
#include <riscv_vector.h>

void vadd(const float *a, const float *b, float *c, size_t n) {
    size_t vl;
    for (size_t i = 0; i < n; i += vl) {
        vl = __riscv_vsetvl_e32m1(n - i);
        vfloat32m1_t va = __riscv_vle32_v_f32m1(a + i, vl);
        vfloat32m1_t vb = __riscv_vle32_v_f32m1(b + i, vl);
        vfloat32m1_t vc = __riscv_vfadd_vv_f32m1(va, vb, vl);
        __riscv_vse32_v_f32m1(c + i, vc, vl);
    }
}
```

**Intrinsic Categories / 内联函数类别：**
- Vector load/store / 向量加载/存储
- Vector arithmetic / 向量算术
- Vector mask operations / 向量掩码操作
- Vector reduction / 向量归约
- Vector permutation / 向量排列

---

## Build Systems and Packaging / 构建系统和打包

### Buildroot / Buildroot

Embedded Linux build system that simplifies creating custom Linux distributions for RISC-V targets.

嵌入式 Linux 构建系统，简化为 RISC-V 目标创建自定义 Linux 发行版。

**Features / 特性：**
- Cross-compilation toolchain generation / 交叉编译工具链生成
- Root filesystem creation / 根文件系统创建
- Kernel configuration / 内核配置
- Package management / 软件包管理

**RISC-V Support / RISC-V 支持：**
- RV32/RV64 targets / RV32/RV64 目标
- Multiple C library options (glibc, musl, uclibc) / 多种 C 库选项（glibc、musl、uclibc）
- Bootloader integration / Bootloader 集成

---

### Yocto Project / Yocto 项目

Professional embedded Linux build system used for creating custom distributions, more flexible than Buildroot.

专业嵌入式 Linux 构建系统，用于创建自定义发行版，比 Buildroot 更灵活。

**RISC-V Layer / RISC-V 层：**
- `meta-riscv`: Official RISC-V layer / 官方 RISC-V 层
- Supports multiple boards and SoCs / 支持多个开发板和 SoC
- Integration with OpenEmbedded / 与 OpenEmbedded 集成

---

### OpenWrt / OpenWrt

Linux-based router and embedded device operating system, now supports RISC-V targets.

基于 Linux 的路由器和嵌入式设备操作系统，现在支持 RISC-V 目标。

**RISC-V Targets / RISC-V 目标：**
- SiFive HiFive Unleashed / SiFive HiFive Unleashed
- SiFive HiFive Unmatched / SiFive HiFive Unmatched
- QEMU virt machine / QEMU virt 机器

---

### Debian RISC-V Port / Debian RISC-V 移植

Official Debian port for RISC-V (rv64gc), provides thousands of pre-compiled packages.

Debian 的官方 RISC-V 移植（rv64gc），提供数千个预编译软件包。

**Status / 状态：**
- Official release: Debian 11 (Bullseye) onward / 官方发行版：Debian 11（Bullseye）起
- Package coverage: >90% of x86_64 packages / 软件包覆盖率：x86_64 软件包的 >90%
- Multiple desktop environments supported / 支持多个桌面环境

**Installation / 安装：**
```bash
# QEMU installation
qemu-system-riscv64 -M virt -m 2G -kernel u-boot.bin \
  -drive file=debian-riscv64.img,format=raw,if=virtio \
  -net nic,model=virtio -net user
```

---

### Fedora RISC-V / Fedora RISC-V

Fedora Linux port for RISC-V, closely tracks upstream and provides cutting-edge packages.

Fedora Linux 的 RISC-V 移植，紧密跟踪上游并提供前沿软件包。

**Features / 特性：**
- Latest kernel and toolchain / 最新内核和工具链
- GNOME desktop environment / GNOME 桌面环境
- Developer tools / 开发工具

---

### Arch Linux RISC-V / Arch Linux RISC-V

Community-driven Arch Linux port for RISC-V, rolling release model.

社区驱动的 Arch Linux RISC-V 移植，滚动发布模型。

**Repository / 仓库：**
- https://github.com/felixonmars/archriscv-packages
- Package count: ~13,000+ / 软件包数量：~13,000+

---

### Alpine Linux RISC-V / Alpine Linux RISC-V

Lightweight Linux distribution using musl libc and BusyBox, supports RISC-V.

使用 musl libc 和 BusyBox 的轻量级 Linux 发行版，支持 RISC-V。

**Characteristics / 特点：**
- Small footprint / 小占用空间
- Fast boot time / 快速启动
- Container-friendly / 容器友好

---

## Programming Language Support / 编程语言支持

### Rust RISC-V / Rust RISC-V

Rust has tier 2 support for RISC-V targets (`riscv64gc-unknown-linux-gnu`, `riscv32imac-unknown-none-elf`).

Rust 对 RISC-V 目标有 tier 2 支持（`riscv64gc-unknown-linux-gnu`、`riscv32imac-unknown-none-elf`）。

**Cross-compilation / 交叉编译：**
```bash
# Install RISC-V target
rustup target add riscv64gc-unknown-linux-gnu

# Compile
cargo build --target riscv64gc-unknown-linux-gnu
```

**Embedded Rust / 嵌入式 Rust：**
- `riscv` crate: Low-level register access / 低级寄存器访问
- `riscv-rt`: Runtime and startup code / 运行时和启动代码
- PAC (Peripheral Access Crate) for specific SoCs / 特定 SoC 的 PAC（外设访问 Crate）

---

### Go RISC-V / Go RISC-V

Go added RISC-V support in version 1.14 for `linux/riscv64`.

Go 在 1.14 版本中为 `linux/riscv64` 添加了 RISC-V 支持。

**Cross-compilation / 交叉编译：**
```bash
GOOS=linux GOARCH=riscv64 go build -o app main.go
```

**Status / 状态：**
- Full standard library support / 完整标准库支持
- CGo support / CGo 支持
- Garbage collector optimized / 垃圾收集器优化

---

### Python RISC-V / Python RISC-V

CPython supports RISC-V through standard Linux distributions, available in Debian/Fedora RISC-V repos.

CPython 通过标准 Linux 发行版支持 RISC-V，在 Debian/Fedora RISC-V 仓库中可用。

**JIT Compilers / JIT 编译器：**
- PyPy: RISC-V backend in progress / RISC-V 后端开发中
- Numba: LLVM-based, should work / 基于 LLVM，应该可以工作

---

### Java RISC-V / Java RISC-V

OpenJDK has RISC-V port (RV64G) available from JDK 17+.

OpenJDK 有 RISC-V 移植（RV64G），从 JDK 17+ 可用。

**JIT Compilation / JIT 编译：**
- C1 (Client) compiler: Supported / 支持
- C2 (Server) compiler: Supported / 支持
- GraalVM: Experimental RISC-V support / 实验性 RISC-V 支持

**Installation / 安装：**
```bash
# Debian/Ubuntu RISC-V
apt install openjdk-17-jdk
```

---

### JavaScript/Node.js RISC-V / JavaScript/Node.js RISC-V

V8 JavaScript engine added RISC-V support, enabling Node.js on RISC-V.

V8 JavaScript 引擎添加了 RISC-V 支持，使 Node.js 在 RISC-V 上可用。

**Status / 状态：**
- Node.js 16+ supports RISC-V / Node.js 16+ 支持 RISC-V
- Full feature parity with other architectures / 与其他架构完全功能对等

---

## Performance Analysis Tools / 性能分析工具

### perf / perf

Linux performance analysis tool, supports RISC-V hardware performance counters.

Linux 性能分析工具，支持 RISC-V 硬件性能计数器。

**Common Commands / 常用命令：**
```bash
# Profile application
perf record ./app
perf report

# CPU profiling
perf stat ./app

# Real-time monitoring
perf top
```

**RISC-V Events / RISC-V 事件：**
- `cycles`: CPU cycles / CPU 周期
- `instructions`: Retired instructions / 已执行指令
- `cache-misses`: Cache misses / 缓存未命中
- `branch-misses`: Branch mispredictions / 分支预测失败

---

### gprof / gprof

GNU profiler for function-level performance analysis.

GNU 分析器，用于函数级性能分析。

**Usage / 使用：**
```bash
# Compile with profiling
riscv64-linux-gnu-gcc -pg -o app app.c

# Run and generate profile
./app
gprof app gmon.out > analysis.txt
```

---

### Valgrind / Valgrind

Memory debugging and profiling tool, RISC-V support added recently.

内存调试和分析工具，最近添加了 RISC-V 支持。

**Tools / 工具：**
- **Memcheck**: Memory leak detection / 内存泄漏检测
- **Cachegrind**: Cache profiling / 缓存分析
- **Callgrind**: Call graph profiling / 调用图分析

**Usage / 使用：**
```bash
valgrind --tool=memcheck ./app
valgrind --tool=cachegrind ./app
```

---

## Development Boards / 开发板

### HiFive Unmatched / HiFive Unmatched

SiFive's RISC-V PC-class development board featuring SiFive U740 (4×U74 + 1×S7) at 1.4 GHz, 16 GB DDR4, PCIe x16 slot, M.2 NVMe, Gigabit Ethernet, USB 3.2.

SiFive 的 RISC-V PC 级开发板，采用 SiFive U740（4×U74 + 1×S7），1.4 GHz，16 GB DDR4，PCIe x16 插槽，M.2 NVMe，千兆以太网，USB 3.2。

---

### VisionFive 2 / VisionFive 2

StarFive's affordable RISC-V single-board computer with JH7110 SoC (4×U74 @ 1.5 GHz), up to 8 GB RAM, HDMI 2.0, dual Gigabit Ethernet, M.2 slot.

StarFive 的实惠 RISC-V 单板计算机，采用 JH7110 SoC（4×U74 @ 1.5 GHz），最多 8 GB RAM，HDMI 2.0，双千兆以太网，M.2 插槽。

---

### Sipeed LicheeRV / Sipeed LicheeRV

Compact RISC-V Linux-capable board with Allwinner D1 SoC (1×C906 @ 1 GHz), 512 MB/1 GB RAM, good for embedded applications.

紧凑的 RISC-V Linux 兼容开发板，采用全志 D1 SoC（1×C906 @ 1 GHz），512 MB/1 GB RAM，适合嵌入式应用。

---

### BeagleV-Ahead / BeagleV-Ahead

BeagleBoard.org's RISC-V board using TH1520 SoC (4×C910 @ 2.0 GHz), 4/8/16 GB RAM, AI acceleration, aimed at edge computing.

BeagleBoard.org 的 RISC-V 开发板，使用 TH1520 SoC（4×C910 @ 2.0 GHz），4/8/16 GB RAM，AI 加速，面向边缘计算。

---

### ESP32-C3 / ESP32-C3

Espressif's Wi-Fi + Bluetooth LE RISC-V microcontroller (1×RV32IMC @ 160 MHz), 400 KB SRAM, popular for IoT projects.

乐鑫的 Wi-Fi + 蓝牙 LE RISC-V 微控制器（1×RV32IMC @ 160 MHz），400 KB SRAM，在物联网项目中流行。

---

**Last Updated / 最后更新：** 2026-07-23
