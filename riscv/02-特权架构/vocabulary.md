---
tags: [riscv, privileged, vocabulary]
domain: privileged
last_updated: 2026-07-23
source: https://github.com/riscv/riscv-isa-manual
---

# RISC-V Privileged Architecture Vocabulary / RISC-V 特权架构词汇表

> Coverage: Privilege modes, CSR, Trap/Interrupt/Exception, Virtual memory, Physical memory protection
> Source: The RISC-V ISA Specification Volume II: Privileged Architecture

> 涵盖：特权模式、CSR、Trap/中断/异常、虚拟内存、物理内存保护
> 来源：The RISC-V ISA Specification Volume II: Privileged Architecture

---

## Privilege Modes / 特权模式

### privilege mode / 特权模式

RISC-V defines multiple privilege levels, and the processor runs in one mode at any given time, determining which instructions and CSRs are accessible.

RISC-V 定义了多个特权层级，处理器在任意时刻运行在某一模式下，决定哪些指令和 CSR 可访问。

| Mode / 模式 | Encoding / 编码 | Full Name / 全称 | Typical Usage / 典型运行内容 |
|------|------|------|------------|
| **U** | 00 | User / Unprivileged | User applications / 用户应用程序 |
| **S** | 01 | Supervisor | Operating system kernel / 操作系统内核 |
| **M** | 11 | Machine | Firmware/Bootloader (OpenSBI) / 固件/Bootloader（OpenSBI）|
| **H** | — | Hypervisor (extension) / Hypervisor（扩展） | Hypervisor / 虚拟机监控程序 |

All RISC-V implementations must support M-mode; S and U modes are optional (embedded systems often only have M+U).

所有 RISC-V 实现必须支持 M 模式；S 和 U 模式可选（嵌入式系统常仅有 M+U）。

### Machine mode (M-mode) / 机器模式

The highest privilege level with access to all CSRs and hardware resources. Firmware (such as OpenSBI) runs in M-mode, responsible for initializing hardware and transferring control to the S-mode kernel.

最高特权级，可访问所有 CSR 和硬件资源。固件（如 OpenSBI）运行在 M 模式，负责初始化硬件并将控制权移交给 S 模式内核。

### Supervisor mode (S-mode) / 监管者模式

The privilege level where operating system kernels run, capable of managing virtual memory (page tables) and handling system calls (ecall) from U-mode. Cannot directly access M-mode CSRs.

操作系统内核运行的特权级，可管理虚拟内存（页表）、处理来自 U 模式的系统调用（ecall）。不能直接访问 M 模式 CSR。

### User mode (U-mode) / 用户模式

The lowest privilege level where user applications run. Cannot execute privileged instructions (such as `mret`/`sret`) or access privileged CSRs; violations trigger illegal instruction exceptions.

最低特权级，用户应用程序运行的环境。无法执行特权指令（如 `mret`/`sret`）或访问特权 CSR，违规触发非法指令异常。

### Hypervisor extension (H) / 虚拟化扩展

Adds VS (Virtual Supervisor) and VU (Virtual User) modes on top of M/S/U, supporting multiple Guest OSes. Introduces two-stage address translation (Stage-1 managed by VS-mode, Stage-2 managed by HS-mode).

在 M/S/U 基础上增加 VS（Virtual Supervisor）和 VU（Virtual User）模式，支持运行多个 Guest OS。引入了两级地址翻译（Stage-1 由 VS-mode 管理，Stage-2 由 HS-mode 管理）。

---

## CSR (Control and Status Registers) / 控制与状态寄存器

### CSR (Control and Status Register) / 控制与状态寄存器

Special registers accessed through dedicated instructions (`csrrw`/`csrrs`/`csrrc`) that control processor behavior (e.g., interrupt enable) or reflect processor state (e.g., exception cause). CSR addresses are 12 bits, with the highest 2 bits encoding the access privilege level.

通过 `csrrw`/`csrrs`/`csrrc` 等专用指令访问的特殊寄存器，控制处理器行为（如中断使能）或反映处理器状态（如异常原因）。CSR 地址 12 位，最高 2 位编码访问特权级。

### CSR Instructions / CSR 指令

| Instruction / 指令 | Operation / 操作 | Description / 说明 |
|------|------|------|
| `CSRRW rd, csr, rs1` | rd←csr, csr←rs1 | Read-write swap / 读写交换 |
| `CSRRS rd, csr, rs1` | rd←csr, csr\|=rs1 | Read and set bits / 读后置位 |
| `CSRRC rd, csr, rs1` | rd←csr, csr&=~rs1 | Read and clear bits / 读后清位 |
| `CSRRWI/CSRRSI/CSRRCI` | Same as above, rs1 replaced by 5-bit immediate / 同上，rs1 换为 5位立即数 | Immediate versions / 立即数版本 |

### mstatus / mstatus

M-mode status register containing critical fields such as global interrupt enable (MIE), previous mode (MPP), and previous interrupt enable (MPIE). Hardware automatically updates on trap entry and restores on `mret`.

M 模式状态寄存器，包含全局中断使能（MIE）、之前模式（MPP）、之前中断使能（MPIE）等关键字段。进入 trap 时硬件自动更新，`mret` 时恢复。

### sstatus / sstatus

S-mode visible subset view of `mstatus`, containing fields such as SIE (S-mode interrupt enable) and SPP (previous privilege mode).

S 模式可见的 `mstatus` 子集视图，包含 SIE（S 模式中断使能）、SPP（之前特权模式）等字段。

### mtvec / mtvec

M-mode Trap Vector base address register storing the entry address of trap handlers. Supports two modes:
- **Direct** (lower 2 bits = 00): All traps jump to a fixed address
- **Vectored** (lower 2 bits = 01): Interrupts jump to base + cause×4 address

M 模式 Trap Vector 基地址寄存器，存储 trap 处理函数的入口地址。支持两种模式：
- **Direct**（低2位=00）：所有 trap 跳转到固定地址
- **Vectored**（低2位=01）：中断跳转到 base + cause×4 地址

### mepc / mepc

M-mode Exception Program Counter, saving the PC value when a trap is triggered. The `mret` instruction resumes execution from mepc.

M 模式异常程序计数器（Exception Program Counter），保存触发 trap 时的 PC 值。`mret` 指令从 mepc 恢复执行。

### mcause / mcause

M-mode trap cause register. The highest bit (Interrupt bit) is 1 for interrupts and 0 for exceptions; lower bits contain the specific cause code.

M 模式 trap 原因寄存器。最高位（Interrupt bit）为 1 表示中断，为 0 表示异常；低位为具体原因码。

