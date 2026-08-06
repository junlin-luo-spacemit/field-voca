# RISC-V 微架构学习笔记

> 注：微架构属于实现层面，ISA 规范不强制规定，此处以主流开源实现为参考

---

## 顺序核 vs 乱序核的典型 RISC-V 实现

| 核心 | 类型 | 流水线深度 | 典型 IPC | 场景 |
|------|------|-----------|---------|------|
| PicoRV32 | 顺序，单发射 | 3–4 | < 1 | FPGA 软核，极小面积 |
| SiFive E2 | 顺序，单发射 | 5 | ~1 | 嵌入式 MCU |
| Rocket | 顺序，单发射 | 5 | ~1 | 参考实现，带 TLB/MMU |
| SiFive U7 / CVA6 | 顺序，单发射 | 6 | ~1.2 | 应用处理器（Linux） |
| T-Head C910 | 乱序，3发射 | 12 | ~3 | 高性能 64位应用 |
| BOOM | 乱序，超标量 | 可配置 | 2–4 | 研究/高性能 |

---

## C 扩展对微架构的挑战

16/32 位混合指令使取指单元更复杂：

```
取 4 字节数据：[ 16bit A ][ 16bit B ] 可能是：
  - 两条 16 位指令 A, B
  - 一条 32 位指令 A|B（若 A[1:0] != 11）
  - 一条 16 位指令 A + 32 位指令的前 16 位
```

实现需要跨缓存行的指令拼接逻辑（instruction realignment buffer），是 RISC-V 设计中不那么"简洁"的地方。

---

## 内存序与 FENCE 的微架构影响

RISC-V 的 RVWMO（弱内存序）允许：
- Store buffer：写入先进 buffer，延迟更新缓存（Store-Load 重排）
- Load speculation：推测式提前加载

这意味着：
- **顺序核**：天然满足 RVWMO（比需要的还强），FENCE 可实现为 nop
- **乱序核**：需要硬件跟踪 FENCE 的语义，排空 store buffer、阻止越界重排

`FENCE.TSO`（TSO 内存序，比 RVWMO 更强但比顺序一致性弱）可以减少 x86 程序移植时需要插入的屏障数量。

---

## 分支预测对 RISC-V 程序的影响

RISC-V 没有条件执行指令（不像 ARM Thumb2 的 `IT` 块），分支密集代码对预测器要求更高。`Zicond` 扩展（条件置零）可将简单的条件赋值转换为无分支代码：

```asm
# 传统方式（需要分支）
beqz a1, skip
mv a0, a2
skip:

# Zicond 无分支方式
czero.eqz t0, a2, a1    # t0 = (a1==0) ? 0 : a2
czero.nez t1, a0, a1    # t1 = (a1!=0) ? 0 : a0
or a0, t0, t1
```

---

## 待深入阅读

- [ ] Rocket Chip 生成器文档（Chisel 参数化设计）
- [ ] BOOM 微架构论文（UC Berkeley Tech Report）
- [ ] CVA6 微架构规格文档
- [ ] RISC-V 缓存一致性实现（TileLink 协议）
