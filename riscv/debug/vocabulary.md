---
tags: [riscv, debug, vocabulary]
domain: debug
last_updated: 2026-07-22
source: https://github.com/riscv/riscv-debug-spec
---

# RISC-V Debug and Trace Vocabulary / RISC-V 调试与跟踪词汇表

> Coverage: JTAG, Debug Module, Trigger, Trace  
> Source: RISC-V External Debug Specification · RISC-V Trace Specification

> 涵盖：JTAG、Debug Module、Trigger、Trace  
> 来源：RISC-V External Debug Specification · RISC-V Trace Specification

---

## External Debug / 外部调试

### RISC-V External Debug Specification / RISC-V 外部调试规范

Defines the protocol and hardware modules for debuggers (such as GDB + OpenOCD) to debug RISC-V harts via JTAG interface, including specifications for Debug Module (DM), Debug Transport Module (DTM), and Trigger Module.

定义调试器（如 GDB + OpenOCD）通过 JTAG 接口调试 RISC-V hart 的协议和硬件模块，包含 Debug Module（DM）、Debug Transport Module（DTM）、Trigger Module 的规范。

### JTAG (Joint Test Action Group) / JTAG

A debug/test standard (IEEE 1149.1) that accesses target devices through a 4-wire serial interface (TCK/TMS/TDI/TDO). RISC-V debugging accesses DTM via JTAG, then accesses DM via DMI, ultimately controlling the hart.

通过 4 线串行接口（TCK/TMS/TDI/TDO）访问目标设备的调试/测试标准（IEEE 1149.1）。RISC-V 调试通过 JTAG 访问 DTM，再经 DMI 访问 DM，最终控制 hart。

### DTM (Debug Transport Module) / 调试传输模块

A bridge circuit connecting the JTAG interface and Debug Module, translating JTAG packets into DMI (Debug Module Interface) bus operations.

连接 JTAG 接口和 Debug Module 的桥接电路，将 JTAG 数据包翻译为 DMI（Debug Module Interface）总线操作。

### DMI (Debug Module Interface) / 调试模块接口

The internal bus between DTM and DM, allowing the debugger to read/write DM registers.

DTM 与 DM 之间的内部总线，允许调试器读写 DM 的寄存器。

### DM (Debug Module) / 调试模块

Hardware module implementing debug functionality:
- **Halt/Resume**: Halt/resume hart execution
- **Abstract Commands**: Read/write hart's GPR, CSR, memory (without needing to halt for abstract command execution)
- **Program Buffer**: Execute small debug code sequences on hart (up to 16 RISC-V instructions)
- **System Bus Access**: Direct memory access via system bus (without halting hart)

实现调试功能的硬件模块：
- **Halt/Resume**：暂停/恢复 hart 执行
- **Abstract Commands**：读写 hart 的 GPR、CSR、内存（无需暂停执行抽象命令）
- **Program Buffer**：在 hart 上执行小段调试代码（最多 16 条 RISC-V 指令）
- **System Bus Access**：通过系统总线直接访问内存（不暂停 hart）

### abstract command / 抽象命令

High-level operation requests sent by the debugger to the DM, automatically executed by DM without the debugger needing to understand the underlying implementation:
- `Access Register`: Read/write GPR/CSR/FPR/vector registers
- `Quick Access`: Quickly execute Program Buffer
- `Access Memory`: Access memory through hart

调试器向 DM 发送的高级操作请求，DM 自动执行而无需调试器了解底层实现：
- `Access Register`：读写 GPR/CSR/FPR/向量寄存器
- `Quick Access`：快速执行 Program Buffer
- `Access Memory`：通过 hart 访问内存

### Program Buffer / 程序缓冲区

A small instruction buffer (2–16 instructions) provided by DM, where the debugger writes instructions for the hart to execute, used to implement complex debug operations (e.g., memory access requires ld/sd instructions).

DM 提供的小型指令缓冲区（2–16 条指令），调试器写入指令后让 hart 执行，用于实现复杂调试操作（如访问内存需通过 ld/sd 指令）。

### halt group / resume group / 暂停/恢复组

DM supports binding multiple harts into groups; when the debugger halts one hart in the group, other members halt synchronously, used for multi-core synchronized debugging.

DM 支持将多个 hart 绑定为组，调试器暂停组内一个 hart 时，其他成员同步暂停，用于多核同步调试。

---

## Trigger Module (Hardware Breakpoints/Watchpoints) / Trigger 模块（硬件断点/观察点）

