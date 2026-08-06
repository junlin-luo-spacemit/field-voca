---
tags: [riscv, glossary]
domain: all
last_updated: 2026-07-23
---

# RISC-V Master Glossary

> 跨领域核心术语速查，详细条目见各领域词汇表

---

## A

- **ABI**（Application Binary Interface）：调用约定、数据布局、寄存器使用规范 → [[01-isa基础/vocabulary]]
- **Abstract Command**：DM 向 hart 发送的高级操作请求（Access Register/Quick Access/Access Memory）→ [[05-调试与工具链/debug/vocabulary]]
- **ACPI**：高级配置与电源接口，服务器平台硬件描述标准 → [[05-调试与工具链/platform/vocabulary]]
- **AES**：高级加密标准，Zkne/Zknd 提供硬件加速 → [[03-标准扩展/vocabulary]]
- **AIA**（Advanced Interrupt Architecture）：PLIC 的现代替代，支持 MSI，包含 APLIC + IMSIC → [[05-调试与工具链/platform/vocabulary]]
- **ACLINT**：高级核本地中断器，标准化定时器和软件中断，包含 MTIMER/MSWI/SSWI → [[05-调试与工具链/platform/vocabulary]]
- **ALU**（Arithmetic Logic Unit）：算术逻辑单元，执行整数运算 → [[04-微架构/vocabulary]]
- **AMO**（Atomic Memory Operation）：原子读改写指令（A 扩展）→ [[01-isa基础/vocabulary]]
- **AMO PMA levels**：AMONone/AMOSwap/AMOLogical/AMOArithmetic，原子操作支持级别 → [[02-特权架构/vocabulary]]
- **APLIC**：AIA 中的高级平台级中断控制器，支持 Direct 和 MSI 模式 → [[05-调试与工具链/platform/vocabulary]]
- **ASID**（Address Space Identifier）：TLB 地址空间标识符，用于多进程 TLB 共享 → [[02-特权架构/vocabulary]]
- **AUIPC**：PC 加高位立即数指令，用于 PC 相对寻址 → [[01-isa基础/vocabulary]]
- **AVL**（Application Vector Length）：向量扩展中应用请求的元素数 → [[03-标准扩展/vocabulary]]
- **address-translation cache**：TLB，缓存页表遍历结果，对 hart 私有 → [[02-特权架构/vocabulary]]

## B

- **B Extension**：位操作扩展（Zba/Zbb/Zbc/Zbs）→ [[03-标准扩展/vocabulary]]
- **BOOM**（Berkeley Out-of-Order Machine）：开源乱序 RISC-V 处理器 → [[05-调试与工具链/software/vocabulary]]
- **BHT**（Branch History Table）：分支历史预测表 → [[04-微架构/vocabulary]]
- **BTB**（Branch Target Buffer）：分支目标缓冲 → [[04-微架构/vocabulary]]

## C

- **C Extension**：压缩指令扩展（16 位编码）→ [[01-isa基础/vocabulary]]
- **CPI**（Cycles Per Instruction）：每指令周期数 → [[04-微架构/vocabulary]]
- **CSR**（Control and Status Register）：控制与状态寄存器 → [[02-特权架构/vocabulary]]
- **CVA6**：开源 RISC-V 应用处理器核（原 Ariane）→ [[05-调试与工具链/software/vocabulary]]
- **CoRR**（Coherence for Read-Read）：读-读一致性，RVWMO PPO Rule 2 保证 → [[01-isa基础/vocabulary]]
- **Contained Trap**：受控陷阱，对执行环境内软件可见并由其处理 → [[01-isa基础/vocabulary]]

## D

- **D Extension**：双精度浮点扩展 → [[03-标准扩展/vocabulary]]
- **DM**（Debug Module）：调试模块 → [[05-调试与工具链/debug/vocabulary]]
- **DMI**（Debug Module Interface）：调试模块接口 → [[05-调试与工具链/debug/vocabulary]]
- **DTB**（Device Tree Blob）：设备树二进制 → [[05-调试与工具链/platform/vocabulary]]
- **DTM**（Debug Transport Module）：调试传输模块 → [[05-调试与工具链/debug/vocabulary]]
- **double trap**：双重陷阱，在 trap 处理关键阶段再次发生 trap → [[02-特权架构/vocabulary]]

## E

- **ebreak**：调试断点指令 → [[02-特权架构/vocabulary]]
- **ecall**：环境调用/系统调用指令 → [[02-特权架构/vocabulary]]
- **EEI**（Execution Environment Interface）：执行环境接口，定义程序运行的完整环境 → [[01-isa基础/vocabulary]]
- **ELEN**：向量元素最大位宽 → [[03-标准扩展/vocabulary]]
- **E-Trace**（Efficient Trace）：RISC-V 高效指令跟踪规范 → [[05-调试与工具链/debug/vocabulary]]
- **endianness**：字节序，指令始终小端存储 → [[01-isa基础/vocabulary]]