| mcause (high bit=0, exception) / mcause（高位=0，异常） | Meaning / 含义 |
|----------------------|------|
| 0 | Instruction address misaligned |
| 1 | Instruction access fault |
| 2 | Illegal instruction |
| 3 | Breakpoint |
| 4 | Load address misaligned |
| 5 | Load access fault |
| 8 | Environment call from U-mode |
| 11 | Environment call from M-mode |
| 12 | Instruction page fault |
| 13 | Load page fault |
| 15 | Store page fault |

### mtval / mtval

M-mode trap value register providing auxiliary diagnostic information for traps (e.g., encoding of illegal instructions, virtual address of page faults).

M 模式 trap 附加信息寄存器，提供 trap 的辅助诊断信息（如非法指令的编码、缺页异常的虚拟地址）。

### mie / mip / 中断使能与挂起寄存器

- **mie** (M Interrupt Enable): Controls which interrupt sources are enabled
- **mip** (M Interrupt Pending): Indicates which interrupts are pending (awaiting handling)

- **mie**（M Interrupt Enable）：控制哪些中断源被使能
- **mip**（M Interrupt Pending）：指示哪些中断正在挂起（待处理）

### mhartid / mhartid

Machine Hart ID register, read-only, returns the unique ID of the current hart. Commonly used during multi-core boot to distinguish the primary core (hartid=0) from secondary cores.

Machine Hart ID 寄存器，只读，返回当前 hart 的唯一编号。常用于多核启动时区分主核（hartid=0）和从核。

### mscratch / sscratch / scratch 寄存器

M/S-mode dedicated temporary storage register, typically used at trap entry to swap with user stack pointer (save user sp, load kernel sp).

M/S 模式专用的临时存储寄存器，通常在 trap 入口处用于与用户栈指针交换（保存用户 sp，加载内核 sp）。

---

## Trap Mechanism / Trap（陷阱）机制

### trap / 陷阱

The collective term for exceptions and interrupts in RISC-V, handled uniformly through the same hardware mechanism (saving PC to xepc, jumping to xtvec).

RISC-V 中异常（exception）和中断（interrupt）的统称，统一通过相同的硬件机制处理（保存 PC 到 xepc，跳转到 xtvec）。

### exception / 异常

Synchronous event directly caused by the currently executing instruction (e.g., illegal instruction, page fault, divide-by-zero, `ecall`). Exceptions occur during specific instruction execution, synchronized with the instruction stream.

同步事件，由当前执行的指令直接引发（如非法指令、页错误、除零、`ecall`）。异常发生在特定指令执行时，与指令流同步。

### interrupt / 中断

Asynchronous event triggered by external hardware or timers, independent of the current instruction stream (e.g., UART reception, timer expiration). RISC-V defines three standard interrupts: software interrupt (MSI/SSI), timer interrupt (MTI/STI), and external interrupt (MEI/SEI).

异步事件，由外部硬件或定时器触发，与当前指令流无关（如 UART 接收、定时器超时）。RISC-V 定义了三种标准中断：软件中断（MSI/SSI）、定时器中断（MTI/STI）、外部中断（MEI/SEI）。

### ecall (Environment Call) / 环境调用

System call instruction for user mode to request services from higher privilege levels. `ecall` in U-mode triggers an "Environment call from U-mode" exception, jumping to S-mode kernel for handling (similar to `syscall` on x86).

用户态向更高特权级请求服务的系统调用指令。`ecall` 在 U 模式触发 Environment call from U-mode 异常，跳转到 S 模式内核处理（类似 `syscall` on x86）。

### ebreak / 调试断点

Triggers a Breakpoint exception (mcause=3), used by debuggers to set breakpoints.

触发 Breakpoint 异常（mcause=3），用于调试器设置断点。

### mret / sret / uret / 模式返回指令

Dedicated instructions for returning from trap handlers, restoring PC from corresponding xepc and restoring interrupt enable and privilege mode from xstatus:
- `mret`: Return from M-mode trap (restore to mode specified by MPP)
- `sret`: Return from S-mode trap
- `uret`: Return from U-mode trap (rarely used)

从 trap 处理程序返回的专用指令，从对应 xepc 恢复 PC，从 xstatus 恢复中断使能和特权模式：
- `mret`：从 M 模式 trap 返回（恢复 MPP 指定的模式）
- `sret`：从 S 模式 trap 返回
- `uret`：从 U 模式 trap 返回（较少使用）

### trap delegation / trap 委托

M-mode can delegate handling of certain exceptions/interrupts to S-mode, configured through `medeleg` (exception delegation) and `mideleg` (interrupt delegation) registers. After delegation, hardware directly jumps to S-mode trap handler when triggered, without entering M-mode first, reducing latency.

M 模式可将某些异常/中断的处理委托给 S 模式，通过 `medeleg`（异常委托）和 `mideleg`（中断委托）寄存器配置。委托后，触发条件时硬件直接跳转到 S 模式 trap 处理器，无需先进入 M 模式再转发，降低延迟。

### WFI (Wait For Interrupt) / 等待中断

Privileged instruction that puts the hart into a low-power waiting state until a pending interrupt awakens it. Not a strict sleep instruction—implementations may choose to implement it as a nop.

特权指令，使 hart 进入低功耗等待状态，直到有待处理中断唤醒。不是严格的休眠指令——实现可以选择 nop 形式实现。

---

## Virtual Memory / 虚拟内存

### virtual memory / 虚拟内存

Address translation mechanism enabled by S-mode through the `satp` CSR, translating virtual addresses in S-mode and U-mode to physical addresses, enabling process isolation and memory protection.

S 模式通过 `satp` CSR 启用的地址翻译机制，将 S 模式和 U 模式的虚拟地址翻译为物理地址，实现进程隔离和内存保护。

### satp (Supervisor Address Translation and Protection) / 监管者地址翻译与保护寄存器

CSR controlling virtual memory, containing:
- **MODE**: Address translation mode (0=disabled, 8=Sv39, 9=Sv48, 10=Sv57)
- **ASID**: Address Space Identifier (reduces TLB flushes)
- **PPN**: Physical Page Number of the root page table

控制虚拟内存的 CSR，包含：
- **MODE**：地址翻译模式（0=关闭，8=Sv39，9=Sv48，10=Sv57）
- **ASID**：地址空间标识符（减少 TLB 刷新）
- **PPN**：根页表的物理页号

### Sv39 / Sv39

The most commonly used 39-bit virtual address translation scheme for RV64, using 3-level page tables with 9-bit indexing per level and 4 KB page size (supports 2 MB large pages and 1 GB huge pages). Valid virtual address ranges: `0x0000_0000_0000_0000` ~ `0x0000_003F_FFFF_FFFF` (user space) and high-address kernel space.