### Trigger Module / 触发器模块

Hardware module implementing hardware breakpoints and watchpoints, configured via CSRs `tselect`/`tdata1`/`tdata2`/`tdata3`.

实现硬件断点（Hardware Breakpoint）和观察点（Watchpoint）的硬件模块，通过 CSR `tselect`/`tdata1`/`tdata2`/`tdata3` 配置。

### tselect / tdata1 / tdata2 / 触发器 CSR

- **tselect**: Selects the trigger number currently being configured
- **tdata1**: Trigger type and control (type/dmode/execute/load/store/timing)
- **tdata2**: Trigger comparison value (address or data value)
- **tdata3**: Extended trigger conditions (optional)

- **tselect**：选择当前配置的触发器编号
- **tdata1**：触发器类型和控制（type/dmode/execute/load/store/timing）
- **tdata2**：触发比较值（地址或数据值）
- **tdata3**：扩展触发条件（可选）

### mcontrol / mcontrol6 / 内存访问触发器

Most commonly used trigger type, triggering a trap or entering debug mode when instruction execution (execute) or data access (load/store) at a specific address occurs:
- `execute=1, address=0x80001000` → Address breakpoint (hardware breakpoint)
- `load=1/store=1, address=X` → Read/write watchpoint

最常用的触发器类型，在指令执行（execute）或数据访问（load/store）特定地址时触发 trap 或进入调试模式：
- `execute=1, address=0x80001000` → 地址断点（硬件断点）
- `load=1/store=1, address=X` → 读/写观察点（Watchpoint）

### icount / 指令计数触发器

Triggers after executing a specified number of instructions, used for single-stepping (icount=1) and performance analysis.

执行指定条数指令后触发，用于单步执行（icount=1）和性能分析。

### itrigger / etrigger / 中断/异常触发器

Triggers when a specific interrupt or exception occurs, used to debug hard-to-reproduce exception conditions.

在特定中断或异常发生时触发，用于调试难以重现的异常条件。

---

## Trace (Execution Trace) / Trace（执行跟踪）

### RISC-V Efficient Trace (E-Trace) / RISC-V 高效跟踪

RISC-V instruction flow trace specification that reconstructs execution paths by recording **branch** information (rather than all instructions), significantly reducing trace data volume. Only records non-sequential execution points (conditional branch outcomes, indirect jump targets), allowing full execution flow reconstruction when combined with binary files.

RISC-V 指令流跟踪规范，通过记录**分支**信息（而非全部指令）重建执行路径，大幅减少跟踪数据量。只记录非顺序执行点（条件分支结果、间接跳转目标），配合二进制文件即可重建完整执行流。

### N-Trace (Nexus Trace) / Nexus 跟踪

RISC-V trace solution based on IEEE-ISTO 5001 (Nexus) standard, compatible with existing Nexus debug tool ecosystem.

基于 IEEE-ISTO 5001（Nexus）标准的 RISC-V 跟踪方案，兼容现有 Nexus 调试工具生态。

### trace encoder / 跟踪编码器

Hardware module that compresses and encodes hart execution information (PC values, branch directions) into a trace data stream.

将 hart 的执行信息（PC 值、分支方向）压缩编码为跟踪数据流的硬件模块。

### ETM (Embedded Trace Macrocell) / 嵌入式跟踪宏单元

Trace terminology from ARM ecosystem; similar concept in RISC-V is implemented by E-Trace encoder.

ARM 生态中的跟踪术语，类似概念在 RISC-V 中由 E-Trace encoder 实现。

---

## Software Debug Tools / 软件调试工具

### OpenOCD / OpenOCD

Open On-Chip Debugger, an open-source debugger that connects to RISC-V DM via JTAG adapter, providing remote debugging services for GDB. RISC-V support branches have been merged upstream.

开源片上调试器（Open On-Chip Debugger），通过 JTAG 适配器连接 RISC-V DM，为 GDB 提供远程调试服务。支持 RISC-V 的 OpenOCD 分支被上游合并。

### GDB RISC-V support / GDB RISC-V 支持

GNU Debugger supports RISC-V architecture (32/64-bit), connects to OpenOCD via `target remote` for hardware debugging, or directly debugs QEMU emulation environments.

GNU Debugger 支持 RISC-V 架构（32/64位），通过 `target remote` 连接 OpenOCD 进行硬件调试，或直接调试 QEMU 仿真环境。

### QEMU / QEMU

