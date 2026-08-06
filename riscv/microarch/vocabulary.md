---
tags: [riscv, microarch, vocabulary]
domain: microarch
last_updated: 2026-07-22
source: https://riscv.org/
---

# RISC-V Microarchitecture Vocabulary / RISC-V 微架构词汇表

> Covers: pipelining, branch prediction, cache, out-of-order execution, Fence/memory ordering
> Note: Microarchitecture is implementation-specific, not mandated by ISA specification

> 涵盖：流水线、分支预测、缓存、乱序执行、Fence/内存序
> 注：微架构是实现层面，不由 ISA 规范强制定义，各实现可自由选择

---

## Pipeline Fundamentals / 流水线基础

### pipeline / 流水线

Microarchitecture technique that decomposes instruction execution into multiple stages that advance in parallel. Classic 5-stage pipeline: IF (Instruction Fetch) → ID (Instruction Decode) → EX (Execute) → MEM (Memory) → WB (Write Back).

将指令执行分解为多个阶段并行推进的微架构技术。经典 5 级流水线：IF（取指）→ ID（译码）→ EX（执行）→ MEM（访存）→ WB（写回）。

### IF (Instruction Fetch) / 取指

Fetches instruction encoding from memory (I-cache) pointed to by PC. RISC-V's C extension requires fetch unit to support mixed 16/32-bit instructions.

从 PC 指向的内存（I-cache）取出指令编码。RISC-V 的 C 扩展要求取指单元支持 16/32 位混合指令。

### ID (Instruction Decode) / 译码

Parses instruction encoding and extracts opcode/funct3/funct7/rd/rs1/rs2/imm fields. RISC-V's fixed immediate sign-bit position design reduces latency in this stage.

解析指令编码，提取 opcode/funct3/funct7/rd/rs1/rs2/imm。RISC-V 的固定立即数符号位位置设计降低了此阶段延迟。

### EX (Execute) / 执行

ALU operations (addition, subtraction, logic, shifts), address calculation, branch condition evaluation.

ALU 运算（加减法、逻辑、移位）、地址计算、分支条件判断。

### MEM / 访存

Performs memory read/write for Load/Store instructions, accessing D-cache.

执行 Load/Store 指令的内存读写，访问 D-cache。

### WB (Write Back) / 写回

Writes computation result or Load data back to destination register.

将计算结果或 Load 数据写回目的寄存器。

### pipeline hazard / 流水线冒险

Conditions that disrupt sequential pipeline execution, classified into three types:
- **Structural hazard**: Multiple instructions compete for the same hardware resource
- **Data hazard**: RAW/WAR/WAW dependency relationships
- **Control hazard**: Incorrect instruction fetch caused by branches/jumps

影响流水线顺序执行的条件，分三类：
- **结构冒险**（Structural）：多条指令竞争同一硬件资源
- **数据冒险**（Data）：RAW/WAR/WAW 依赖关系
- **控制冒险**（Control）：分支/跳转导致的取指错误

### data forwarding / 数据前递（旁路）

Directly forwards execution results to subsequent instruction inputs before write-back completes, eliminating most RAW data hazards and avoiding pipeline stalls.

在写回完成前将执行结果直接转发给后续指令的输入，消除大多数 RAW 数据冒险，避免流水线停顿（stall）。

### stall / 流水线停顿

When hazards cannot be resolved through forwarding, inserts bubbles (NOPs) to pause pipeline advancement, losing CPI efficiency.

当冒险无法通过前递解决时，插入气泡（NOP）暂停流水线推进，损失 CPI。

### CPI (Cycles Per Instruction) / 每指令周期数

Metric measuring processor efficiency. Ideal pipeline CPI=1, actual is > 1 due to hazards, cache misses, etc. Out-of-order processors can achieve CPI < 1 under certain conditions (executing multiple instructions per cycle, IPC > 1).

衡量处理器效率的指标。理想流水线 CPI=1，实际因冒险、缓存缺失等 > 1。乱序处理器可在某些条件下 CPI < 1（每周期执行多条指令，IPC > 1）。

### IPC (Instructions Per Cycle) / 每周期指令数

Reciprocal of CPI, common performance metric for superscalar processors (e.g., IPC=3 means average of 3 instructions retired per cycle).

CPI 的倒数，超标量处理器的常用性能指标（如 IPC=3 表示平均每周期退休 3 条指令）。

---

## Branch Prediction / 分支预测