## F

- **F Extension**：单精度浮点扩展 → [[03-标准扩展/vocabulary]]
- **Fatal Trap**：致命陷阱，导致执行环境终止 → [[01-isa基础/vocabulary]]
- **fcsr**：浮点控制与状态寄存器 → [[03-标准扩展/vocabulary]]
- **FENCE**：内存屏障指令，含 predecessor/successor 位域 → [[01-isa基础/vocabulary]]
- **FENCE.I**：I-cache 同步屏障 → [[01-isa基础/vocabulary]]
- **FS/VS/XS/SD**：mstatus 扩展上下文状态（Off/Initial/Clean/Dirty）→ [[02-特权架构/vocabulary]]
- **FMA**（Fused Multiply-Add）：融合乘加 → [[03-标准扩展/vocabulary]]
- **frozen**：规范冻结状态（高度稳定）→ [[01-isa基础/vocabulary]]

## G

- **G**：通用扩展组合（=I+M+A+F+D+Zicsr+Zifencei）→ [[01-isa基础/vocabulary]]
- **GDB**：GNU 调试器，支持 RISC-V → [[05-调试与工具链/debug/vocabulary]]
- **GMO**（Global Memory Order）：全局内存序，RVWMO 核心概念 → [[01-isa基础/vocabulary]]

## H

- **H Extension**：虚拟化扩展 → [[03-标准扩展/vocabulary]]
- **hart**（Hardware Thread）：硬件线程，基本执行单元 → [[01-isa基础/vocabulary]]
- **hgatp**：Stage-2 地址翻译根页表寄存器 → [[03-标准扩展/vocabulary]]
- **HINT instruction**：HINT 指令，不修改架构状态，向微架构传递提示 → [[01-isa基础/vocabulary]]
- **HiFive Unmatched**：SiFive PC级RISC-V开发板，U740 SoC，16GB DDR4 → [[05-调试与工具链/software/vocabulary]]
- **HSM**（Hart State Management）：hart 生命周期管理 SBI 扩展（EID 0x48534D），启动/停止/挂起hart → [[05-调试与工具链/platform/vocabulary]]
- **hstatus**：Hypervisor状态寄存器，控制虚拟化特性 → [[03-标准扩展/vocabulary]]
- **hvip**：Hypervisor虚拟中断挂起寄存器，注入虚拟中断 → [[03-标准扩展/vocabulary]]

## I

- **I**：基础整数指令集 → [[01-isa基础/vocabulary]]
- **I-cache**：指令缓存，L1级指令取指缓存 → [[04-微架构/vocabulary]]
- **IALIGN**：指令地址对齐约束（16或32位）→ [[01-isa基础/vocabulary]]
- **ID**（Instruction Decode）：指令译码流水线阶段 → [[04-微架构/vocabulary]]
- **ILEN**：实现支持的最大指令长度 → [[01-isa基础/vocabulary]]
- **IMSIC**（Incoming MSI Controller）：AIA中的MSI接收控制器，每hart一个，支持中断文件和虚拟化 → [[05-调试与工具链/platform/vocabulary]]
- **IPC**（Instructions Per Cycle）：每周期指令数 → [[04-微架构/vocabulary]]
- **IPI**（Inter-Processor Interrupt）：处理器间中断SBI扩展（EID 0x735049），发送软件中断 → [[05-调试与工具链/platform/vocabulary]]
- **ISA**（Instruction Set Architecture）：指令集架构 → [[01-isa基础/vocabulary]]
- **I-Type**：立即数指令格式，编码ADDI/LOAD/JALR等，12位立即数 → [[01-isa基础/vocabulary]]
- **icount**：指令计数触发器（单步调试）→ [[05-调试与工具链/debug/vocabulary]]
- **idempotency PMA**：幂等性物理内存属性，描述访问副作用 → [[02-特权架构/vocabulary]]
- **instruction window**：乱序执行的指令窗口，受ROB大小限制（64-256项）→ [[04-微架构/vocabulary]]
- **Invisible Trap**：透明陷阱，执行环境透明处理，软件不感知 → [[01-isa基础/vocabulary]]
- **issue width**：每周期可分派到执行单元的最大指令数 → [[04-微架构/vocabulary]]

## J

- **JAL**：跳转并链接 → [[01-isa基础/vocabulary]]
- **JALR**：寄存器跳转并链接（函数返回）→ [[01-isa基础/vocabulary]]
- **JTAG**：调试接口标准（IEEE 1149.1）→ [[05-调试与工具链/debug/vocabulary]]

## L