Open-source processor emulator supporting RISC-V 32/64-bit, can emulate full systems (`qemu-system-riscv64`) or user-mode (`qemu-riscv64`). Commonly used for kernel and firmware development/debugging when hardware is unavailable.

开源处理器模拟器，支持 RISC-V 32/64 位，可仿真完整系统（`qemu-system-riscv64`）或用户态（`qemu-riscv64`）。常用于无硬件时开发调试内核和固件。

### spike / spike

RISC-V official reference simulator (implemented in C++), accurately simulates ISA behavior, serving as the golden reference for verifying new extension implementation correctness.

RISC-V 官方参考模拟器（用 C++ 实现），精确模拟 ISA 行为，是验证新扩展实现正确性的黄金参考。

### riscv-tests / riscv-tests

RISC-V official instruction-level test suite, containing assembly-level functional tests for every instruction and extension, used to verify processor implementation correctness.

RISC-V 官方指令级测试套件，包含对每条指令和每个扩展的汇编级功能测试，用于验证处理器实现的正确性。

---

## Debug Module (DM) Details / 调试模块详解

### Debug Module Registers / 调试模块寄存器

The Debug Module exposes a set of memory-mapped registers accessible via DMI for controlling debug operations.

调试模块通过 DMI 暴露一组内存映射寄存器，用于控制调试操作。

**Core DM Registers / 核心 DM 寄存器：**

| Address | Register | Description / 说明 |
|---------|----------|-------------------|
| 0x04 | `data0`-`data11` | Data registers for abstract commands / 抽象命令的数据寄存器 |
| 0x10 | `dmcontrol` | Debug module control / 调试模块控制 |
| 0x11 | `dmstatus` | Debug module status / 调试模块状态 |
| 0x12 | `hartinfo` | Hart information / Hart 信息 |
| 0x13 | `haltsum1` | Halt summary (groups of 32 harts) / 暂停摘要（32 个 hart 为一组）|
| 0x16 | `hawindowsel` | Hart array window select / Hart 数组窗口选择 |
| 0x17 | `command` | Abstract command / 抽象命令 |
| 0x18 | `abstractcs` | Abstract command status / 抽象命令状态 |
| 0x19 | `abstractauto` | Abstract command auto-execute / 抽象命令自动执行 |
| 0x1D | `nextdm` | Next debug module address / 下一个调试模块地址 |
| 0x20 | `progbuf0`-`progbuf15` | Program buffer / 程序缓冲区 |
| 0x38 | `sbcs` | System bus access control / 系统总线访问控制 |
| 0x39 | `sbaddress0` | System bus address (low) / 系统总线地址（低位）|
| 0x3A | `sbaddress1` | System bus address (high) / 系统总线地址（高位）|
| 0x3C | `sbdata0` | System bus data / 系统总线数据 |

---

### dmcontrol Register / dmcontrol 寄存器

Controls the overall operation of the Debug Module.

控制调试模块的整体操作。

**Key Fields / 关键字段：**

| Bit(s) | Field | Description / 说明 |
|--------|-------|-------------------|
| 31 | `haltreq` | Request selected harts to halt / 请求选定的 hart 暂停 |
| 30 | `resumereq` | Request selected harts to resume / 请求选定的 hart 恢复 |
| 29 | `hartreset` | Reset selected harts / 复位选定的 hart |
| 28 | `ackhavereset` | Acknowledge hart reset / 确认 hart 复位 |
| 27 | `hasel` | Hart array select enable / Hart 数组选择使能 |
| 25:16 | `hartsello` | Hart select (low 10 bits) / Hart 选择（低 10 位）|
| 15:6 | `hartselhi` | Hart select (high 10 bits) / Hart 选择（高 10 位）|
| 3 | `ndmreset` | System reset (not debug module) / 系统复位（非调试模块）|
| 1 | `clrresethaltreq` | Clear reset halt request / 清除复位暂停请求 |
| 0 | `dmactive` | Debug module active / 调试模块激活 |

---

### dmstatus Register / dmstatus 寄存器

Reports the current status of the Debug Module and selected harts.

报告调试模块和选定 hart 的当前状态。

**Key Fields / 关键字段：**