RV64 最常用的 39 位虚拟地址翻译方案，使用 3 级页表，每级页表索引 9 位，页大小 4 KB（支持 2 MB 大页和 1 GB 超大页）。虚拟地址有效范围：`0x0000_0000_0000_0000` ~ `0x0000_003F_FFFF_FFFF`（用户空间）和高地址内核空间。

### Sv48 / Sv48

48-bit virtual address scheme with 4-level page tables, maximum virtual address space of 256 TB. Used by default in Linux kernel (e.g., kernel 6.x versions).

48 位虚拟地址方案，4 级页表，最大虚拟地址空间 256 TB。Linux 内核默认使用 Sv48（如内核 6.x 版本）。

### Sv57 / Sv57

57-bit virtual address scheme with 5-level page tables, maximum virtual address space of 128 PB, for extremely large memory systems.

57 位虚拟地址方案，5 级页表，最大虚拟地址空间 128 PB，用于超大内存系统。

### page table / 页表

Data structure mapping virtual page numbers to physical page numbers. In RISC-V, each page table level occupies one 4 KB physical page containing 512 8-byte page table entries (PTE).

将虚拟页号映射到物理页号的数据结构。RISC-V 每级页表占一个 4 KB 物理页，含 512 个 8 字节页表项（PTE）。

### PTE (Page Table Entry) / 页表项

An entry in a page table recording the mapping from virtual page to physical page and access permissions:

页表中的一个条目，记录虚拟页到物理页的映射及访问权限：

| Field / 位域 | Meaning / 含义 |
|------|------|
| V (Valid) | Whether this PTE is valid / 此 PTE 是否有效 |
| R/W/X | Readable/Writable/Executable / 可读/可写/可执行 |
| U (User) | Whether U-mode can access / U 模式是否可访问 |
| G (Global) | Global mapping (shared across all ASIDs) / 全局映射（所有 ASID 共享） |
| A (Accessed) | Set by hardware (page accessed) / 硬件置位（页被访问过） |
| D (Dirty) | Set by hardware (page written) / 硬件置位（页被写过） |
| PPN | Page Frame Number of next-level page table or physical page / 下级页表或物理页的页帧号 |

### TLB (Translation Lookaside Buffer) / 快表

Hardware structure caching recent address translation results, avoiding page table traversal on every memory access. The `sfence.vma` instruction is used to actively flush the TLB (called after modifying page tables).

缓存最近地址翻译结果的硬件结构，避免每次访存都遍历页表。`sfence.vma` 指令用于主动刷新 TLB（在修改页表后调用）。

### SFENCE.VMA / SFENCE.VMA

Memory management barrier instruction that flushes TLB entries for specified ASID and virtual address ranges. Must be executed after modifying page table mappings, otherwise the TLB may retain stale entries causing errors.

内存管理屏障指令，刷新指定 ASID 和虚拟地址范围的 TLB 条目。修改页表映射后必须执行，否则 TLB 中可能残留旧条目导致错误。

### ASID (Address Space Identifier) / 地址空间标识符

Optional field in satp marking which address space TLB entries belong to. During process switching, changing ASID can avoid full TLB flushes, improving context switch performance.

satp 中的可选字段，标记 TLB 条目归属的地址空间。进程切换时可通过更换 ASID 避免全量 TLB 刷新，提升上下文切换性能。

---

## Physical Memory Protection (PMP) / 物理内存保护（PMP）

### PMP (Physical Memory Protection) / 物理内存保护

Hardware access control mechanism configured by M-mode, defining physical address ranges and read/write/execute permissions for M/S/U modes. Used in embedded systems to implement memory isolation without virtual memory.

M 模式配置的硬件访问控制机制，定义物理地址范围及 M/S/U 模式的读写执行权限。嵌入式系统中用于实现内存隔离（无需虚拟内存）。

### pmpaddr / pmpcfg / PMP 配置寄存器

- **pmpaddr0–pmpaddr63**: Define address boundaries of protected regions
- **pmpcfg0–pmpcfg15**: Permission configuration for each region (R/W/X + address matching mode)

- **pmpaddr0–pmpaddr63**：定义保护区域的地址边界
- **pmpcfg0–pmpcfg15**：每个区域的权限配置（R/W/X + 地址匹配模式）

### PMP address matching modes / PMP 地址匹配模式

- **TOR** (Top Of Range): Region from previous pmpaddr to current pmpaddr
- **NA4**: Naturally aligned 4-byte region
- **NAPOT**: Naturally aligned power-of-2 sized region (most commonly used)

- **TOR**（Top Of Range）：区域从上一个 pmpaddr 到当前 pmpaddr
- **NA4**：自然对齐的 4 字节区域
- **NAPOT**：自然对齐的 2 的幂次方大小区域（最常用）

---

## S-mode CSRs / S 模式 CSR

### stvec / stvec

S-mode Trap Vector base address register, with the same structure as `mtvec` (Direct / Vectored modes). When a trap is delegated to S-mode, hardware jumps to `stvec`.

S 模式 Trap Vector 基地址寄存器，与 `mtvec` 结构相同（Direct / Vectored 模式）。当 trap 被委托到 S 模式时，硬件跳转到 `stvec`。

### sepc / sepc

S-mode Exception Program Counter, saving the PC value when an S-mode trap is triggered. `sret` resumes execution from `sepc`.

S 模式异常程序计数器，保存触发 S 模式 trap 时的 PC 值。`sret` 从 `sepc` 恢复执行。

### scause / scause

S-mode trap cause register, with the same format as `mcause` (highest bit = interrupt flag). Common values correspond to `mcause`, but the domain only includes causes delegated to S-mode handling.

S 模式 trap 原因寄存器，格式与 `mcause` 相同（最高位=中断标志）。常见值与 `mcause` 对应，但域仅包含被委托给 S 模式处理的 cause。

### stval / stval

