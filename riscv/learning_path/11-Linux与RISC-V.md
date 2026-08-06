# 11 Linux 与 RISC-V

## 一句话先记住

> RISC-V 定义 CPU 能执行什么指令、遵守什么特权规则；Linux 是运行在这套规则之上的操作系统；SBI 是两者之间的粘合层。

---

## 第一步：看懂这张图

```
┌─────────────────────────────┐
│  用户程序  (U-mode)          │  ← 普通应用，权限最低
│  浏览器、Python 脚本…        │
└──────────┬──────────────────┘
           │ ecall（系统调用）
           ▼
┌─────────────────────────────┐
│  Linux 内核  (S-mode)        │  ← 你平时说的"跑 Linux"
│  进程管理、内存、文件系统…   │
└──────────┬──────────────────┘
           │ ecall to SBI（请求固件服务）
           ▼
┌─────────────────────────────┐
│  OpenSBI  (M-mode)           │  ← 固件，权限最高，直接管硬件
│  timer、hart 启停、早期串口…  │
└──────────┬──────────────────┘
           │
           ▼
    RISC-V 物理芯片
```

**生活类比**：
- Linux 内核像酒店管理公司，负责给每个住客（进程）分配房间、处理入住退房；
- OpenSBI 像物业总公司，掌握总电闸和门禁钥匙，直接管着硬件；
- 管理公司要动配电室，必须打电话给物业，不能自己动手——这就是 Linux 通过 SBI 请求 M-mode 服务的原因。

**三层职责一句话概括**：

| 层 | 谁在这里 | 能做什么 |
|---|---|---|
| U-mode | 用户程序 | 只能做普通计算，碰不了硬件 |
| S-mode | Linux 内核 | 管内存、进程调度，可以读写大部分 CSR |
| M-mode | OpenSBI 固件 | 控制一切硬件，权限最高 |

> **Reference**: RISC-V Privileged Architecture Spec §3 (Machine ISA), §4 (Supervisor ISA)
> — https://riscv.org/specifications/ratified/

---

## 第二步：Linux 具体用了 RISC-V 的哪些能力

Linux 不是凭空运行的，它依赖 RISC-V 提供的具体硬件机制。把上面的图拆开看：

| Linux 需要做的事 | 用到的 RISC-V 机制 | 怎么用 |
|---|---|---|
| 隔离用户程序与内核 | S-mode / U-mode 特权级 | 内核在 S-mode，程序在 U-mode，越权会触发 trap |
| 给每个进程独立地址空间 | Sv39 页表 + `satp` CSR | 写 `satp` 开启页表，CPU 自动做地址翻译 |
| 捕获异常和中断 | `stvec` CSR + trap 机制 | 内核把异常处理函数地址写入 `stvec` |
| 接收用户程序的请求 | `ecall` 指令 | 程序执行 `ecall`，CPU 陷入 S-mode，内核处理 |
| 设置定时器、启停其他 CPU 核 | SBI 接口（再 `ecall` 进 M-mode） | S-mode 不能直碰硬件，只能通过 SBI 请求 M-mode 代劳 |

> **Reference**: Linux 内核 `arch/riscv/` 目录
> — https://github.com/torvalds/linux/tree/master/arch/riscv

---

## 第三步：SBI 到底解决什么问题

**问题**：Linux 跑在 S-mode，但有些操作（比如设置 timer、关闭某个 CPU 核）只有 M-mode 才有权限做。Linux 怎么办？

**答案**：调用 SBI（Supervisor Binary Interface）——一套标准接口规范，类似 x86 的 BIOS/UEFI。OpenSBI 是这套规范的具体实现，跑在 M-mode；Linux 通过 `ecall` 陷入 M-mode，调用 SBI 提供的功能（比如设置 timer、hart 管理、早期控制台输出）。

```
Linux 内核 (S-mode)
  li  a7, 0x54494d45   # SBI 功能号：设置 timer
  ecall                # 陷入 M-mode
    ↓
OpenSBI (M-mode)       # 真正操作定时器硬件
  sret                 # 返回 S-mode
    ↓
Linux 继续执行
```

> SBI 规范：https://github.com/riscv-non-isa/riscv-sbi-doc
> OpenSBI 实现：https://github.com/riscv-software-src/opensbi

---

## 第四步：从上电到 shell（完整启动过程）

```
上电
  ↓
OpenSBI (M-mode) 启动
  — 初始化 DRAM、CLINT（timer）、PLIC（中断控制器）
  — 切换到 S-mode，跳转到 Linux 内核入口
  ↓
Linux _start  (arch/riscv/kernel/head.S)
  — 关中断、设置栈指针（sp）、清 BSS 段
  ↓
setup_vm_final()
  — 写 satp CSR → 开启 Sv39 虚拟内存
  — 从这里开始，所有地址都是虚拟地址
  ↓
start_kernel()  (init/main.c)
  — 解析 DTB（设备树，描述板子上有哪些硬件）
  — 初始化调度器、内存分配器、驱动
  — 启动 init 进程（PID 1）
  ↓
Shell 出现 / systemd 接管
```

关键转折点：`setup_vm_final()` 写 `satp` 之后虚拟内存开启，这是 Linux 与裸机程序最本质的分界线。

> **Reference**: OpenSBI 项目 — https://github.com/riscv-software-src/opensbi
> **Reference**: Linux 内核文档 RISC-V 章节 — https://www.kernel.org/doc/html/latest/riscv/