| Bit(s) | Field | Description / 说明 |
|--------|-------|-------------------|
| 23 | `impebreak` | Implicit ebreak in program buffer / 程序缓冲区中的隐式 ebreak |
| 22 | `allhavereset` | All selected harts have been reset / 所有选定的 hart 已复位 |
| 21 | `anyhavereset` | Any selected hart has been reset / 任何选定的 hart 已复位 |
| 20 | `allresumeack` | All selected harts have resumed / 所有选定的 hart 已恢复 |
| 19 | `anyresumeack` | Any selected hart has resumed / 任何选定的 hart 已恢复 |
| 18 | `allnonexistent` | All selected harts don't exist / 所有选定的 hart 不存在 |
| 17 | `anynonexistent` | Any selected hart doesn't exist / 任何选定的 hart 不存在 |
| 16 | `allunavail` | All selected harts unavailable / 所有选定的 hart 不可用 |
| 15 | `anyunavail` | Any selected hart unavailable / 任何选定的 hart 不可用 |
| 14 | `allrunning` | All selected harts running / 所有选定的 hart 正在运行 |
| 13 | `anyrunning` | Any selected hart running / 任何选定的 hart 正在运行 |
| 12 | `allhalted` | All selected harts halted / 所有选定的 hart 已暂停 |
| 11 | `anyhalted` | Any selected hart halted / 任何选定的 hart 已暂停 |
| 8 | `authenticated` | Authentication passed / 已通过身份验证 |
| 7 | `authbusy` | Authentication busy / 身份验证忙 |
| 6 | `hasresethaltreq` | Has reset halt request / 有复位暂停请求 |
| 5 | `confstrptrvalid` | Configuration string pointer valid / 配置字符串指针有效 |
| 3:0 | `version` | Debug spec version / 调试规范版本 |

---

### Abstract Commands / 抽象命令

High-level operations that the debugger can request the DM to perform.

调试器可以请求 DM 执行的高级操作。

**Command Types / 命令类型：**

| cmdtype | Command | Description / 说明 |
|---------|---------|-------------------|
| 0 | Access Register | Read/write GPR, CSR, FPR / 读写 GPR、CSR、FPR |
| 1 | Quick Access | Execute program buffer / 执行程序缓冲区 |
| 2 | Access Memory | Read/write memory / 读写内存 |

**Access Register Command Format / 访问寄存器命令格式：**

| Bit(s) | Field | Description / 说明 |
|--------|-------|-------------------|
| 31:24 | `cmdtype` | 0 for Access Register / 访问寄存器为 0 |
| 23 | `aarsize` | Access size (0=32-bit, 1=64-bit, 2=128-bit) / 访问大小 |
| 22:20 | `aarpostincrement` | Post-increment regno / 后递增 regno |
| 19 | `postexec` | Execute program buffer after / 之后执行程序缓冲区 |
| 18 | `transfer` | Perform transfer / 执行传输 |
| 17 | `write` | 1=write, 0=read / 1=写，0=读 |
| 15:0 | `regno` | Register number / 寄存器编号 |

**Register Number Encoding / 寄存器编号编码：**
- `0x0000-0x0FFF`: CSRs / CSR 寄存器
- `0x1000-0x101F`: GPRs (x0-x31) / 通用寄存器（x0-x31）
- `0x1020-0x103F`: FPRs (f0-f31) / 浮点寄存器（f0-f31）

**Example: Read x10 (a0) / 示例：读取 x10（a0）：**
```
1. Write command register: 0x0022100A
   - cmdtype=0 (Access Register)
   - aarsize=2 (32-bit)
   - transfer=1
   - write=0 (read)
   - regno=0x100A (x10)
   
2. Wait for abstractcs.busy=0
3. Read data0 register to get x10 value
```

---

### Program Buffer / 程序缓冲区详解

A small memory region where the debugger can write instructions for the hart to execute.

调试器可以在其中写入指令供 hart 执行的小内存区域。

**Program Buffer Size / 程序缓冲区大小：**
- Minimum: 2 instructions / 最小：2 条指令
- Typical: 16 instructions / 典型：16 条指令
- Size reported in `hartinfo.progbufsize` / 大小在 `hartinfo.progbufsize` 中报告

**Usage Pattern / 使用模式：**
1. Debugger writes instructions to `progbuf0`-`progbufN` / 调试器将指令写入 `progbuf0`-`progbufN`
2. Debugger issues abstract command with `postexec=1` / 调试器发出 `postexec=1` 的抽象命令
3. Hart executes program buffer in debug mode / Hart 在调试模式下执行程序缓冲区
4. Hart returns to debug mode after execution / 执行后 hart 返回调试模式

