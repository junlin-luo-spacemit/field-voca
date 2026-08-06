# RISC-V 软件生态学习笔记

> 文档入口：
> - GNU Toolchain: https://github.com/riscv-collab/riscv-gnu-toolchain
> - LLVM: https://github.com/llvm/llvm-project
> - Linux: https://github.com/torvalds/linux (arch/riscv/)

---

## 工具链选型快速决策

```
目标场景？
├── 裸机/MCU（无 OS）→ riscv64-unknown-elf-gcc (newlib)
│     -march=rv32imac -mabi=ilp32
├── Linux 用户态编译 → riscv64-linux-gnu-gcc (glibc)
│     -march=rv64gc -mabi=lp64d  ← 标准 Linux ABI
├── 交叉编译 Linux 内核 → riscv64-linux-gnu-gcc
└── 需要向量/最新扩展 → LLVM/Clang (RISC-V 新扩展支持更快)
```

---

## march 字符串构建规则

`-march` 接受扩展名的串联，Z 扩展用下划线分隔：

```bash
-march=rv64gc                    # = rv64i + m + a + f + d + zicsr + zifencei + c
-march=rv64gcv                   # + 向量扩展
-march=rv64gc_zba_zbb_zbc_zbs    # + B 扩展全部子集
-march=rv64gc_zicond             # + 条件操作
-march=rv64imafdc                # 显式列出（等价于 rv64gc）
```

`-mtune` 不影响生成指令集，只影响调度（如流水线延迟、指令顺序）。

---

## QEMU 常用调试命令

```bash
# 全系统仿真，启动 Linux，GDB 服务端监听 1234
qemu-system-riscv64 \
  -machine virt -nographic \
  -bios opensbi-riscv64-generic-fw_dynamic.bin \
  -kernel Image -append "root=/dev/vda console=ttyS0" \
  -drive file=rootfs.img,format=raw,id=hd0 \
  -device virtio-blk-device,drive=hd0 \
  -s -S    # -s = GDB 服务端:1234，-S = 启动时暂停

# 连接 GDB
riscv64-linux-gnu-gdb vmlinux
(gdb) target remote :1234
(gdb) continue

# 用户态仿真（单个 ELF）
qemu-riscv64 -cpu rv64,v=true,vlen=256 ./hello_riscv
```

---

## Linux RISC-V 内核启动关键路径

```
_start (arch/riscv/kernel/head.S)
  ↓ 关中断，设置 sp，清 BSS
setup_vm_final()
  ↓ 启用 Sv39 虚拟内存（写 satp）
start_kernel() (init/main.c)
  ↓ 通用内核初始化
  ↓ of_flat_dt_probe() 解析 DTB
  ↓ 初始化驱动、调度器、文件系统
```

RISC-V Linux 特有：
- `arch/riscv/kernel/sbi.c`：SBI 调用封装
- `arch/riscv/mm/`：Sv39/Sv48 页表管理
- `arch/riscv/kernel/traps.c`：异常/中断处理

---

## 典型开发板与 SoC

| 开发板/SoC | 核心 | 用途 |
|------------|------|------|
| SiFive HiFive Unleashed | SiFive U54 (RV64GC) | 第一块跑 Linux 的 RISC-V 开发板 |
| StarFive VisionFive 2 | T-Head C910 (RV64GCV) | 低成本 Linux SBC，带 GPU |
| Milk-V Pioneer | SG2042 64核 (RV64GCV) | 服务器级 RISC-V |
| Sipeed Longan Nano | GD32VF103 (RV32IMAC) | 嵌入式 MCU，类 STM32 |
| WCH CH32V003 | QingKe V2A (RV32EC) | 极低成本 MCU（< ¥1） |
| ESP32-C3/C6 | Espressif (RV32IMC) | WiFi/BT MCU |

---

## 待深入阅读

- [ ] psABI 文档（栈帧、TLS、PLT/GOT 完整规范）
- [ ] Linux arch/riscv/ 源码导读
- [ ] OpenSBI 源码（M 模式固件实现）
- [ ] GCC RISC-V 后端实现（gcc/config/riscv/）
- [ ] RISC-V Vector Intrinsic API 文档
