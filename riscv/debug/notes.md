# RISC-V 调试与跟踪学习笔记

> 文档入口：
> - Debug Spec: https://github.com/riscv/riscv-debug-spec
> - OpenOCD: https://openocd.org/
> - Spike: https://github.com/riscv-software-src/riscv-isa-sim

---

## 调试链路架构

```
GDB (主机)
  ↕ GDB Remote Serial Protocol (RSP, TCP)
OpenOCD (主机)
  ↕ JTAG 信号（TCK/TMS/TDI/TDO）
DTM (Debug Transport Module，芯片内)
  ↕ DMI (Debug Module Interface)
DM (Debug Module，芯片内)
  ↕ 系统总线 / 内部控制信号
Hart（目标处理器）
```

关键认知：GDB 不直接说 RISC-V 语言，它通过标准 RSP 协议与 OpenOCD 通信，OpenOCD 再将 GDB 的"读寄存器/设断点/单步"翻译为对 DM 的 JTAG/DMI 操作。

---

## 硬件断点 vs 软件断点

| 类型 | 实现方式 | 数量限制 | 适用场景 |
|------|---------|---------|---------|
| 软件断点 | 将目标地址指令替换为 `ebreak` | 无限（受内存写权限限制） | ROM 以外的任意地址 |
| 硬件断点 | Trigger 模块 `mcontrol`，`execute=1` | 受 Trigger 数量限制（通常 2–8 个） | ROM、Flash（只读内存） |
| 观察点 | Trigger 模块 `mcontrol`，`load/store=1` | 同上 | 监视内存地址读写 |

调试 ROM 中的代码（如 BootROM）**只能用硬件断点**。

---

## 单步执行原理

RISC-V 没有专用单步标志位。调试器通过两种方式实现单步：

1. **icount Trigger**：配置 Trigger 的 `icount=1, action=进入调试模式`，执行 `dpc`（调试 PC）处一条指令后自动暂停
2. **软件模拟**：在下一条指令地址设软件断点，resume，触发断点后删除——效率低

GDB 的 `si`（stepi）命令内部使用方式 1（若 DM 支持 icount trigger）。

---

## OpenOCD 连接 RISC-V 配置示例

```tcl
# openocd.cfg（JTAG + RISC-V DM 配置）
adapter driver ftdi
ftdi_vid_pid 0x0403 0x6010

transport select jtag
jtag newtap riscv cpu -irlen 5

target create riscv.cpu riscv -chain-position riscv.cpu
riscv.cpu configure -work-area-phys 0x80000000 -work-area-size 0x10000

init
halt
```

```bash
# 启动 OpenOCD
openocd -f openocd.cfg &

# GDB 连接
riscv64-unknown-elf-gdb firmware.elf
(gdb) target extended-remote :3333
(gdb) monitor reset halt
(gdb) load          # 烧写程序
(gdb) break main
(gdb) continue
```

---

## Spike 调试技巧

```bash
# 以交互模式启动 spike（内置调试器）
spike --isa=rv64gcv pk hello

# spike 交互命令
(spike) r          # 运行
(spike) s          # 单步
(spike) until pc 0 0x80001000   # 运行到指定 PC
(spike) reg 0 a0   # 查看 hart 0 的 a0 寄存器
(spike) mem 0 0x80000000 64    # 查看内存内容

# 生成执行跟踪（指令级日志）
spike --log-commits -l --isa=rv64gc pk hello 2> trace.log
```

---

## 待深入阅读

- [ ] RISC-V Debug Spec 全文（Abstract Command 类型详解）
- [ ] OpenOCD RISC-V 支持文档
- [ ] RISC-V Trace Spec（E-Trace 编码格式）
- [ ] riscv-tests 的使用方法（验证处理器实现）