### branch prediction / 分支预测

Predicts branch direction/target before branch result is determined, reducing pipeline flush caused by control hazards.

在分支结果确定前预测跳转方向/目标，减少控制冒险导致的流水线冲刷（flush）。

### branch predictor / 分支预测器

Hardware structures implementing branch prediction:
- **BHT** (Branch History Table): Predicts direction based on history (2-bit saturating counter)
- **BTB** (Branch Target Buffer): Caches branch target addresses
- **RAS** (Return Address Stack): Predicts return address for `ret` (depth typically 8–16 entries)

实现分支预测的硬件结构：
- **BHT**（Branch History Table）：基于历史预测方向（2bit 饱和计数器）
- **BTB**（Branch Target Buffer）：缓存跳转目标地址
- **RAS**（Return Address Stack）：预测 `ret` 的返回地址（深度通常 8–16 项）

### misprediction penalty / 预测失败惩罚

Additional cycles required to flush pipeline and re-fetch from correct address when branch prediction fails. 5-stage pipeline: ~3–5 cycles, deep pipelines: up to 15–20 cycles.

分支预测错误时需冲刷流水线并从正确地址重新取指的额外周期数。5 级流水线约 3–5 周期，深流水线可达 15–20 周期。

### conditional branch / 条件分支

RISC-V B-format instructions (`BEQ`/`BNE`/`BLT`/`BGE`/`BLTU`/`BGEU`), direction determined by register comparison result.

RISC-V 的 B 格式指令（`BEQ`/`BNE`/`BLT`/`BGE`/`BLTU`/`BGEU`），方向由寄存器比较结果决定。

---

## Cache / 缓存

### cache / 缓存

High-speed small-capacity storage between processor and main memory, exploiting temporal and spatial locality to reduce average memory access latency.

位于处理器和主存之间的高速小容量存储器，利用时间局部性和空间局部性减少平均访存延迟。

### cache line / 缓存行

Minimum data unit in cache, data block loaded from memory at once (typically 64 bytes).

缓存的最小数据单元，一次从内存加载的数据块（通常 64 字节）。

### I-cache / D-cache / 指令缓存 / 数据缓存

Separate instruction fetch and data access caches (Harvard cache architecture), avoiding competition for the same port between instruction fetch and memory access.

分离的取指和数据访问缓存（哈佛缓存结构），避免取指和访存竞争同一端口。

### TLB (Translation Lookaside Buffer) / 快表

→ See [[privileged/vocabulary]]

→ 详见 [[privileged/vocabulary]]

### cache coherency / 缓存一致性

Mechanism ensuring consistent cache views across cores in multi-core systems:
- **MESI protocol**: Four-state cache coherency protocol (Modified/Exclusive/Shared/Invalid)

多核系统中保证各核心缓存视图一致的机制：
- **MESI 协议**：Modified/Exclusive/Shared/Invalid 四状态缓存一致性协议
- RISC-V 本身不规定缓存一致性实现，但 RVWMO 内存模型定义了软件可见语义

### VIPT / PIPT / 缓存索引与标记方式

Cache indexing and tagging schemes:
- **VIPT** (Virtually Indexed Physically Tagged): Uses virtual address for indexing and physical address for tagging. Good performance but requires handling aliasing issues.
- **PIPT** (Physically Indexed Physically Tagged): Uses physical address for both indexing and tagging. No aliasing issues but requires address translation first.

缓存索引与标记方式：
- **VIPT**（Virtually Indexed Physically Tagged）：用虚拟地址索引、物理地址标记，性能好但需处理别名问题
- **PIPT**（Physically Indexed Physically Tagged）：完全物理地址，无别名但需先翻译地址

---

## Out-of-Order Execution / 乱序执行

### out-of-order execution (OoO) / 乱序执行

Processor executes instructions based on operand readiness rather than program order, dynamically scheduling to exploit instruction-level parallelism (ILP).

处理器不按程序顺序执行指令，而是按操作数就绪情况动态调度，挖掘指令级并行（ILP）。

### ROB (Reorder Buffer) / 重排序缓冲区

Circular buffer tracking all in-flight instruction states, ensuring instructions commit in program order to enable precise exceptions.

跟踪所有 in-flight 指令状态的循环缓冲区，确保指令按程序顺序提交（retire），实现精确异常。

### reservation station / 保留站

Instruction queue waiting for operands to become ready, issuing to execution units once operands are available.