- **L1/L2/L3 cache**：一/二/三级缓存层次结构 → [[04-微架构/vocabulary]]
- **LB/LH/LW/LD**：加载字节/半字/字/双字指令，符号扩展 → [[01-isa基础/vocabulary]]
- **LBU/LHU/LWU**：无符号加载指令，零扩展 → [[01-isa基础/vocabulary]]
- **little-endian**：小端字节序，RISC-V存储格式，LSB在最低地址 → [[01-isa基础/vocabulary]]
- **LMUL**（Length Multiplier）：向量寄存器组倍增系数（1/8, 1/4, 1/2, 1, 2, 4, 8）→ [[03-标准扩展/vocabulary]]
- **load queue**：加载队列，乱序核心中保存推测性加载 → [[04-微架构/vocabulary]]
- **load-store architecture**：加载-存储架构，内存操作仅通过Load/Store → [[01-isa基础/vocabulary]]
- **lp64d**：64位Linux主流ABI，双精度浮点通过f寄存器传递 → [[01-isa基础/vocabulary]]
- **LR/SC**（Load-Reserved/Store-Conditional）：原子操作原语 → [[01-isa基础/vocabulary]]
- **LSU**（Load-Store Unit）：加载存储单元，执行内存访问 → [[04-微架构/vocabulary]]
- **LUI**：加载高位立即数指令，设置高20位 → [[01-isa基础/vocabulary]]
- **load value axiom**：加载值公理，RVWMO 三大公理之一 → [[01-isa基础/vocabulary]]

## M

- **M Extension**：整数乘除法扩展 → [[03-标准扩展/vocabulary]]
- **M-mode**（Machine mode）：最高特权级 → [[02-特权架构/vocabulary]]
- **march**：GCC 目标指令集选项 → [[05-调试与工具链/software/vocabulary]]
- **mcontrol**：内存访问触发器类型 → [[05-调试与工具链/debug/vocabulary]]
- **mcause**：trap 原因寄存器 → [[02-特权架构/vocabulary]]
- **MDT**（M-mode Disable Trap）：双重陷阱检测位，进入 M-mode trap 时自动置 1 → [[02-特权架构/vocabulary]]
- **mepc**：异常程序计数器 → [[02-特权架构/vocabulary]]
- **menvcfg**：机器环境配置寄存器（FIOM/CBIE/CBZE/STCE 等）→ [[02-特权架构/vocabulary]]
- **mhartid**：hart ID 寄存器 → [[02-特权架构/vocabulary]]
- **misa**：机器 ISA 寄存器，指示支持的扩展集合 → [[02-特权架构/vocabulary]]
- **MPRV**（Modify PRiVilege）：mstatus 字段，令 Load/Store 以 MPP 模式翻译地址 → [[02-特权架构/vocabulary]]
- **mret**：M 模式 trap 返回指令 → [[02-特权架构/vocabulary]]
- **MSI**（Message Signaled Interrupt）：消息信号中断 → [[05-调试与工具链/platform/vocabulary]]
- **mseccfg**：机器安全配置 CSR（SSEED/USEED/RLB/MMWP/MML/MLPE）→ [[02-特权架构/vocabulary]]
- **mstatus**：M 模式状态寄存器 → [[02-特权架构/vocabulary]]
- **mtime / mtimecmp**：实时时钟寄存器（MMIO），定时器中断来源 → [[02-特权架构/vocabulary]]
- **mtval**：trap 附加信息寄存器（缺页地址/非法指令编码）→ [[02-特权架构/vocabulary]]
- **mtvec**：trap 向量基地址寄存器（Direct / Vectored 模式）→ [[02-特权架构/vocabulary]]
- **MXR**（Make eXecutable Readable）：mstatus 字段，允许加载只执行页 → [[02-特权架构/vocabulary]]

## N

- **NMI**（Non-Maskable Interrupt）：不可屏蔽中断，用于硬件错误，不重置处理器状态 → [[02-特权架构/vocabulary]]

## O

- **OoO**（Out-of-Order）：乱序执行，按操作数就绪情况动态调度 → [[04-微架构/vocabulary]]
- **OpenOCD**：开源片上调试器，通过JTAG连接DM，为GDB提供远程调试服务 → [[05-调试与工具链/debug/vocabulary]]
- **OpenSBI**：RISC-V参考SBI固件实现，M-mode运行，提供15个SBI扩展 → [[05-调试与工具链/platform/vocabulary]]
- **Opcode**：操作码，指令的bits[6:0]，标识指令类型 → [[01-isa基础/vocabulary]]
- **atomicity axiom**：原子性公理，RVWMO 三大公理之一（LR/SC 之间无外部干涉）→ [[01-isa基础/vocabulary]]

## P