S-mode trap value (corresponding to M-mode's `mtval`), storing the virtual address triggering the exception during page faults, or the instruction encoding during illegal instruction exceptions.

S 模式 trap 附加信息（对应 M 模式的 `mtval`），缺页异常时存储触发异常的虚拟地址，非法指令异常时存储指令编码。

### sie / sip / S 模式中断使能与挂起

- **sie** (S Interrupt Enable): Controls which interrupts are enabled in S-mode (SSIE/STIE/SEIE)
- **sip** (S Interrupt Pending): Indicates which interrupts are pending, set by hardware
- Both are S-mode visible subsets of `mie`/`mip`

- **sie**（S Interrupt Enable）：控制哪些中断在 S 模式被使能（SSIE/STIE/SEIE）
- **sip**（S Interrupt Pending）：指示哪些中断挂起，由硬件置位
- 均为 `mie`/`mip` 的 S 模式可见子集

---

## Machine Identification CSRs / 机器标识 CSR

### mvendorid / mvendorid

Vendor ID register (read-only), returning JEDEC vendor encoding (0 indicates non-commercial implementation, such as open-source cores).

供应商 ID 寄存器（只读），返回 JEDEC 供应商编码（0 表示非商业实现，如开源核心）。

### marchid / marchid

Architecture ID (read-only), identifying processor microarchitecture (assigned by RISC-V International, see [marchid.md](https://github.com/riscv/riscv-isa-manual/blob/main/marchid.md)).

架构 ID（只读），标识处理器微架构（由 RISC-V International 分配，见 [marchid.md](https://github.com/riscv/riscv-isa-manual/blob/main/marchid.md)）。

### mimpid / mimpid

Implementation ID (read-only), vendor-defined implementation version identifier (e.g., RTL version number).

实现 ID（只读），供应商自定义的实现版本标识（如 RTL 版本号）。

### misa / misa

Machine ISA register indicating which ISA extensions the current hart supports (each letter corresponds to bits[0–25]). The `MXL` field indicates base ISA width (1=32, 2=64, 3=128). Some implementations allow runtime dynamic modification (e.g., disabling floating-point extension), but most implementations are read-only.

机器 ISA 寄存器，指示当前 hart 支持的 ISA 扩展（每个字母对应 bit[0–25]）。`MXL` 字段指示基础 ISA 宽度（1=32, 2=64, 3=128）。某些实现允许运行时动态修改（如关闭浮点扩展），但大多数实现为只读。

### mconfigptr / mconfigptr

Pointer to platform configuration data structure (such as device tree or ACPI tables) physical address, allowing firmware to pass system configuration information to software (introduced in newer specifications, optional implementation).

指向平台配置数据结构（如设备树或 ACPI 表）的物理地址指针，允许固件向软件传递系统配置信息（较新规范引入，可选实现）。

---

## Counter Control / 计数器控制

### mcounteren / scounteren / 计数器访问控制寄存器

Control whether lower privilege levels can directly access performance counter CSRs (cycle/time/instret/hpmcounter3–31):
- **mcounteren**: M-mode controls S/U-mode counter access rights
- **scounteren**: S-mode controls U-mode counter access rights
- When the corresponding bit is 0, lower privilege level access to that counter triggers an illegal instruction exception (emulated by M/S-mode)

控制低特权级是否可直接访问性能计数器 CSR（cycle/time/instret/hpmcounter3–31）：
- **mcounteren**：M 模式控制 S/U 模式的计数器访问权
- **scounteren**：S 模式控制 U 模式的计数器访问权
- 对应位为 0 时，低特权级访问该计数器触发非法指令异常（由 M/S 模式模拟）

### mcycle / minstret / 机器级计数器

- **mcycle**: M-mode writable 64-bit clock cycle counter (the `cycle` CSR is its read-only shadow)
- **minstret**: M-mode writable 64-bit retired instruction counter (the `instret` CSR is its read-only shadow)
- M-mode software can zero these counters for precise performance measurement

- **mcycle**：M 模式可写的 64 位时钟周期计数器（`cycle` CSR 是其只读影射）
- **minstret**：M 模式可写的 64 位已退休指令计数器（`instret` CSR 是其只读影射）
- M 模式软件可清零这些计数器用于精确性能测量

### mcountinhibit / 计数器抑制寄存器

Controls which counters (cycle/instret/hpmcounter3–31) stop counting, used for precise performance measurement (pause counters to exclude the impact of measurement code itself).

控制哪些计数器（cycle/instret/hpmcounter3–31）停止计数，用于精确性能测量（暂停计数器以排除测量代码本身的影响）。

### Sscofpmf (Counter Overflow and Privilege Mode Filtering) / 计数器溢出与特权模式过滤

Ratified extension (2022) that adds two capabilities to performance counters:

已批准扩展（2022），为性能计数器添加两项能力：

**1. Counter Overflow Interrupts / 计数器溢出中断：**
- Each `hpmcounter3–31` can trigger a local counter overflow interrupt when it overflows
- **scountovf** CSR (0xDA0): Readable by S-mode; each bit indicates if the corresponding counter has overflowed
- `mip.LCOFIP` (bit 13): Local Counter Overflow Interrupt Pending
- `mie.LCOFIE` (bit 13): Local Counter Overflow Interrupt Enable
- Used for sampling-based profiling (set counter to -N, trap when N events occur)

每个 `hpmcounter3–31` 溢出时可触发本地计数器溢出中断：
- **scountovf** CSR（0xDA0）：S 模式可读，每位指示对应计数器是否溢出
- `mip.LCOFIP`（bit 13）：本地计数器溢出中断挂起
- `mie.LCOFIE`（bit 13）：本地计数器溢出中断使能
- 用于基于采样的性能分析（将计数器设为 -N，N 个事件后触发陷阱）

**2. Privilege Mode Filtering / 特权模式过滤：**
- Each HPM event selector (`mhpmeventX`) gains filter bits in the upper bits:
  - **MINH** (bit 62): Inhibit counting in M-mode / 在 M 模式下禁止计数
  - **SINH** (bit 61): Inhibit counting in S/HS-mode / 在 S/HS 模式下禁止计数
  - **UINH** (bit 60): Inhibit counting in U-mode / 在 U 模式下禁止计数
  - **VSINH** (bit 59): Inhibit counting in VS-mode / 在 VS 模式下禁止计数
  - **VUINH** (bit 58): Inhibit counting in VU-mode / 在 VU 模式下禁止计数
- This allows `perf stat --per-thread` and kernel/user-space separated measurements

每个 HPM 事件选择器（`mhpmeventX`）高位新增过滤位：
- 可按特权模式精确过滤计数，支持 `perf stat --per-thread` 及内核/用户态分离测量

**Usage Example / 使用示例：**
```c
// Count cache misses only in U-mode (exclude M/S)
// 仅统计 U 模式下的缓存未命中（排除 M/S）
uint64_t event = CACHE_MISS_EVENT | MINH | SINH;
csr_write(mhpmevent3, event);
csr_write(mhpmcounter3, -1000ULL);  // Overflow after 1000 events
csr_set(mie, MIE_LCOFIE);           // Enable overflow interrupt
```

Control the feature set visible to lower privilege modes (introduced in ratified 2021+ specifications):

**menvcfg** (Machine Environment Configuration, RV64: 0x30A, RV32: 0x30A + menvcfgh 0x31A):
- **FIOM** (bit 0): Fence of I/O implies Memory - Whether I/O fence also acts as memory fence
- **CBIE** (bits 5:4): Cache Block Invalidate/Flush Enable - Controls S/U-mode cache block operations (Zicbom): 00=illegal, 01=flush, 11=invalidate
- **CBZE** (bit 6): Cache Block Zero Enable - Allow S/U-mode to execute cache block zero (Zicboz)
- **CBCFE** (bit 7): Cache Block Clean and Flush Enable - Allow S/U-mode cache clean/flush (Zicbom)
- **ADUE** (bit 61/RV64): Enable hardware A/D bit updates for S-mode (Svadu)
- **PBMTE** (bit 62/RV64): Enable page-based memory types for S-mode (Svpbmt)
- **STCE** (bit 63/RV64): Enable S-mode access to stimecmp (Sstc)

**senvcfg** (Supervisor Environment Configuration, 0x10A):
- Similar fields controlling U-mode capabilities
- Filtered by menvcfg settings (can only grant subset of what M-mode allows)

**henvcfg** (Hypervisor Environment Configuration, RV64: 0x60A, RV32: 0x60A + henvcfgh 0x61A):
- Controls VS/VU-mode features in virtualization environments
- Similar field structure to menvcfg/senvcfg
- Used with H extension for fine-grained guest control

控制较低特权模式可见的特性集合（2021+规范批准引入）：

**menvcfg**（机器环境配置，RV64: 0x30A，RV32: 0x30A + menvcfgh 0x31A）：
- **FIOM**（bit 0）：I/O fence 是否同时作为内存 fence
- **CBIE**（bits 5:4）：允许 S/U 模式执行缓存块操作（Zicbom）：00=非法，01=刷新，11=失效
- **CBZE**（bit 6）：允许 S/U 模式执行缓存块清零（Zicboz）
- **CBCFE**（bit 7）：允许 S/U 模式缓存清理/刷新（Zicbom）
- **ADUE**（bit 61/RV64）：启用 S 模式硬件 A/D 位更新（Svadu）
- **PBMTE**（bit 62/RV64）：启用 S 模式基于页的内存类型（Svpbmt）
- **STCE**（bit 63/RV64）：启用 S 模式访问 stimecmp（Sstc）

**senvcfg**（监管者环境配置，0x10A）：
- 类似字段控制 U 模式能力
- 受 menvcfg 设置过滤（只能授予 M 模式允许的子集）

**henvcfg**（Hypervisor 环境配置，RV64: 0x60A，RV32: 0x60A + henvcfgh 0x61A）：
- 控制虚拟化环境中的 VS/VU 模式特性
- 与 menvcfg/senvcfg 类似的字段结构
- 与 H 扩展配合用于细粒度客户机控制

---

## Interrupt Architecture / 中断架构

### local interrupt / 本地中断

Interrupt sources private to each hart, defined in fixed bit fields of `mip`/`mie`:
- **MSIP / SSIP** (Software Interrupt Pending): Software interrupt (triggered by other harts or M-mode writes)
- **MTIP / STIP** (Timer Interrupt Pending): Timer interrupt (set by CLINT/ACLINT hardware)
- **MEIP / SEIP** (External Interrupt Pending): External interrupt (set by PLIC/AIA)

每个 hart 私有的中断源，定义在 `mip`/`mie` 的固定位域：
- **MSIP / SSIP**（Software Interrupt Pending）：软件中断（由其他 hart 或 M 模式写入触发）
- **MTIP / STIP**（Timer Interrupt Pending）：定时器中断（由 CLINT/ACLINT 硬件置位）
- **MEIP / SEIP**（External Interrupt Pending）：外部中断（由 PLIC/AIA 置位）

### CLINT (Core Local Interruptor) / 核本地中断器

Timer and software interrupt implementation designed by SiFive, widely adopted by early RISC-V platforms (QEMU virt, HiFive, etc.). Provides MMIO registers `mtime` (global clock) and `mtimecmp` (per-hart timer compare value):
- `mtime >= mtimecmp` → Triggers MTIP timer interrupt
- Writing `msip` MMIO register → Triggers MSIP software interrupt

Has been standardized and replaced by ACLINT specification, but CLINT implementations remain widespread.

SiFive 设计的定时器和软件中断实现，广泛被早期 RISC-V 平台采用（QEMU virt、HiFive 等）。提供 MMIO 寄存器 `mtime`（全局时钟）和 `mtimecmp`（per-hart 定时器比较值）：
- `mtime >= mtimecmp` → 触发 MTIP 定时器中断
- 写 `msip` MMIO 寄存器 → 触发 MSIP 软件中断

已被 ACLINT 规范标准化替代，但 CLINT 实现仍广泛存在。

### mtime / mtimecmp / 定时器寄存器

- **mtime**: Global monotonically increasing real-time timer (typically counts at reference clock frequency, not CPU frequency)
- **mtimecmp**: Per-hart timer compare value; when `mtime >= mtimecmp`, MTIP is set to 1

- **mtime**：全局单调递增的实时计时器（通常以参考时钟频率计数，非 CPU 频率）
- **mtimecmp**：per-hart 的定时器比较值，当 `mtime >= mtimecmp` 时 MTIP 置 1

### Sstc extension / Sstc 扩展

Allows S-mode to set timers directly (adds `stimecmp` CSR), without going through SBI (M-mode), reducing timer interrupt latency. Linux kernel 6.x already supports it.

允许 S 模式直接设置定时器（新增 `stimecmp` CSR），无需通过 SBI（M 模式）设置，降低定时器中断延迟，Linux 内核 6.x 已支持。

### interrupt priority / 中断优先级

RISC-V local interrupt priority (high to low): External interrupt (MEI/SEI) > Software interrupt (MSI/SSI) > Timer interrupt (MTI/STI). External interrupt priority is managed internally by PLIC/AIA.

RISC-V 本地中断优先级（高到低）：外部中断（MEI/SEI）> 软件中断（MSI/SSI）> 定时器中断（MTI/STI）。外部中断优先级由 PLIC/AIA 内部管理。

---

## Page Faults and Access Faults / 页错误与访问错误

### page fault / 页错误

Exception triggered when virtual memory translation fails or permission check does not pass:
- **Instruction page fault** (cause=12): Page fault during instruction fetch
- **Load page fault** (cause=13): Page fault during Load
- **Store/AMO page fault** (cause=15): Page fault during Store or AMO

When triggered, `stval` (or `mtval`) saves the virtual address causing the fault, for software handling (e.g., CoW, demand paging).

虚拟内存翻译失败或权限检查不通过时触发的异常：
- **Instruction page fault**（cause=12）：取指时页错误
- **Load page fault**（cause=13）：Load 时页错误
- **Store/AMO page fault**（cause=15）：Store 或 AMO 时页错误

触发时 `stval`（或 `mtval`）保存引发错误的虚拟地址，供软件处理（如 CoW、demand paging）。

---

## mstatus Key Fields (Detailed) / mstatus 关键字段（精读补充）

### MPRV (Modify PRiVilege) / 修改特权位

`mstatus.MPRV`: When set to 1, explicit memory accesses (Load/Store) use the privilege mode specified by `mstatus.MPP` for address translation and protection checks, rather than the current actual privilege mode. Instruction fetch is unaffected. Commonly used in M-mode trap handlers to perform memory accesses on behalf of U/S-mode (e.g., emulating misaligned accesses). Automatically cleared to zero when `mret`/`sret` switches to lower privilege.

`mstatus.MPRV`：当为 1 时，显式内存访问（Load/Store）使用 `mstatus.MPP` 指定的特权模式进行地址翻译和保护检查，而不是当前实际特权模式。指令取址不受影响。常用于 M 模式陷阱处理程序中代替 U/S 模式执行内存访问（如模拟非对齐访问）。`mret`/`sret` 切换到更低权限时自动清零。

### MXR (Make eXecutable Readable) / 使可执行页可读

`mstatus.MXR`: When set to 1, allows load operations from pages marked as executable (X=1) but not readable (R=0). Primary use: allows loading instruction words from execute-only pages (instruction emulation/debugging) without marking the page as readable. Only effective when virtual memory is enabled.

`mstatus.MXR`：当为 1 时，允许从标记为可执行（X=1）但不可读（R=0）的页面进行加载操作。主要用途：允许从只执行页面加载指令字（指令模拟/调试），无需将页面标记为可读。仅在启用虚拟内存时生效。

### SUM (permit Supervisor User Memory) / 允许监管者访问用户内存

`mstatus.SUM`: When 0 (default), S-mode access to pages with U bit set to 1 triggers a page fault. When 1, S-mode can access U-bit pages. Prevents kernel from accidentally accessing user memory (security mechanism). Unaffected: S-mode can never fetch instructions from U-mode pages (regardless of SUM).

`mstatus.SUM`：当为 0 时（默认），S 模式访问 U 位为 1 的页面会触发 page fault。当为 1 时，S 模式可访问 U 位页面。防止内核意外访问用户内存（安全机制）。不受影响：S 模式永远不能从 U 模式页面取指（无论 SUM 如何）。

### TVM (Trap Virtual Memory) / 陷阱虚拟内存

`mstatus.TVM`: When set to 1, executing `csrr/csrw satp` or `sfence.vma`/`sinval.vma` in S-mode triggers an illegal instruction exception. Hypervisor uses this mechanism to intercept Guest OS address translation operations, implementing lazy population of shadow page tables.

`mstatus.TVM`：当为 1 时，S 模式执行 `csrr/csrw satp` 或 `sfence.vma`/`sinval.vma` 时触发非法指令异常。Hypervisor 通过此机制拦截 Guest OS 的地址翻译操作，实现 shadow page table 的懒填充（lazy population）。

### TW (Timeout Wait) / 超时等待

`mstatus.TW`: When set to 1, executing `wfi` instruction at privilege level lower than M triggers an illegal instruction exception if it exceeds an implementation-specific time limit. Allows Hypervisor to intercept Guest OS WFI, triggering VM switch rather than actual waiting.

`mstatus.TW`：当为 1 时，在低于 M 特权级执行 `wfi` 指令若超过实现特定时限则触发非法指令异常。允许 Hypervisor 拦截 Guest OS 的 WFI，触发虚拟机切换而非真正等待。

### TSR (Trap SRet) / 陷阱 SRET

`mstatus.TSR`: When set to 1, executing `sret` in S-mode triggers an illegal instruction exception. Used when emulating virtualization functionality through M-mode on implementations without H extension support.

`mstatus.TSR`：当为 1 时，S 模式执行 `sret` 触发非法指令异常。用于在不支持 H 扩展的实现上通过 M 模式模拟虚拟化功能时使用。

### FS / VS / XS / SD / 上下文状态字段

Used to quickly determine whether saving/restoring floating-point (FS), vector (VS), or other extension (XS) state is needed, avoiding unnecessary context save overhead:
- **Off (0)**: Extension disabled, access triggers illegal instruction exception
- **Initial (1)**: State initialized but not yet modified
- **Clean (2)**: State exists but hasn't been modified since last save
- **Dirty (3)**: State modified, needs saving before context switch
- **SD** (Status Dirty): Summary bit; when any of FS/VS/XS is Dirty, SD=1

用于快速确定是否需要保存/恢复浮点（FS）、向量（VS）或其他扩展（XS）状态，避免不必要的上下文保存开销：
- **Off（0）**：扩展已禁用，访问触发非法指令异常
- **Initial（1）**：状态已初始化但尚未修改
- **Clean（2）**：状态存在但自上次保存后未被修改
- **Dirty（3）**：状态已修改，上下文切换前需保存
- **SD**（Status Dirty）：汇总位；当 FS/VS/XS 任一为 Dirty 时，SD=1
- **Initial（1）**：扩展已启用，状态为初始值（上下文切换时无需保存）
- **Clean（2）**：状态与上次保存时相同（无需保存）
- **Dirty（3）**：状态已修改（需要保存）
- **SD**（最高位只读）：FS/VS/XS 中任一为 Dirty 则为 1，快速判断是否需要保存扩展状态

---

## 重置与 NMI

### reset vector / 重置向量
hart 复位后 PC 的初始值，由平台规范定义（不在 ISA 规范中固定）。`mstatus.MIE` 和 `MPRV` 在复位时清零，`misa` 复位到最大支持扩展集，`pc` 设为实现定义的复位向量地址。

### NMI (Non-Maskable Interrupt) / 不可屏蔽中断
只用于硬件错误条件，无论中断使能状态如何立即跳转到实现定义的 NMI 向量（运行在 M 模式）。与普通复位不同，NMI **不重置**处理器状态，允许诊断和报告硬件错误。`mcause` 设置为 NMI 来源，`mcause` 高位（Interrupt bit）应为 1。

### Smrnmi (Resumable Non-Maskable Interrupts) / 可恢复不可屏蔽中断扩展

Ratified June 2024. Adds support for resumable NMIs (RNMI), which allow recovery from double traps and critical errors without halting the hart.

2024年6月批准。添加可恢复NMI（RNMI）支持，允许从双重陷阱和关键错误中恢复而不停止hart。

**Key Features / 关键特性：**

1. **RNMI Handler / RNMI处理器：**
   - Separate NMI handler that can recover from double traps
   - Uses dedicated CSRs: `mnepc` (NMI exception PC), `mncause` (NMI cause), `mnstatus` (NMI status)
   - Entry via `mntvec` (NMI trap vector base)
   - Return via `mnret` instruction
   
   独立的NMI处理器可从双重陷阱恢复
   - 使用专用CSR：`mnepc`（NMI异常PC）、`mncause`（NMI原因）、`mnstatus`（NMI状态）
   - 通过`mntvec`（NMI陷阱向量基址）进入
   - 通过`mnret`指令返回

2. **Double Trap Handling / 双重陷阱处理：**
   - When double trap occurs (`mstatus.MDT=1` and another M-mode trap), hart enters RNMI mode instead of critical-error state
   - Saves context to `mnepc`, `mncause`, `mnstatus`
   - Handler can log error and attempt recovery
   
   双重陷阱发生时（`mstatus.MDT=1`且再次M模式陷阱），hart进入RNMI模式而非关键错误状态
   - 保存上下文到`mnepc`、`mncause`、`mnstatus`
   - 处理器可记录错误并尝试恢复

3. **RNMI Priority / RNMI优先级：**
   - RNMI has higher priority than all other interrupts and exceptions
   - Cannot be masked or disabled
   - Nested RNMIs result in critical-error state
   
   RNMI优先级高于所有其他中断和异常
   - 无法屏蔽或禁用
   - 嵌套RNMI导致关键错误状态

**CSRs / 控制状态寄存器：**
- `mntvec` (0x7C0): RNMI trap vector base address
- `mnepc` (0x7C1): RNMI exception program counter
- `mncause` (0x7C2): RNMI cause register
- `mnstatus` (0x7C3): RNMI status register (subset of mstatus)

**Use Cases / 使用场景：**
- Fault-tolerant systems that must recover from software errors
- System management interrupt (SMI) handling in servers
- Hardware error reporting without halting the system
- Debugging double trap conditions

容错系统从软件错误恢复、服务器系统管理中断（SMI）处理、硬件错误报告而不停机、调试双重陷阱条件

### double trap / 双重陷阱
在 trap 处理的非可重入阶段（关键阶段，处理程序尚未保存足够状态以处理嵌套 trap）发生另一个 trap 的情况。由 `mstatus.MDT`（M 模式禁用陷阱位）控制检测：
- 进入 M 模式 trap 时 MDT 自动置 1
- 若 MDT 已为 1 时再次进入 M 模式 trap → 触发 unexpected trap（双重陷阱）
- 若支持 Smrnmi 扩展，转入 RNMI 处理；否则 hart 进入 critical-error 状态（停止执行，发出平台信号）
- `mret` 或 `sret` 自动清零 MDT，表示 trap 处理程序已恢复可重入状态

---

## PMA（物理内存属性）

### PMA (Physical Memory Attribute) / 物理内存属性
描述物理地址空间每个区域固有属性的规范，包括：内存类型（主存 vs I/O）、支持的访问宽度、原子操作支持级别（AMONone/AMOSwap/AMOLogical/AMOArithmetic）、保留集支持（RsrvNone/RsrvNonEventual/RsrvEventual）、缓存一致性、是否幂等（idempotency）。

**关键特性**：PMA 不随执行上下文变化（区别于 PMP），通常在芯片设计时或板级设计时固定，通过 PMA Checker 硬件检查，违规产生 access-fault 异常（而非 page-fault）。

### AMO PMA levels / 原子操作 PMA 级别
- **AMONone**：不支持任何 AMO
- **AMOSwap**：仅支持 `amoswap`
- **AMOLogical**：AMOSwap + `amoand/amoor/amoxor`
- **AMOArithmetic**：全部 A 扩展 AMO（C/C++11 标准映射要求 AMOArithmetic）

### RsrvEventual / RsrvNonEventual / 保留集 PMA
- **RsrvEventual**：支持 LR/SC，**保证**最终成功（C/C++11 映射要求）
- **RsrvNonEventual**：支持 LR/SC，但**不保证**成功，需要软件备用机制
- **RsrvNone**：不支持 LR/SC（不可保留区域）

### idempotency PMA / 幂等性 PMA
描述读写操作是否有副作用。主存区域通常是幂等的（可安全预取/推测读写），I/O 区域可能是非幂等的（读可能有副作用，如读空 FIFO）。非幂等区域禁止推测或冗余访问，非对齐访问应产生 access-fault（而非 misaligned-address fault，防止软件尝试拆分访问）。

---

## 地址翻译算法（Sv39 精读）

### page table walk / 页表遍历
地址翻译算法步骤（以 Sv39 为例，LEVELS=3，PTESIZE=8）：
1. `a ← satp.PPN × 4096`，`i ← 2`（从根页表开始）
2. 读 PTE：`pte ← memory[a + va.vpn[i] × 8]`，检查 PMA/PMP
3. 若 `pte.V=0` 或 `pte.R=0 且 pte.W=1`，或保留位非零 → page-fault
4. 若 `pte.R=1 或 pte.X=1` → 跳到步骤 5（叶节点）；否则 `i--`，若 `i<0` → page-fault，否则 `a ← pte.PPN × 4096` 返回步骤 2
5. 叶 PTE 找到。若 `i>0` 且 `pte.PPN[i-1:0] ≠ 0` → 超大页未对齐 → page-fault
6. 检查 U 位（U-mode 只能访问 U=1 的页，S-mode 受 SUM 控制）
7. 检查 R/W/X 权限
8. 若 `pte.A=0`（或 store 且 `pte.D=0`）：Svade 扩展 → page-fault；否则原子更新 A/D 位
9. 物理地址：`pa.pgoff = va.pgoff`；若 `i>0`（超大页）则 `pa.ppn[i-1:0] = va.vpn[i-1:0]`；`pa.ppn[LEVELS-1:i] = pte.PPN[LEVELS-1:i]`

### superpage / 超大页（大页）
任意级别的叶 PTE 均可作为叶节点，实现不同粒度的大页：
| 方案 | 级别 | 页大小 |
|------|------|--------|
| Sv39 | i=0 | 4 KiB（标准页） |
| Sv39 | i=1 | 2 MiB（Megapage）|
| Sv39 | i=2 | 1 GiB（Gigapage）|
| Sv48 | i=3 | 512 GiB（Terapage）|
| Sv57 | i=4 | 256 TiB（Petapage）|

超大页必须在虚拟和物理上都对齐到其大小边界，否则触发 page-fault。

### Svade extension / Svade 扩展
要求在 A 位为 0 的页面被访问时（或 D 位为 0 的页面被写入时）触发 page-fault，由软件负责更新 A/D 位。未实现 Svade 时，硬件可自动更新 A/D 位（需原子操作），A 位更新可以是推测式的，D 位更新必须精确（按程序序）。

### Svadu extension / Svadu 扩展
通过 `menvcfg.ADUE` 启用硬件自动更新 PTE A/D 位的扩展，使 Svade 的 page-fault 路径被绕过（performance 优化）。

### address-translation cache / 地址翻译缓存（TLB）
规范允许实现缓存页表遍历结果（TLB），但这些缓存对 hart 而言是私有的。对同一虚拟地址/ASID 可存在多条缓存条目（TLB 中可能存在重叠条目）。SFENCE.VMA 用于使相关缓存条目失效。

### SFENCE.VMA 精确语义（原文）
`sfence.vma rs1, rs2` 的四种组合：
- `rs1=x0, rs2=x0`：刷新所有 ASID 的所有翻译缓存（全局）
- `rs1=x0, rs2≠x0`：刷新 rs2 指定 ASID 的所有缓存（全局映射不受影响）
- `rs1≠x0, rs2=x0`：仅刷新与 rs1 虚拟地址对应的叶 PTE 缓存（所有 ASID）
- `rs1≠x0, rs2≠x0`：仅刷新 rs2 指定 ASID 中与 rs1 虚拟地址对应的叶 PTE（全局映射不受影响）

若 rs1 持有非法虚拟地址，SFENCE.VMA 无效果且不触发异常。实现可选择"过度刷新"（保守实现）。

### Svnapot extension / 自然对齐幂次大页扩展
使用 PTE 的 bit 63（N 位）标记一组连续页面组成逻辑上的自然对齐大页，大页粒度不限于 Sv39 的标准超大页级别。例如：64 KiB 的 NAPOT 大页（16 个 4 KiB 页），通过 N 位和 PTE 地址编码指定大小。

### Svpbmt extension / 页表内存类型扩展
使用 PTE 的 bit 62-61（PBMT 字段）为每个页面指定内存类型，覆盖默认 PMA 设置：
- **00（PMA）**：使用物理内存的默认 PMA
- **01（NC，Non-Cacheable）**：不可缓存的幂等内存（弱序，可预取）
- **10（IO）**：幂等性 I/O（弱序，禁止预取/推测）
- **11**：保留

通过 `menvcfg.PBMTE` 控制 S 模式和 G-stage 地址翻译是否启用此扩展。

---

## mseccfg（安全配置寄存器）

### mseccfg / mseccfg
机器安全配置 CSR，由需要它的扩展实现时存在：
- **SSEED/USEED**（Zkr 扩展）：控制 S/U 模式对 `seed` CSR（硬件熵源）的访问权
- **RLB/MMWP/MML**（Smepmp 扩展）：增强 PMP 的 M 模式内存隔离策略（PMP 复位前 sticky 位）
- **MLPE**（Zicfilp 扩展）：M 模式落地垫（landing pad）使能

### Smepmp / Smepmp
增强 PMP 的机器模式保护扩展，修改 `pmpcfg.L` 位语义：
- **MML=0**（默认）：L 位表示 PMP 规则锁定（M 模式也受限）
- **MML=1**：L 位区分 M-mode only 规则（L=1）和 S/U-mode only 规则（L=0），以及 Shared-Region（`RW=01` 编码）

### access fault / 访问错误
物理内存访问违反 PMP 规则，或访问不存在的物理地址时触发（cause=1/5/7），区别于页错误（页错误是虚拟内存翻译问题）。

### demand paging / 按需分页
操作系统延迟实际物理内存分配的技术：初始时页表中不映射（PTE V=0），首次访问触发 page fault，OS 在 fault handler 中分配物理页并填充 PTE，然后 sret 重新执行导致 fault 的指令。

### Copy-on-Write (CoW) / 写时复制
fork 系统调用的高效实现：父子进程共享物理页（PTE 标记只读），当一方写入时触发 Store page fault，OS 复制该页并更新 PTE，使两者各有独立副本。RISC-V 的 A（Accessed）和 D（Dirty）位辅助 OS 追踪页面状态。

---

## Recent Ratified Extensions (2024-2026) / 近期批准扩展

### Smctr / Ssctr (Control Transfer Records) / 控制转移记录扩展

Ratified November 2024. Enables hardware recording of control flow transfers (branches, jumps, exceptions, interrupts) for debugging and performance analysis.

2024年11月批准。启用硬件记录控制流转移（分支、跳转、异常、中断）用于调试和性能分析。

**Smctr** (Machine-mode CTR): M-mode control transfer recording
**Ssctr** (Supervisor-mode CTR): S-mode control transfer recording (requires Smctr)

**Key Features / 关键特性：**

1. **Control Transfer Filtering / 控制转移过滤：**
   - Record branches, jumps, calls, returns
   - Record exceptions and interrupts
   - Configurable filtering by transfer type
   - Per-privilege-mode enable/disable
   
2. **CTR Buffer / CTR缓冲区：**
   - Hardware FIFO storing (source PC, target PC, type) tuples
   - Overflow handling with interrupt generation
   - Memory-mapped or CSR-accessible interface

3. **Use Cases / 使用场景：**
   - Last-branch-record (LBR) for debugging
   - Coverage analysis for testing
   - Performance profiling of hot paths
   - Control-flow integrity (CFI) validation

### Smcdeleg / Ssccfg (Supervisor Counter Delegation) / 监管者计数器委托

Ratified March 2024. Allows S-mode to delegate performance counter access to U-mode on a per-counter basis.

2024年3月批准。允许S模式按计数器粒度将性能计数器访问委托给U模式。

**Key Features / 关键特性：**
- S-mode can control which counters U-mode can read
- S-mode can inhibit specific counters (scountinhibit)
- S-mode can handle counter overflow interrupts (scountovf)
- Enables user-space performance monitoring without kernel mediation

### Smcsrind / Sscsrind (Indirect CSR Access) / 间接CSR访问

Ratified February 2024. Provides indirect mechanism to access many CSRs through a small window.

2024年2月批准。通过小窗口间接访问大量CSR。

**CSRs:**
- miselect (0x350) / mireg (0x351) for M-mode
- siselect (0x150) / sireg (0x151) for S-mode
- Used by AIA for large interrupt file CSR sets

### Svvptc / Svrsw60t59b / Ssdbltrp / Smdbltrp

Additional privileged extensions ratified 2024-2025. See individual documentation for details.