**Example: Write to Memory / 示例：写入内存：**
```assembly
# Write value in s0 to address in s1
progbuf0: sw s0, 0(s1)
progbuf1: ebreak  # Return to debug mode
```

---

### System Bus Access / 系统总线访问

Allows the debugger to directly access system memory without halting the hart.

允许调试器直接访问系统内存，无需暂停 hart。

**System Bus Registers / 系统总线寄存器：**
- `sbcs`: System Bus Access Control and Status / 系统总线访问控制和状态
- `sbaddress0-3`: 128-bit address / 128 位地址
- `sbdata0-3`: 128-bit data / 128 位数据

**sbcs Register Fields / sbcs 寄存器字段：**

| Bit(s) | Field | Description / 说明 |
|--------|-------|-------------------|
| 29 | `sbbusy` | System bus access in progress / 系统总线访问进行中 |
| 22 | `sberror` | System bus error / 系统总线错误 |
| 21 | `sbautoread` | Auto-read on address write / 地址写入时自动读取 |
| 20 | `sbautoincrement` | Auto-increment address / 自动递增地址 |
| 19:17 | `sbaccess` | Access size (0=8bit, 1=16bit, 2=32bit, 3=64bit, 4=128bit) / 访问大小 |
| 14:12 | `sbasize` | System bus address width / 系统总线地址宽度 |

**Usage Example / 使用示例：**
```
# Read 32-bit value from address 0x80000000
1. Write sbcs: sbaccess=2 (32-bit), sbautoread=1
2. Write sbaddress0 = 0x80000000
3. Read sbdata0 (auto-read triggers on address write)
```

---

## JTAG Interface Details / JTAG 接口详解

### JTAG Signals / JTAG 信号

| Signal | Direction | Description / 说明 |
|--------|-----------|-------------------|
| **TCK** | Input | Test Clock / 测试时钟 |
| **TMS** | Input | Test Mode Select / 测试模式选择 |
| **TDI** | Input | Test Data In / 测试数据输入 |
| **TDO** | Output | Test Data Out / 测试数据输出 |
| **TRST** | Input (optional) | Test Reset / 测试复位 |

---

### JTAG TAP States / JTAG TAP 状态

The JTAG Test Access Port (TAP) operates as a state machine:

JTAG 测试访问端口（TAP）作为状态机运行：

```
     Test-Logic-Reset
           ↓
      Run-Test/Idle
       ↙         ↘
Select-DR    Select-IR
   ↓             ↓
Capture-DR   Capture-IR
   ↓             ↓
Shift-DR     Shift-IR
   ↓             ↓
Exit1-DR     Exit1-IR
   ↓             ↓
Pause-DR     Pause-IR
   ↓             ↓
Exit2-DR     Exit2-IR
   ↓             ↓
Update-DR    Update-IR
```

---

### JTAG Instructions / JTAG 指令

RISC-V Debug defines mandatory and optional JTAG instructions:

RISC-V 调试定义了必需和可选的 JTAG 指令：

**Mandatory Instructions / 必需指令：**

| Instruction | IR Value | Description / 说明 |
|-------------|----------|-------------------|
| **BYPASS** | All 1's | Bypass DR for faster scans / 旁路 DR 以加快扫描 |
| **IDCODE** | 0x01 | Read device identification / 读取设备标识 |
| **DTMCS** | 0x10 | DTM Control and Status / DTM 控制和状态 |
| **DMI** | 0x11 | Debug Module Interface access / 调试模块接口访问 |

**Optional Instructions / 可选指令：**
- **BYPASS** (alternate encoding) / 旁路（备用编码）
- Custom vendor instructions / 自定义供应商指令

---

### DTMCS Register / DTMCS 寄存器

DTM Control and Status register accessed via JTAG DTMCS instruction.

通过 JTAG DTMCS 指令访问的 DTM 控制和状态寄存器。

**Fields / 字段：**

| Bit(s) | Field | Description / 说明 |
|--------|-------|-------------------|
| 31:18 | `dmihardreset` | Hard reset DMI / 硬复位 DMI |
| 17 | `dmireset` | Reset DMI / 复位 DMI |
| 16 | `idle` | Minimum idle cycles / 最小空闲周期 |
| 15:12 | `dmistat` | DMI status (0=no error, 3=busy) / DMI 状态 |
| 11:10 | `abits` | DMI address width / DMI 地址宽度 |
| 3:0 | `version` | DTM version / DTM 版本 |

---

### DMI Access / DMI 访问