- **PC**（Program Counter）：程序计数器，保存当前指令地址 → [[01-isa基础/vocabulary]]
- **page table walk**：页表遍历，9步算法将虚拟地址翻译为物理地址 → [[02-特权架构/vocabulary]]
- **physical register file**：物理寄存器堆，用于寄存器重命名（128-256个寄存器）→ [[04-微架构/vocabulary]]
- **pk**（Proxy Kernel）：Spike 代理内核 → [[05-调试与工具链/software/vocabulary]]
- **PLIC**（Platform Level Interrupt Controller）：平台级中断控制器，Gateway/Priority/Context架构 → [[05-调试与工具链/platform/vocabulary]]
- **PMA**（Physical Memory Attribute）：物理内存属性 → [[02-特权架构/vocabulary]]
- **PMU**（Performance Monitoring Unit）：性能监控单元SBI扩展（EID 0x504D55），配置硬件性能计数器 → [[05-调试与工具链/platform/vocabulary]]
- **PMP**（Physical Memory Protection）：物理内存保护 → [[02-特权架构/vocabulary]]
- **PPO**（Preserved Program Order）：保留程序序，RVWMO核心概念，13条规则 → [[01-isa基础/vocabulary]]
- **Program Buffer**：程序缓冲区，DM中供调试器写入指令让hart执行（2-16条指令）→ [[05-调试与工具链/debug/vocabulary]]
- **profile**：RISC-V平台配置文件（RVA22/RVA23）→ [[01-isa基础/vocabulary]]
- **progress axiom**：前向进展公理，RVWMO三大公理之一 → [[01-isa基础/vocabulary]]
- **pseudo-instruction**：伪指令，汇编器便利功能（nop/li/mv/ret）→ [[01-isa基础/vocabulary]]
- **psABI**：处理器特定 ABI 规范 → [[05-调试与工具链/software/vocabulary]]
- **PTE**（Page Table Entry）：页表项（V/R/W/X/U/G/A/D/PPN）→ [[02-特权架构/vocabulary]]

## Q

- **QEMU**：开源处理器模拟器 → [[05-调试与工具链/software/vocabulary]]

## R

- **ra**（return address）：返回地址寄存器（x1），保存函数返回地址 → [[01-isa基础/vocabulary]]
- **RAS**（Return Address Stack）：返回地址预测栈，预测ret指令目标（8-16项）→ [[04-微架构/vocabulary]]
- **RAS hint**：JAL/JALR 通过寄存器号隐式编码 Push/Pop 操作提示 → [[01-isa基础/vocabulary]]
- **RAT**（Register Allocation Table）：寄存器分配表，将架构寄存器映射到物理寄存器 → [[04-微架构/vocabulary]]
- **ratified**：已批准规范状态 → [[01-isa基础/vocabulary]]
- **RCpc / RCsc**：Release Consistency Processor/Sequentially Consistent，RVWMO 获取释放语义强度 → [[01-isa基础/vocabulary]]
- **reset vector**：复位后 PC 初始值，由平台规范定义 → [[02-特权架构/vocabulary]]
- **retirement width**：每周期可退休的最大指令数 → [[04-微架构/vocabulary]]
- **RFNC**（Remote Fence）：SBI 远程 TLB 刷新扩展（EID 0x52464E43），远程 SFENCE.VMA → [[05-调试与工具链/platform/vocabulary]]
- **RISC-V**：开放标准指令集架构 → [[01-isa基础/vocabulary]]
- **ROB**（Reorder Buffer）：重排序缓冲区，跟踪所有in-flight指令，确保顺序提交（64-256项）→ [[04-微架构/vocabulary]]
- **Rocket Chip**：UC Berkeley 开源 RISC-V 生成器 → [[05-调试与工具链/software/vocabulary]]
- **RsrvEventual / RsrvNonEventual**：LR/SC 保留集 PMA，是否保证最终成功 → [[02-特权架构/vocabulary]]
- **R-Type**：寄存器-寄存器指令格式，编码ADD/SUB/AND/OR/XOR/SLL/SRL/SRA → [[01-isa基础/vocabulary]]
- **RV32E/RV32I/RV64I**：RISC-V 基础 ISA 变体 → [[01-isa基础/vocabulary]]
- **RVWMO**：RISC-V 弱内存序，含 GMO/PPO/三大公理 → [[01-isa基础/vocabulary]]

## S