等待操作数就绪的指令队列，操作数准备好后向执行单元发射（issue）。

### register renaming / 寄存器重命名

Maps architectural registers (ISA-visible x0–x31) to a larger physical register file, eliminating WAW and WAR false dependencies—a key prerequisite for out-of-order execution.

将逻辑寄存器（ISA 可见的 x0–x31）映射到更大的物理寄存器堆，消除 WAW 和 WAR 假相关，是乱序执行的关键前提。

### superscalar / 超标量

Processor capable of fetching, issuing, and executing multiple instructions per clock cycle. RISC-V's concise instruction encoding facilitates wide superscalar implementations.

每个时钟周期可取指、发射、执行多条指令的处理器。RISC-V 简洁的指令编码有利于宽度超标量实现。

### in-order / 顺序执行

Simple implementation that fetches, executes, and commits instructions in program order. Small area and low power consumption, suitable for embedded applications (e.g., SiFive E2 series).

按程序顺序取指、执行、提交的简单实现方式，面积功耗小，适合嵌入式（如 SiFive E2 系列）。

---

## RISC-V-Specific Microarchitecture Considerations / RISC-V 特有微架构考量

### misaligned access handling / 非对齐访问处理

Base ISA requires natural alignment; misaligned accesses trigger exceptions. Implementations may choose to handle misaligned accesses in hardware (splitting into multiple aligned accesses), or Linux can emulate them via exception handlers.

基础 ISA 要求自然对齐，非对齐触发 exception。实现可选择在硬件中处理非对齐（拆分为多次对齐访问），Linux 会通过异常处理程序模拟。

### FENCE implementation / FENCE 实现

`FENCE` instruction implementation varies: in-order cores can treat it as a nop (naturally ordered), while out-of-order cores implement it as a memory barrier (drain store buffer + order load queue).

`FENCE` 指令的实际实现可以是：顺序核心的 nop（天然顺序），或乱序核心的内存屏障（drain store buffer + order load queue）。

### Zihintpause (pause hint) / 暂停提示

`pause` instruction (encoded as a specific `fence` variant) hints to hardware that the current code is in a spin-wait loop, allowing the processor to reduce power consumption or yield hyperthreading resources, similar to x86's `pause` instruction.

`pause` 指令（编码为特定 `fence` 变体），向硬件提示当前处于自旋等待（spin-wait loop），允许处理器降低功耗或让出超线程资源，类似 x86 的 `pause` 指令。

### Zawrs (Wait on Reservation Set) / 等待保留集

After LR (load-reserved) establishes a reservation, if the reservation set remains unmodified, the hart enters a low-power wait state until the reservation set is written to (waking it up)—more energy-efficient than `wfi` + polling spinlocks.

在 LR（load-reserved）建立保留后，若保留集未被修改则 hart 进入低功耗等待状态，直到保留集被写入唤醒——比 `wfi` + 轮询的自旋锁更节能。

---

## Advanced Pipeline Concepts / 高级流水线概念

### superscalar pipeline / 超标量流水线

Multiple-issue pipeline that can fetch, decode, execute, and retire multiple instructions per cycle. RISC-V's regular encoding simplifies multi-way decode.

每周期可取指、译码、执行、退休多条指令的多发射流水线。RISC-V 的规整编码简化了多路译码。

**Typical configurations / 典型配置：**
- 2-way superscalar: Common in embedded (e.g., SiFive U74) / 2 路超标量：常见于嵌入式（如 SiFive U74）
- 4-way superscalar: Application processors / 4 路超标量：应用处理器
- 6-8 way: High-performance servers / 6-8 路：高性能服务器

### issue width / 发射宽度

Maximum number of instructions that can be dispatched to execution units per cycle. Wider issue requires more decode bandwidth and execution units.

每周期可分派到执行单元的最大指令数。更宽的发射需要更多译码带宽和执行单元。

### retirement width / 退休宽度

Maximum number of instructions that can complete and update architectural state per cycle. Often matches or is narrower than issue width.

每周期可完成并更新架构状态的最大指令数。通常匹配或窄于发射宽度。

---

### dual-issue / triple-issue / 双发射 / 三发射

Describes specific superscalar widths. RISC-V's compressed instructions can enable higher effective issue rates when mixed with 32-bit instructions.

描述特定的超标量宽度。RISC-V 的压缩指令与 32 位指令混合时可实现更高的有效发射率。

---

## Out-of-Order Execution Details / 乱序执行详解

