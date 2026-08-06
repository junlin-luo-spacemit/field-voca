# RISC-V 特权架构学习笔记

> 文档入口：
> - ISA Manual Vol.II (Privileged): https://github.com/riscv/riscv-isa-manual
> - SBI Spec: https://github.com/riscv-non-isa/riscv-sbi-doc

---

## 特权模式的软件分层模型

```
┌──────────────────────────────────────┐
│  U-mode: 用户应用（App）               │
├──────────────────────────────────────┤
│  S-mode: OS 内核 (Linux/RTOS)         │  ← ecall → M-mode (SBI)
│           管理虚拟内存 (satp/页表)     │
├──────────────────────────────────────┤
│  M-mode: 固件 (OpenSBI)               │  ← 直接访问硬件
│           初始化硬件 / 处理 M 级 trap  │
└──────────────────────────────────────┘
           ↕ 物理硬件
```

关键认知：M 模式相当于 x86 的 BIOS/UEFI 固件层，S 模式相当于 OS 内核，U 模式是应用程序。Linux 运行在 S 模式，通过 SBI（Supervisor Binary Interface）调用 M 模式固件提供的服务（如定时器、HSM 核心管理）。

---

## Trap 处理流程（以 U→S 模式 ecall 为例）

```
1. U 模式执行 ecall 指令
2. 硬件自动：
   - sstatus.SPP ← U（记录来源模式）
   - sstatus.SPIE ← sstatus.SIE（保存中断使能）
   - sstatus.SIE ← 0（关闭中断）
   - sepc ← PC（保存返回地址）
   - scause ← 8（Environment call from U-mode）
   - stval ← 0（ecall 无附加信息）
   - PC ← stvec（跳转到 trap 处理函数）
3. 软件（内核 trap handler）：
   - 保存所有通用寄存器（陷阱帧）
   - 根据 scause 分发处理（ecall / 缺页 / 非法指令...）
   - 处理完毕，恢复寄存器
   - sret 返回（PC←sepc，模式←SPP，SIE←SPIE）
```

---

## 虚拟内存：Sv39 地址翻译过程

39 位虚拟地址结构：
```
VA[38:30] = VPN[2]  (9位，一级页表索引)
VA[29:21] = VPN[1]  (9位，二级页表索引)
VA[20:12] = VPN[0]  (9位，三级页表索引)
VA[11:0]  = Page Offset (12位)
```

翻译步骤：
```
1. 从 satp.PPN 找到根页表物理地址
2. 用 VPN[2] 索引根页表，找到二级页表 PPN
3. 用 VPN[1] 索引二级页表，找到三级页表 PPN
4. 用 VPN[0] 索引三级页表，找到物理页 PPN
5. PA = PPN << 12 | VA[11:0]
```

大页（Huge Page）优化：若页表项 R/W/X 非全零，则该 PTE 是叶节点（Leaf PTE），此时：
- 在第1级（VPN[2]）命中 → 1 GiB 大页（Sv39 Gigapage）
- 在第2级（VPN[1]）命中 → 2 MiB 大页（Sv39 Megapage）

---

## CSR 访问权限规则

CSR 地址的高 4 位编码了访问权限：
```
csr[11:10] = 00/01/10 → 读写 CSR
           = 11       → 只读 CSR（写入触发非法指令异常）

csr[9:8]   = 00       → U 模式可访问
           = 01       → S 模式可访问
           = 10       → Hypervisor 可访问
           = 11       → M 模式可访问
```

在低特权级访问高特权级 CSR → 立即触发 Illegal Instruction 异常。这是 RISC-V 特权隔离的硬件基础。

---

## 中断控制关键路径

```
外部中断到达
  ↓
mip.MEIP = 1（由 PLIC/ACLINT 设置）
  ↓
检查 mstatus.MIE = 1 且 mie.MEIE = 1？
  ↓ 是
中断被接受，硬件保存现场 → 跳转 mtvec
  ↓
trap handler：读 mcause（最高位=1 表示中断）
  → 中断号 11 = MEI → 调用 PLIC 完成（claim/complete）
  ↓
mret 恢复
```

若 S 模式需要处理外部中断：M 模式通过 `mideleg` 将中断委托给 S 模式，则中断直接触发 S 模式 trap，降低延迟。

---

## 待深入阅读

- [ ] ISA Manual Vol.II 全文（完整 CSR 列表、虚拟化 H 扩展）
- [ ] Linux RISC-V 内核启动流程（arch/riscv/kernel/）
- [ ] OpenSBI 源码（M 模式固件实现）
- [ ] RISC-V PMP 配置实战（嵌入式安全隔离）
