# RISC-V 平台规范学习笔记

> 文档入口：
> - SBI Spec: https://github.com/riscv-non-isa/riscv-sbi-doc
> - Platform Specs: https://github.com/riscv/riscv-platform-specs
> - AIA Spec: https://github.com/riscv/riscv-aia

---

## 软件层次与接口边界

```
┌──────────────────────────────────────────────┐
│  用户应用（User App）                          │  U-mode
├──────────────────────────────────────────────┤
│  操作系统内核（Linux / RTOS）                  │  S-mode
│     ↑ 设备驱动通过 Device Tree / ACPI 初始化  │
│     ↑ 时钟、IPI、系统复位 通过 SBI ecall      │
├──────────────────────────────────────────────┤
│  SBI 接口层（OpenSBI FW_DYNAMIC / FW_JUMP）   │  M-mode
│     ↑ 从 Bootloader 加载                     │
├──────────────────────────────────────────────┤
│  Bootloader（U-Boot SPL / GRUB）              │  M→S-mode
├──────────────────────────────────────────────┤
│  ROM / BootROM                                │  M-mode
└──────────────────────────────────────────────┘
```

关键认知：RISC-V 没有强制规定固件（M-mode 软件）的实现，但 SBI 规范统一了 S-mode OS 与 M-mode 固件的接口，使同一个 Linux 内核可以运行在任何实现了 SBI 的平台上（无论固件是 OpenSBI、TF-A RISC-V 还是厂商自研固件）。

---

## SBI 与 Linux 内核的交互

Linux 内核通过两种方式获得平台信息：
1. **Device Tree（DTB）**：Bootloader 将 DTB 地址传入 `a1` 寄存器，内核据此发现设备
2. **SBI 调用**：运行时服务通过 `ecall` 按需调用

常见 Linux 内核 SBI 调用时机：
| 时机 | SBI 调用 |
|------|---------|
| SMP 启动辅助核 | `sbi_hart_start`（HSM） |
| 设置定时器中断 | `sbi_set_timer`（TIME） |
| 发送 IPI | `sbi_send_ipi`（sPI） |
| TLB shootdown | `sbi_remote_sfence_vma`（RFNC） |
| 系统关机/重启 | `sbi_system_reset`（SRST） |
| 早期控制台输出 | `sbi_debug_console_write`（DBCN） |

---

## PLIC vs AIA 的演进逻辑

| 特性 | PLIC | AIA（APLIC+IMSIC） |
|------|------|-------------------|
| 中断类型 | 线状中断（wire） | 线状 + MSI |
| 虚拟化支持 | 无 | 原生支持（IMSIC 虚拟化） |
| 可扩展性 | 受限于中断源数量 | MSI 理论上无限 |
| PCIe 兼容 | 需要额外转换 | 原生 MSI 支持 |
| 现有成熟度 | ✅ 广泛部署 | 新兴，高端 SoC 采用 |

对嵌入式 MCU：用 ACLINT（本地定时器+IPI）+ PLIC（外部中断）即可。
对服务器/高性能 SoC：AIA 是长期演进方向。

---

## 典型启动流程（QEMU + OpenSBI + U-Boot + Linux）

```
1. QEMU 复位 → hart 0 从 0x1000 开始执行 reset vector
2. Reset vector 跳转到 0x80000000 → OpenSBI FW_DYNAMIC
3. OpenSBI 初始化：
   - 初始化 M 模式 CSR（mtvec、mie 等）
   - 发现/启动辅助 hart（HSM）
   - 准备 SBI 服务
4. OpenSBI 切换到 S-mode → U-Boot（或直接跳 Linux）
5. U-Boot：初始化 DDR、外设，加载 kernel Image + DTB
6. U-Boot 跳转到 Linux kernel（S-mode）
   - a0 = hartid，a1 = DTB 物理地址
7. Linux 启动，读取 DTB 初始化驱动，挂载根文件系统
```

---

## 待深入阅读

- [ ] SBI Specification 全文（各 Extension 的参数和返回值）
- [ ] OpenSBI 源码结构（lib/sbi/ 目录）
- [ ] AIA Specification（IMSIC 硬件模型）
- [ ] RISC-V Server Platform Specification
- [ ] UEFI on RISC-V（edk2 RISC-V 移植）