### instruction window / 指令窗口

The pool of instructions simultaneously in-flight in an out-of-order processor, limited by ROB size (typically 64-256 entries for high-performance cores).

乱序处理器中同时在飞行的指令池，受 ROB 大小限制（高性能核心通常 64-256 项）。

### issue queue / 发射队列

Queue holding decoded instructions waiting for operands. When operands are ready, instructions are selected for execution (issue logic).

保存已译码但等待操作数的指令的队列。操作数就绪时，指令被选择执行（发射逻辑）。

### wakeup and select / 唤醒与选择

Two-phase process in OoO cores:
1. **Wakeup**: Broadcasting result tags to waiting instructions / 向等待指令广播结果标签
2. **Select**: Choosing ready instructions for execution units / 为执行单元选择就绪指令

乱序核心的两阶段过程：
1. **唤醒**：向等待指令广播结果标签
2. **选择**：为执行单元选择就绪指令

---

### physical register file / 物理寄存器堆

Large register file (e.g., 128-256 physical registers) used by register renaming to eliminate false dependencies (WAW, WAR).

寄存器重命名使用的大型寄存器堆（如 128-256 个物理寄存器），消除假依赖（WAW、WAR）。

### register allocation table (RAT) / 寄存器分配表

Maps architectural registers (x0-x31) to physical registers. Updated during decode/rename and restored on misspeculation.

将架构寄存器（x0-x31）映射到物理寄存器。在译码/重命名时更新，推测错误时恢复。

### free list / 空闲列表

Tracks available physical registers for allocation. When instructions commit, their old physical registers are returned to the free list.

跟踪可分配的物理寄存器。指令提交时，其旧物理寄存器返回空闲列表。

---

### speculative execution / 推测执行

Executing instructions before knowing if they're on the correct path (after unresolved branches). Requires rollback on misprediction.

在不确定是否在正确路径上（未解决的分支之后）时执行指令。预测错误时需要回滚。

### checkpoint / 检查点

Saved architectural state snapshot that can be restored on misspeculation. Some designs use checkpoints instead of ROB-based recovery.

可在推测错误时恢复的架构状态快照。一些设计使用检查点而非基于 ROB 的恢复。

---

## Execution Units / 执行单元

### ALU (Arithmetic Logic Unit) / 算术逻辑单元

Performs integer arithmetic and logical operations. High-performance cores have multiple ALUs (2-4) to exploit ILP.

执行整数算术和逻辑操作。高性能核心有多个 ALU（2-4）以利用 ILP。

### multiplier / divider / 乘法器 / 除法器

M extension execution units. Designs vary widely:

M 扩展执行单元。设计差异很大：

**Multiplier designs / 乘法器设计：**
- Iterative (32 cycles for RV32): Area-efficient / 迭代式（RV32 需 32 周期）：面积高效
- Pipelined (1-4 cycles): Higher throughput / 流水线式（1-4 周期）：更高吞吐量
- Single-cycle: Highest performance, largest area / 单周期：最高性能，最大面积

**Divider designs / 除法器设计：**
- Iterative (16-64 cycles): Most common / 迭代式（16-64 周期）：最常见
- Pipelined: Rare due to area cost / 流水线式：因面积成本少见

---

### FPU (Floating-Point Unit) / 浮点单元

F/D extension execution units. Typically includes:

F/D 扩展执行单元。通常包括：

- **FADD**: Floating-point adder (3-5 cycle latency) / 浮点加法器（3-5 周期延迟）
- **FMUL**: Floating-point multiplier (3-5 cycles) / 浮点乘法器（3-5 周期）
- **FMA**: Fused multiply-add (4-6 cycles) / 融合乘加（4-6 周期）
- **FDIV**: Floating-point divider (10-20+ cycles, often iterative) / 浮点除法器（10-20+ 周期，通常迭代）
- **FSQRT**: Square root (15-30+ cycles) / 平方根（15-30+ 周期）

**FP register file**: Separate 32-entry f0-f31 register file, 64-bit per entry for D extension.

**FP 寄存器堆**：独立的 32 项 f0-f31 寄存器堆，D 扩展时每项 64 位。

---

### vector unit / 向量单元

V extension execution unit. Complex design with multiple lanes processing elements in parallel.

V 扩展执行单元。具有多个通道并行处理元素的复杂设计。