The DMI register provides access to Debug Module registers via JTAG.

DMI 寄存器通过 JTAG 提供对调试模块寄存器的访问。

**DMI Scan Format / DMI 扫描格式：**
```
[address (abits)] [data (32 bits)] [op (2 bits)]
```

**DMI Operations / DMI 操作：**
- `00`: No operation / 无操作
- `01`: Read / 读
- `10`: Write / 写
- `11`: Reserved / 保留

**Example: Read dmstatus (address 0x11) / 示例：读取 dmstatus（地址 0x11）：**
```
1. Shift-IR: Load DMI instruction (0x11)
2. Shift-DR: [0x11][0x00000000][01] (read dmstatus)
3. Wait idle cycles
4. Shift-DR: [0x00][0x00000000][00] (capture result)
5. Result in data field
```

---

## Trigger Module Details / 触发器模块详解

### Trigger CSRs / 触发器 CSR

| CSR Name | Address | Privilege | Description / 说明 |
|----------|---------|-----------|-------------------|
| `tselect` | 0x7A0 | M/S/D | Select trigger / 选择触发器 |
| `tdata1` | 0x7A1 | M/S/D | Trigger data 1 / 触发器数据 1 |
| `tdata2` | 0x7A2 | M/S/D | Trigger data 2 / 触发器数据 2 |
| `tdata3` | 0x7A3 | M/S/D | Trigger data 3 / 触发器数据 3 |
| `tinfo` | 0x7A4 | M/S/D | Trigger info / 触发器信息 |
| `tcontrol` | 0x7A5 | M | Trigger control / 触发器控制 |
| `mcontext` | 0x7A8 | M | Machine context / 机器上下文 |
| `scontext` | 0x7AA | S | Supervisor context / 监管者上下文 |

---

### Trigger Types / 触发器类型

| Type | Name | Description / 说明 |
|------|------|-------------------|
| 0 | None | No trigger / 无触发器 |
| 2 | mcontrol | Address/data match (legacy) / 地址/数据匹配（传统）|
| 3 | icount | Instruction count / 指令计数 |
| 4 | itrigger | Interrupt trigger / 中断触发 |
| 5 | etrigger | Exception trigger / 异常触发 |
| 6 | mcontrol6 | Enhanced address/data match / 增强地址/数据匹配 |
| 7 | tmexttrigger | External trigger / 外部触发 |
| 8-14 | Reserved | Reserved for future / 为将来保留 |
| 15 | Disabled | Disabled trigger / 禁用的触发器 |

---

### mcontrol6 (Type 6) / mcontrol6 触发器

Enhanced address/data match trigger replacing mcontrol (Type 2).

增强的地址/数据匹配触发器，替代 mcontrol（类型 2）。

**tdata1 Layout for mcontrol6 / mcontrol6 的 tdata1 布局：**

| Bit(s) | Field | Description / 说明 |
|--------|-------|-------------------|
| XLEN-1:XLEN-4 | `type` | 6 for mcontrol6 / mcontrol6 为 6 |
| XLEN-5 | `dmode` | Debug-only access / 仅调试访问 |
| XLEN-11:XLEN-6 | `maskmax` | Maximum mask width / 最大掩码宽度 |
| 26 | `hit` | Trigger hit indicator / 触发器命中指示器 |
| 25 | `vs` | Trigger in VS-mode / 在 VS 模式下触发 |
| 24 | `vu` | Trigger in VU-mode / 在 VU 模式下触发 |
| 22 | `hit1` | Hit indicator 1 / 命中指示器 1 |
| 21 | `hit0` | Hit indicator 0 / 命中指示器 0 |
| 20:18 | `select` | Address/data select / 地址/数据选择 |
| 17:12 | `timing` | Trigger timing / 触发时机 |
| 11:7 | `size` | Access size / 访问大小 |
| 6 | `action` | Debug exception / 调试异常 |
| 5 | `chain` | Chain with next trigger / 与下一个触发器链接 |
| 4 | `match` | Match condition / 匹配条件 |
| 3 | `m` | Enable in M-mode / 在 M 模式下启用 |
| 2 | `s` | Enable in S-mode / 在 S 模式下启用 |
| 1 | `u` | Enable in U-mode / 在 U 模式下启用 |
| 0 | `execute/load/store` | Trigger on execute/load/store / 在执行/加载/存储时触发 |

**tdata2: Comparison Value / 比较值**
- For address match: address to compare / 对于地址匹配：要比较的地址
- For data match: data value to compare / 对于数据匹配：要比较的数据值