- **satp**：监管者地址翻译与保护寄存器（MODE/ASID/PPN）→ [[02-特权架构/vocabulary]]
- **SB/SH/SW/SD**：存储字节/半字/字/双字指令 → [[01-isa基础/vocabulary]]
- **SBI**（Supervisor Binary Interface）：监管者二进制接口，15个扩展 → [[05-调试与工具链/platform/vocabulary]]
- **semihosting**：调试器通过特定 ebreak 序列提供服务的机制 → [[01-isa基础/vocabulary]]
- **SFENCE.VMA**：TLB 刷新指令，四种 rs1/rs2 组合语义 → [[02-特权架构/vocabulary]]
- **S-mode**（Supervisor mode）：OS 内核特权级 → [[02-特权架构/vocabulary]]
- **sp**（stack pointer）：栈指针寄存器（x2）→ [[01-isa基础/vocabulary]]
- **spike**：RISC-V 官方参考模拟器 → [[05-调试与工具链/software/vocabulary]]
- **sret**：S 模式 trap 返回 → [[02-特权架构/vocabulary]]
- **SRST**（System Reset）：系统复位SBI扩展（EID 0x53525354），执行系统重启/关机 → [[05-调试与工具链/platform/vocabulary]]
- **store buffer forwarding**：store buffer 转发，load value axiom 的第2选项来源 → [[01-isa基础/vocabulary]]
- **store queue**：存储队列，乱序核心缓冲存储直到提交，提供转发 → [[04-微架构/vocabulary]]
- **S-Type**：存储指令格式，编码SB/SH/SW，12位偏移量分两个字段 → [[01-isa基础/vocabulary]]
- **SUM**（permit Supervisor User Memory）：mstatus 字段，控制 S 模式访问 U 页面 → [[02-特权架构/vocabulary]]
- **superpage**：超大页，任意级别叶 PTE 均可（Sv39：2MiB/1GiB）→ [[02-特权架构/vocabulary]]
- **superscalar**：超标量流水线，每周期可发射/执行多条指令（2/4/6-8路）→ [[04-微架构/vocabulary]]
- **SUSP**（System Suspend）：系统挂起SBI扩展（EID 0x53555350），低功耗挂起模式 → [[05-调试与工具链/platform/vocabulary]]
- **Sv39/Sv48/Sv57**：虚拟地址翻译方案（3/4/5级页表）→ [[02-特权架构/vocabulary]]
- **Svade / Svadu**：A/D 位软件异常 vs 硬件更新扩展 → [[02-特权架构/vocabulary]]
- **Svnapot**：自然对齐幂次大页扩展（PTE bit 63 N位）→ [[02-特权架构/vocabulary]]
- **Svpbmt**：页表内存类型扩展（PTE bit 62-61 PBMT，PMA/NC/IO）→ [[02-特权架构/vocabulary]]
- **syntactic dependency**：句法依赖（地址/数据/控制），RVWMO PPO Rules 9-11 → [[01-isa基础/vocabulary]]

## T

- **tdata1/tdata2/tdata3**：触发器数据CSR，配置硬件断点/观察点 → [[05-调试与工具链/debug/vocabulary]]
- **TIME**（Timer）：定时器SBI扩展（EID 0x54494D45），设置定时器中断 → [[05-调试与工具链/platform/vocabulary]]
- **TLB**（Translation Lookaside Buffer）：快表，缓存虚拟到物理地址翻译 → [[02-特权架构/vocabulary]]
- **trap**：异常与中断的统称 → [[02-特权架构/vocabulary]]
- **Trigger Module**：硬件断点/观察点模块，通过tselect/tdata1/tdata2/tdata3配置 → [[05-调试与工具链/debug/vocabulary]]
- **tselect**：触发器选择CSR，选择当前配置的触发器编号 → [[05-调试与工具链/debug/vocabulary]]
- **TSR / TVM / TW**：mstatus 虚拟化陷阱控制字段（Trap SRet/VirtualMemory/Wait）→ [[02-特权架构/vocabulary]]

## U

- **U-Boot**：嵌入式平台开源 Bootloader，支持RISC-V → [[05-调试与工具链/platform/vocabulary]]
- **UEFI**：统一可扩展固件接口，服务器启动标准 → [[05-调试与工具链/platform/vocabulary]]
- **U-mode**（User mode）：用户特权级，最低特权，应用程序运行 → [[02-特权架构/vocabulary]]
- **UNSPECIFIED**：规范术语，故意不限制的行为或值 → [[01-isa基础/vocabulary]]
- **U-Type**：高位立即数指令格式，编码LUI/AUIPC，20位立即数放在高20位 → [[01-isa基础/vocabulary]]

## V

- **V Extension**：向量扩展 → [[03-标准扩展/vocabulary]]
- **Verilator**：RTL 周期精确仿真工具 → [[05-调试与工具链/software/vocabulary]]
- **vl**（Vector Length）：向量操作元素数 → [[03-标准扩展/vocabulary]]
- **VLEN**：向量寄存器位宽 → [[03-标准扩展/vocabulary]]
- **VMID**（Virtual Machine Identifier）：虚拟机标识符 → [[03-标准扩展/vocabulary]]
- **vsetvli**：向量配置指令 → [[03-标准扩展/vocabulary]]
- **vtype**：向量类型寄存器 → [[03-标准扩展/vocabulary]]

## W

- **WFI**（Wait For Interrupt）：等待中断（低功耗），合法实现可作 NOP → [[02-特权架构/vocabulary]]
- **word / halfword / doubleword**：RISC-V 内存单位（32/16/64位，与 XLEN 无关）→ [[01-isa基础/vocabulary]]

## X