**Key components / 关键组件：**
- **Vector register file**: 32 vector registers (v0-v31), each VLEN bits / 向量寄存器堆：32 个向量寄存器（v0-v31），每个 VLEN 位
- **Execution lanes**: Multiple parallel lanes (2-16 typical) / 执行通道：多个并行通道（通常 2-16）
- **Lane width**: Bits processed per lane per cycle / 通道宽度：每通道每周期处理的位数
- **Crossbar**: Connects lanes for permutation ops / 交叉开关：连接通道用于置换操作

**Design tradeoffs / 设计权衡：**
- More lanes = higher throughput but more area/power / 更多通道 = 更高吞吐量但更多面积/功耗
- Wider lanes = fewer cycles per operation / 更宽通道 = 每操作更少周期

---

## Load-Store Unit (LSU) / 加载存储单元

### load queue / store queue / 加载队列 / 存储队列

Out-of-order cores use separate queues for loads and stores:

乱序核心对加载和存储使用独立队列：

**Load queue / 加载队列：**
- Holds speculative loads / 保存推测性加载
- Checks for address conflicts with earlier stores / 检查与早期存储的地址冲突
- Can execute out-of-order / 可乱序执行

**Store queue / 存储队列：**
- Buffers stores until commit / 缓冲存储直到提交
- Provides store-to-load forwarding / 提供存储到加载转发
- Drains in program order / 按程序顺序排空

---

### store-to-load forwarding / 存储到加载转发

When a load reads from a recently stored address still in the store queue, the data is forwarded directly without waiting for cache write.

当加载从仍在存储队列中的最近存储地址读取时，数据直接转发，无需等待缓存写入。

### memory disambiguation / 内存消歧

Predicting whether a load address matches earlier store addresses. Conservative approach stalls loads; aggressive approach speculates and squashes on mismatch.

预测加载地址是否与早期存储地址匹配。保守方法停顿加载；激进方法推测并在不匹配时清空。

---

### TLB (Translation Lookaside Buffer) / 快表

Caches virtual-to-physical address translations. Critical for performance with virtual memory.

缓存虚拟到物理地址转换。对虚拟内存性能至关重要。

**Structure / 结构：**
- **Fully associative**: Small TLB (8-16 entries) / 全相联：小型 TLB（8-16 项）
- **Set associative**: Larger TLB (64-512 entries) / 组相联：较大 TLB（64-512 项）
- **Split I-TLB/D-TLB**: Separate for instructions and data / 分离 I-TLB/D-TLB：指令和数据分离

**TLB miss handling / TLB 缺失处理：**
- **Hardware page table walk**: Automatic (common in RISC-V) / 硬件页表遍历：自动（RISC-V 中常见）
- **Software TLB refill**: Trap to software handler (MIPS-style) / 软件 TLB 重填：陷入软件处理程序（MIPS 风格）

---

## Cache Hierarchies / 缓存层次结构

### L1 cache / 一级缓存

Smallest, fastest cache directly attached to the core. Split into I-cache and D-cache.

最小、最快的缓存，直接连接到核心。分为 I-cache 和 D-cache。

**Typical L1 parameters / 典型 L1 参数：**
- **Size**: 16-64 KB per cache / 大小：每个缓存 16-64 KB
- **Associativity**: 2-8 way / 相联度：2-8 路
- **Latency**: 2-4 cycles / 延迟：2-4 周期
- **Line size**: 64 bytes / 行大小：64 字节

---

### L2 cache / 二级缓存

Larger, slower unified cache shared or private per core.

更大、更慢的统一缓存，每核共享或私有。

**Typical L2 parameters / 典型 L2 参数：**
- **Size**: 256 KB - 2 MB / 大小：256 KB - 2 MB
- **Associativity**: 8-16 way / 相联度：8-16 路
- **Latency**: 10-20 cycles / 延迟：10-20 周期
- **Inclusive/Exclusive**: Design choice / 包含/排斥：设计选择

---

### L3 cache / 三级缓存

Last-level cache (LLC) shared across all cores in multi-core systems.

多核系统中所有核心共享的最后一级缓存（LLC）。

**Typical L3 parameters / 典型 L3 参数：**
- **Size**: 4-64 MB / 大小：4-64 MB
- **Associativity**: 16-32 way / 相联度：16-32 路
- **Latency**: 30-60 cycles / 延迟：30-60 周期
- **Usually inclusive**: Contains copies of L1/L2 data / 通常包含：包含 L1/L2 数据的副本

---

**Last Updated / 最后更新：** 2026-07-23