---

### Trigger Chaining / 触发器链接

Multiple triggers can be chained together to create complex conditions using the `chain` bit.

可以使用 `chain` 位将多个触发器链接在一起以创建复杂条件。

**Example: Break on write to address X with data value Y / 示例：在向地址 X 写入数据值 Y 时中断：**
```
Trigger 0: match address X, chain=1
Trigger 1: match data Y, chain=0
```
Both conditions must be true simultaneously.
两个条件必须同时为真。

---

## Trace Specifications / 跟踪规范

### RISC-V Processor Trace (E-Trace) / RISC-V 处理器跟踪

Efficient trace specification that reconstructs instruction flow by encoding only non-sequential execution (branches, exceptions).

高效跟踪规范，通过仅编码非顺序执行（分支、异常）来重建指令流。

**Key Concepts / 关键概念：**
- **Instruction Delta Trace**: Only trace branch outcomes / 仅跟踪分支结果
- **Lossless Compression**: Minimal bandwidth using branch maps / 使用分支映射的最小带宽
- **Offline Reconstruction**: Combine trace + ELF to recover execution / 结合跟踪 + ELF 恢复执行

---

### Trace Packet Types / 跟踪数据包类型

| Packet Type | Description / 说明 |
|-------------|-------------------|
| **Format 3** | Synchronization / 同步 |
| **Format 2** | Branch map / 分支映射 |
| **Format 1** | Differential branch / 差分分支 |
| **Format 0** | Address / 地址 |

**Format 3 (Sync) / 格式 3（同步）：**
- Full PC value / 完整 PC 值
- Context information / 上下文信息
- Sent periodically for synchronization / 定期发送以进行同步

**Format 2 (Branch Map) / 格式 2（分支映射）：**
- Bitmap of branch outcomes (taken/not taken) / 分支结果的位图（采用/未采用）
- Up to 31 branches per packet / 每个数据包最多 31 个分支

**Format 1 (Differential Branch) / 格式 1（差分分支）：**
- Branch count since last packet / 自上一个数据包以来的分支计数
- Used when branches fit in small encoding / 当分支适合小编码时使用

**Format 0 (Address) / 格式 0（地址）：**
- Target address for indirect jumps / 间接跳转的目标地址
- Exception/interrupt handler address / 异常/中断处理程序地址

---

### Trace Encoder / 跟踪编码器

Hardware module that monitors hart execution and generates trace packets.

监视 hart 执行并生成跟踪数据包的硬件模块。

**Encoder Components / 编码器组件：**
- **Ingress Port**: Receives execution information from hart / 从 hart 接收执行信息
- **Compression Logic**: Encodes trace packets / 编码跟踪数据包
- **FIFO Buffer**: Stores packets temporarily / 临时存储数据包
- **Egress Port**: Outputs trace stream / 输出跟踪流

**Trace Transport / 跟踪传输：**
- **On-chip RAM**: Store trace in memory / 在内存中存储跟踪
- **AMBA ATB**: ARM Trace Bus / ARM 跟踪总线
- **MIPI STP**: MIPI System Trace Protocol / MIPI 系统跟踪协议
- **USB**: High-speed off-chip transfer / 高速片外传输

---

### Trace Filtering / 跟踪过滤

Mechanisms to reduce trace bandwidth by selective recording.

通过选择性记录减少跟踪带宽的机制。

**Filter Types / 过滤器类型：**
- **Address Range**: Only trace specific code regions / 仅跟踪特定代码区域
- **Privilege Mode**: Only trace M/S/U mode / 仅跟踪 M/S/U 模式
- **Context ID**: Filter by ASID or VMID / 按 ASID 或 VMID 过滤
- **Instruction Type**: Only trace branches/loads/stores / 仅跟踪分支/加载/存储

---

### N-Trace (Nexus Trace) / N-跟踪

Alternative trace specification based on IEEE-ISTO 5001 Nexus standard.

基于 IEEE-ISTO 5001 Nexus 标准的替代跟踪规范。

**Nexus Message Types / Nexus 消息类型：**
- **Program Trace**: Instruction execution / 指令执行
- **Data Trace**: Data read/write / 数据读/写
- **Ownership Trace**: Task switching / 任务切换
- **Resource Full**: Buffer overflow / 缓冲区溢出

**Advantages / 优势：**
- Industry standard / 行业标准
- Existing tool support / 现有工具支持
- Unified debug/trace infrastructure / 统一调试/跟踪基础设施