- **XLEN**：整数寄存器位宽 → [[01-isa基础/vocabulary]]
- **x0–x31**：通用整数寄存器 → [[01-isa基础/vocabulary]]

## Z

- **Z-extension**：Z 前缀扩展，标准化ISA扩展 → [[01-isa基础/vocabulary]]
- **Zawrs**：等待保留集扩展，wrs.nto/wrs.sto低功耗等待 → [[04-微架构/vocabulary]]
- **Zba/Zbb/Zbc/Zbs**：B 扩展子集，位操作指令 → [[03-标准扩展/vocabulary]]
- **Zicbom / Zicboz / Zicbop**：缓存块管理/清零/预取扩展，cbo.*指令 → [[03-标准扩展/vocabulary]]
- **Zicfiss**（Shadow Stack）：影子栈CFI扩展，防护返回地址，防御ROP攻击 → [[03-标准扩展/vocabulary]]
- **Zicfilp**（Landing Pad）：着陆垫CFI扩展，lpad指令，防御JOP攻击 → [[03-标准扩展/vocabulary]]
- **Zicond**：条件操作扩展，czero.eqz/czero.nez → [[03-标准扩展/vocabulary]]
- **Zicsr**：CSR 访问指令扩展，csrrw/csrrs/csrrc → [[03-标准扩展/vocabulary]]
- **Zifencei**：FENCE.I 扩展，I-cache同步 → [[03-标准扩展/vocabulary]]
- **Zihintpause**：pause 暂停提示扩展，自旋等待优化 → [[04-微架构/vocabulary]]
- **Zfinx/Zdinx**：浮点使用整数寄存器，节省寄存器地址空间 → [[03-标准扩展/vocabulary]]
- **Zkn / Zks / Zkr / Zkt**：密码学扩展族（NIST/国密/熙源/数据无关延迟）→ [[03-标准扩展/vocabulary]]
- **Zkne/Zknd**：AES加密/解密扩展，aes32*/aes64*指令 → [[03-标准扩展/vocabulary]]
- **Zknh**：SHA-2哈希扩展，sha256*/sha512*指令 → [[03-标准扩展/vocabulary]]
- **Ztso**：全存储序扩展，将 RVWMO 提升为 TSO → [[03-标准扩展/vocabulary]]

# RISC-V Master Glossary

> 跨领域核心术语速查，详细条目见各领域词汇表

---

## A

- **ABI**（Application Binary Interface）：调用约定、数据布局、寄存器使用规范 → [[01-isa基础/vocabulary]]
- **ACPI**：高级配置与电源接口，服务器平台硬件描述标准 → [[05-调试与工具链/platform/vocabulary]]
- **AIA**（Advanced Interrupt Architecture）：PLIC 的现代替代，支持 MSI → [[05-调试与工具链/platform/vocabulary]]
- **ACLINT**：高级核本地中断器，标准化定时器和软件中断 → [[05-调试与工具链/platform/vocabulary]]
- **AMO**（Atomic Memory Operation）：原子读改写指令 → [[01-isa基础/vocabulary]]
- **APLIC**：AIA 中的高级 PLIC 组件 → [[05-调试与工具链/platform/vocabulary]]
- **ASID**（Address Space Identifier）：TLB 地址空间标识符 → [[02-特权架构/vocabulary]]
- **AVL**（Application Vector Length）：向量扩展中应用请求的元素数 → [[03-标准扩展/vocabulary]]
- **AUIPC**：PC 加高位立即数指令 → [[01-isa基础/vocabulary]]

## B

- **B Extension**：位操作扩展（Zba/Zbb/Zbc/Zbs）→ [[03-标准扩展/vocabulary]]
- **BOOM**（Berkeley Out-of-Order Machine）：开源乱序 RISC-V 处理器 → [[05-调试与工具链/software/vocabulary]]
- **BHT**（Branch History Table）：分支历史预测表 → [[04-微架构/vocabulary]]
- **BTB**（Branch Target Buffer）：分支目标缓冲 → [[04-微架构/vocabulary]]

## C

- **C Extension**：压缩指令扩展（16 位编码）→ [[01-isa基础/vocabulary]]
- **CPI**（Cycles Per Instruction）：每指令周期数 → [[04-微架构/vocabulary]]
- **CSR**（Control and Status Register）：控制与状态寄存器 → [[02-特权架构/vocabulary]]
- **CVA6**：开源 RISC-V 应用处理器核（原 Ariane）→ [[05-调试与工具链/software/vocabulary]]

## D

- **D Extension**：双精度浮点扩展 → [[03-标准扩展/vocabulary]]
- **DM**（Debug Module）：调试模块 → [[05-调试与工具链/debug/vocabulary]]
- **DMI**（Debug Module Interface）：调试模块接口 → [[05-调试与工具链/debug/vocabulary]]
- **DTB**（Device Tree Blob）：设备树二进制 → [[05-调试与工具链/platform/vocabulary]]
- **DTM**（Debug Transport Module）：调试传输模块 → [[05-调试与工具链/debug/vocabulary]]