---

## 第五步：系统调用怎么走完整一圈

以 `open("file.txt")` 为例，追踪一次完整的系统调用，把前面讲的 `ecall`、trap、特权级切换串起来看：

```c
用户程序（U-mode）
  open("file.txt", O_RDONLY)
    ↓ glibc 封装为:
    li  a7, 56        # 系统调用号 56 = openat
    ecall             # CPU 陷入 trap，跳转到 stvec 指向的内核入口

内核（S-mode）
  do_trap_ecall_u()          # trap 入口
    → 读 a7 = 56，查系统调用表
    → 执行 sys_openat()
    → 结果写回 a0（文件描述符）
    sret                     # 返回 U-mode

用户程序
  fd = a0                    # 拿到返回值，继续执行
```

记住这条主线：**`ecall` → trap → 内核处理 → `sret` 返回**。这个模式在 RISC-V 上通用于所有系统调用，也是第三步 SBI 调用使用的同一套机制（只是目标从 S-mode 换成了 M-mode）。

> **Reference**: RISC-V SBI 规范（`ecall` 进入 M-mode 的格式与系统调用相同）
> — https://github.com/riscv-non-isa/riscv-sbi-doc

---

## Linux 支持 RISC-V 的历史

讲完技术机制，补充一些历史背景，看看 Linux + RISC-V 这个组合是怎么走到今天的。

| 时间 | 事件 |
|---|---|
| **2017** | Palmer Dabbelt 提交首个 RISC-V Linux 补丁 |
| **2018 / v4.15** | RV64GC 正式合并 Linux 主线内核 |
| **2019** | SiFive HiFive Unleashed：第一块能跑主线 Linux 的开发板 |
| **2022** | StarFive VisionFive 2 发布，低成本 Linux SBC 普及 |
| **2023+** | Milk-V Pioneer（64 核）可运行完整 Debian/Ubuntu |

RISC-V 官网的定性描述：

> "RISC-V supports the open-source operating system Linux natively, providing a flexible, open software-hardware stack."
> — https://riscv.org/blog/tag/linux/

技术细节补充：Linux 从 **v4.15（2018）** 起正式支持 RISC-V（`arch/riscv/` 目录），主要支持 RV64GC（64 位，G = IMAFD + C 扩展）；RV32 支持有限，主流是 RV64。

> **Reference**: "Full-Fat, Kernel-Ready: Why RISC-V Linux Needs Everyone Upstream"
> — https://riscv.org/blog/risc-v-upstreaming/

---

## 动手：用 QEMU 跑起来（10 分钟实验）

理解这一课最好的方式是亲眼看到 Linux shell 出现在 RISC-V 虚拟机里。

```bash
# 用 Buildroot 构建最小系统（约 15 分钟）
git clone https://github.com/buildroot/buildroot
cd buildroot
make qemu_riscv64_virt_defconfig
make -j$(nproc)

# 用 QEMU 启动
qemu-system-riscv64 \
  -machine virt -nographic \
  -bios output/images/fw_dynamic.bin \
  -kernel output/images/Image \
  -drive file=output/images/rootfs.ext2,format=raw \
  -append "root=/dev/vda rw console=ttyS0"
```

看到 `buildroot login:` 提示符就成功了。此刻你运行的正是本课的三层结构：OpenSBI（M-mode）→ Linux 内核（S-mode）→ BusyBox shell（U-mode）。

> **Reference**: QEMU RISC-V 文档 — https://www.qemu.org/docs/master/system/target-riscv.html
> **Reference**: Buildroot 项目 — https://buildroot.org/

---

## 真实开发板选择

| 开发板 | 价格区间 | 适合学习 |
|---|---|---|
| **SiFive HiFive Unleashed** | 较高 | 第一块主线 Linux 板，参考意义强 |
| **StarFive VisionFive 2** | ¥300 左右 | 最具性价比的 Linux SBC |
| **Milk-V Pioneer** | 较高 | 64 核，服务器体验 |

> **Reference**: RISC-V 官网开发板页
> — https://riscv.org/developers/boards/

---

## 暂时跳过

先不用研究 DTB/设备树格式、驱动模型（platform bus）、PLIC 中断控制器细节、或者如何编译自己的内核。先把 QEMU 跑起来，能看到 Linux shell，已经理解了整个软硬件栈。

---

## 过关标准

能回答：

1. Linux 运行在哪个特权模式？OpenSBI 呢？
2. 用户程序请求读文件，控制权经过哪几个层次？
3. Linux 怎么启用虚拟内存（用到哪个 CSR）？

---

## 深入阅读

- [[../05-调试与工具链/software/notes|软件生态笔记]]（工具链选型、QEMU 调试命令、内核启动路径）
- [[../02-特权架构/notes|特权架构笔记]]（M/S/U 模式详细规则）
- [[../atoms/Sv39|Sv39]]（Linux 页表格式）
- [[../atoms/trap|Trap]]（ecall 陷入机制）
- [[../05-调试与工具链/platform/notes|平台规范笔记]]（SBI 接口、设备树）
- [Linux 内核 RISC-V 文档](https://www.kernel.org/doc/html/latest/riscv/)
- [riscv.org Linux 相关博客](https://riscv.org/blog/tag/linux/)