---

## Debug Tools Ecosystem / 调试工具生态

### OpenOCD Configuration / OpenOCD 配置

**Interface Configuration / 接口配置：**
```tcl
# FTDI-based JTAG adapter
adapter driver ftdi
ftdi device_desc "Dual RS232-HS"
ftdi vid_pid 0x0403 0x6010
ftdi layout_init 0x0008 0x001b
ftdi layout_signal nTRST -data 0x0100 -oe 0x0100
```

**Target Configuration / 目标配置：**
```tcl
# RISC-V target
set _CHIPNAME riscv
jtag newtap $_CHIPNAME cpu -irlen 5 -expected-id 0x10e31913

set _TARGETNAME $_CHIPNAME.cpu
target create $_TARGETNAME riscv -chain-position $_TARGETNAME

# Hart configuration
riscv set_prefer_sba on
riscv set_command_timeout_sec 10

init
```

---

### GDB Remote Protocol / GDB 远程协议

GDB communicates with OpenOCD using the GDB Remote Serial Protocol (RSP).

GDB 使用 GDB 远程串行协议（RSP）与 OpenOCD 通信。

**Connection / 连接：**
```bash
# In OpenOCD
telnet_port 4444
gdb_port 3333

# In GDB
riscv64-unknown-elf-gdb
(gdb) target extended-remote localhost:3333
(gdb) load
(gdb) continue
```

**Common GDB Commands / 常用 GDB 命令：**
```gdb
# Hardware breakpoint
hbreak main

# Watchpoint
watch variable

# Read register
info registers

# Read memory
x/10x 0x80000000

# Single step
stepi

# Continue execution
continue
```

---

### Lauterbach TRACE32 / Lauterbach TRACE32

Commercial debug and trace tool with comprehensive RISC-V support.

商业调试和跟踪工具，全面支持 RISC-V。

**Features / 特性：**
- JTAG debugging / JTAG 调试
- Instruction trace / 指令跟踪
- Power trace / 功耗跟踪
- Code coverage / 代码覆盖率
- Profiling / 性能分析

---

### Segger J-Link / Segger J-Link

Popular commercial JTAG debugger with RISC-V support.

流行的商业 JTAG 调试器，支持 RISC-V。

**Supported Features / 支持的功能：**
- Unlimited breakpoints (flash breakpoints) / 无限断点（闪存断点）
- Real-time terminal (RTT) / 实时终端（RTT）
- Instruction trace (on select devices) / 指令跟踪（在选定设备上）
- GDB server / GDB 服务器

---

**Last Updated / 最后更新：** 2026-07-23

---

## Debug Extensions (Formal Names) / 调试扩展（正式名称）

### Sdext (Debug Extension) / 调试扩展

Ratified February 2025. Formalizes the external debug specification as a standard ISA extension with the name "Sdext".

2025年2月批准。将外部调试规范正式化为标准ISA扩展，名称为"Sdext"。

**Includes / 包含：**
- Debug Module (DM) specification
- Debug Transport Module (DTM) specification  
- Abstract commands (Access Register, Quick Access, Access Memory)
- Program Buffer execution
- System Bus Access
- Debug Module Interface (DMI)

调试模块（DM）规范、调试传输模块（DTM）规范、抽象命令、Program Buffer执行、系统总线访问、调试模块接口（DMI）

### Sdtrig (Debug Triggers) / 调试触发器扩展

Ratified February 2025. Formalizes the trigger module specification as a standard ISA extension with the name "Sdtrig".

2025年2月批准。将触发器模块规范正式化为标准ISA扩展，名称为"Sdtrig"。

**Includes / 包含：**
- Trigger CSRs: tselect, tdata1, tdata2, tdata3, tinfo, tcontrol
- Trigger types: mcontrol, mcontrol6, icount, itrigger, etrigger
- Trigger chaining and priorities
- Hardware breakpoints and watchpoints

触发器CSR：tselect、tdata1、tdata2、tdata3、tinfo、tcontrol
触发器类型：mcontrol、mcontrol6、icount、itrigger、etrigger
触发器链接和优先级、硬件断点和观察点

**Note / 注意：**
These are formal extension names for specifications that were already ratified. The functionality was previously documented but now has official extension names for version checking and feature detection.

这些是已批准规范的正式扩展名称。功能之前已记录但现在有正式扩展名称用于版本检查和特性检测。