## E

- **ebreak**：调试断点指令 → [[02-特权架构/vocabulary]]
- **ecall**：环境调用/系统调用指令 → [[02-特权架构/vocabulary]]
- **ELEN**：向量元素最大位宽 → [[03-标准扩展/vocabulary]]
- **E-Trace**（Efficient Trace）：RISC-V 高效指令跟踪规范 → [[05-调试与工具链/debug/vocabulary]]

## F

- **F Extension**：单精度浮点扩展 → [[03-标准扩展/vocabulary]]
- **fcsr**：浮点控制与状态寄存器 → [[03-标准扩展/vocabulary]]
- **FENCE**：内存屏障指令 → [[01-isa基础/vocabulary]]
- **FENCE.I**：I-cache 同步屏障 → [[01-isa基础/vocabulary]]
- **FMA**（Fused Multiply-Add）：融合乘加 → [[03-标准扩展/vocabulary]]
- **frozen**：规范冻结状态（高度稳定）→ [[01-isa基础/vocabulary]]

## G

- **G**：通用扩展组合（=I+M+A+F+D+Zicsr+Zifencei）→ [[01-isa基础/vocabulary]]
- **GDB**：GNU 调试器，支持 RISC-V → [[05-调试与工具链/debug/vocabulary]]

## H

- **H Extension**：虚拟化扩展 → [[03-标准扩展/vocabulary]]
- **hart**（Hardware Thread）：硬件线程，基本执行单元 → [[01-isa基础/vocabulary]]
- **hgatp**：Stage-2 地址翻译根页表寄存器 → [[03-标准扩展/vocabulary]]
- **HSM**（Hart State Management）：hart 生命周期管理 SBI 扩展 → [[05-调试与工具链/platform/vocabulary]]

## I

- **I**：基础整数指令集 → [[01-isa基础/vocabulary]]
- **icount**：指令计数触发器（单步调试）→ [[05-调试与工具链/debug/vocabulary]]
- **IMSIC**（Incoming MSI Controller）：MSI 接收器 → [[05-调试与工具链/platform/vocabulary]]
- **IPC**（Instructions Per Cycle）：每周期指令数 → [[04-微架构/vocabulary]]
- **ISA**（Instruction Set Architecture）：指令集架构 → [[01-isa基础/vocabulary]]

## J

- **JAL**：跳转并链接 → [[01-isa基础/vocabulary]]
- **JALR**：寄存器跳转并链接（函数返回）→ [[01-isa基础/vocabulary]]
- **JTAG**：调试接口标准（IEEE 1149.1）→ [[05-调试与工具链/debug/vocabulary]]

## L

- **LMUL**（Length Multiplier）：向量寄存器组倍增系数 → [[03-标准扩展/vocabulary]]
- **lp64d**：64位 Linux 主流 ABI → [[01-isa基础/vocabulary]]
- **LR/SC**（Load-Reserved/Store-Conditional）：原子操作原语 → [[01-isa基础/vocabulary]]
- **LUI**：加载高位立即数 → [[01-isa基础/vocabulary]]

## M

- **M Extension**：整数乘除法扩展 → [[03-标准扩展/vocabulary]]
- **M-mode**（Machine mode）：最高特权级 → [[02-特权架构/vocabulary]]
- **march**：GCC 目标指令集选项 → [[05-调试与工具链/software/vocabulary]]
- **mcontrol**：内存访问触发器类型 → [[05-调试与工具链/debug/vocabulary]]
- **mcause**：trap 原因寄存器 → [[02-特权架构/vocabulary]]
- **mepc**：异常程序计数器 → [[02-特权架构/vocabulary]]
- **mhartid**：hart ID 寄存器 → [[02-特权架构/vocabulary]]
- **mret**：M 模式 trap 返回指令 → [[02-特权架构/vocabulary]]
- **MSI**（Message Signaled Interrupt）：消息信号中断 → [[05-调试与工具链/platform/vocabulary]]
- **mstatus**：M 模式状态寄存器 → [[02-特权架构/vocabulary]]
- **mtval**：trap 附加信息寄存器 → [[02-特权架构/vocabulary]]
- **mtvec**：trap 向量基地址寄存器 → [[02-特权架构/vocabulary]]

## O

- **OoO**（Out-of-Order）：乱序执行 → [[04-微架构/vocabulary]]
- **OpenOCD**：开源片上调试器 → [[05-调试与工具链/debug/vocabulary]]
- **OpenSBI**：RISC-V 参考 SBI 固件 → [[05-调试与工具链/platform/vocabulary]]

## P

- **pc**（Program Counter）：程序计数器 → [[01-isa基础/vocabulary]]
- **pk**（Proxy Kernel）：Spike 代理内核 → [[05-调试与工具链/software/vocabulary]]
- **PLIC**（Platform Level Interrupt Controller）：平台级中断控制器 → [[05-调试与工具链/platform/vocabulary]]
- **PMP**（Physical Memory Protection）：物理内存保护 → [[02-特权架构/vocabulary]]
- **profile**：RISC-V 平台配置文件（RVA22/RVA23）→ [[01-isa基础/vocabulary]]
- **psABI**：处理器特定 ABI 规范 → [[05-调试与工具链/software/vocabulary]]
- **PTE**（Page Table Entry）：页表项 → [[02-特权架构/vocabulary]]

## Q

- **QEMU**：开源处理器模拟器 → [[05-调试与工具链/software/vocabulary]]

## R

- **ratified**：已批准规范状态 → [[01-isa基础/vocabulary]]
- **RAS**（Return Address Stack）：返回地址预测栈 → [[04-微架构/vocabulary]]
- **RFNC**（Remote Fence）：SBI 远程 TLB 刷新扩展 → [[05-调试与工具链/platform/vocabulary]]
- **RISC-V**：开放标准指令集架构 → [[01-isa基础/vocabulary]]
- **ROB**（Reorder Buffer）：重排序缓冲区 → [[04-微架构/vocabulary]]
- **Rocket Chip**：UC Berkeley 开源 RISC-V 生成器 → [[05-调试与工具链/software/vocabulary]]
- **RV32E/RV32I/RV64I**：RISC-V 基础 ISA 变体 → [[01-isa基础/vocabulary]]
- **RVWMO**：RISC-V 弱内存序 → [[01-isa基础/vocabulary]]

## S

- **satp**：监管者地址翻译与保护寄存器 → [[02-特权架构/vocabulary]]
- **SBI**（Supervisor Binary Interface）：监管者二进制接口 → [[05-调试与工具链/platform/vocabulary]]
- **S-mode**（Supervisor mode）：OS 内核特权级 → [[02-特权架构/vocabulary]]
- **SFENCE.VMA**：TLB 刷新指令 → [[02-特权架构/vocabulary]]
- **spike**：RISC-V 官方参考模拟器 → [[05-调试与工具链/software/vocabulary]]
- **sret**：S 模式 trap 返回 → [[02-特权架构/vocabulary]]
- **Sv39/Sv48/Sv57**：虚拟地址翻译方案 → [[02-特权架构/vocabulary]]

## T

- **TLB**（Translation Lookaside Buffer）：快表 → [[02-特权架构/vocabulary]]
- **trap**：异常与中断的统称 → [[02-特权架构/vocabulary]]
- **Trigger Module**：硬件断点/观察点模块 → [[05-调试与工具链/debug/vocabulary]]

## U

- **U-Boot**：嵌入式平台开源 Bootloader → [[05-调试与工具链/platform/vocabulary]]
- **UEFI**：统一可扩展固件接口 → [[05-调试与工具链/platform/vocabulary]]
- **U-mode**（User mode）：用户特权级 → [[02-特权架构/vocabulary]]

## V

- **V Extension**：向量扩展 → [[03-标准扩展/vocabulary]]
- **Verilator**：RTL 周期精确仿真工具 → [[05-调试与工具链/software/vocabulary]]
- **vl**（Vector Length）：向量操作元素数 → [[03-标准扩展/vocabulary]]
- **VLEN**：向量寄存器位宽 → [[03-标准扩展/vocabulary]]
- **VMID**（Virtual Machine Identifier）：虚拟机标识符 → [[03-标准扩展/vocabulary]]
- **vsetvli**：向量配置指令 → [[03-标准扩展/vocabulary]]
- **vtype**：向量类型寄存器 → [[03-标准扩展/vocabulary]]

## W

- **WFI**（Wait For Interrupt）：等待中断（低功耗）→ [[02-特权架构/vocabulary]]

## X

- **XLEN**：整数寄存器位宽 → [[01-isa基础/vocabulary]]
- **x0–x31**：通用整数寄存器 → [[01-isa基础/vocabulary]]

## Z

- **Z-extension**：Z 前缀扩展 → [[01-isa基础/vocabulary]]
- **Zawrs**：等待保留集扩展 → [[04-微架构/vocabulary]]
- **Zba/Zbb/Zbc/Zbs**：B 扩展子集 → [[03-标准扩展/vocabulary]]
- **Zicond**：条件操作扩展 → [[03-标准扩展/vocabulary]]
- **Zicsr**：CSR 访问指令扩展 → [[03-标准扩展/vocabulary]]
- **Zifencei**：FENCE.I 扩展 → [[03-标准扩展/vocabulary]]
- **Zihintpause**：pause 暂停提示扩展 → [[04-微架构/vocabulary]]
- **Zfinx/Zdinx**：浮点使用整数寄存器 → [[03-标准扩展/vocabulary]]
