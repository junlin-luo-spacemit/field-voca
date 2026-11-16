---
tags: [tools, trace, performance, spacemit]
domain: tools
aliases: [Trace Tool, 跟踪工具, Trace, 执行跟踪]
status: approved
created: 2026-08-06
updated: 2026-11-16
source: C:\Users\luojunlin\docs-tool\zh\user_guide\trace_user_guide.md
superseded_by:
---

# Trace Tool（跟踪工具）

**中文译名**：跟踪工具 / 执行跟踪工具

## 定义

用于记录和分析程序执行流程与性能的工具，可包括系统级或硬件级跟踪，不影响程序实时运行。

## 为什么重要

- **性能分析**：找出性能瓶颈与热点函数
- **行为验证**：验证程序执行路径
- **非侵入式**：不暂停程序，保持实时性
- **大规模数据**：记录完整执行历史

## 跟踪类型

1. **指令跟踪**
   - 记录每条指令执行
   - PC（程序计数器）流
   - 分支跳转记录

2. **函数跟踪**
   - 函数调用关系（Call Graph）
   - 函数执行时间
   - 调用频次统计

3. **系统跟踪**
   - 系统调用记录
   - 进程/线程切换
   - 中断处理

4. **性能事件跟踪**
   - Cache miss
   - 分支预测失败
   - TLB miss

## 工具示例

- **RISC-V Trace**：RISC-V N-Trace（Nexus Trace）协议 —— **SpacemiT K3 采用此方案**
- **软件跟踪**：Linux `perf`、`ftrace`
- **配套工具**：NexRv（读取并解析 Trace 数据）

> ⚠️ 不要混入 ARM 的 **CoreSight ETM** —— 那是 ARM 架构专用，RISC-V 芯片不使用。

## 分析流程

```
1. 配置跟踪选项（跟踪范围、事件类型）
2. 运行程序并记录
3. 停止跟踪，导出数据
4. 结合匹配的 ELF 文件解析 PC 跳转轨迹
5. 定位性能瓶颈
```

## 在 SpacemiT 场景

- **K3 系列**基于 RISC-V N-Trace 协议实现，OpenOCD 配置脚本为 `tcl/spacemit/spacemit-k3-trace.cfg`
- 典型用途：**死机问题分析** —— 系统崩溃前启动 Trace，崩溃后通过 JTAG 读回数据，重建程序 PC 跳转轨迹
- 启动 Trace 的三种方式：**JTAG 寄存器操作**、**Linux sysfs 节点**、**perf 工具**

**已知限制**（来自官方手册）：

- 只记录**跳转指令**，不记录非跳转指令，也不记录大部分指令的绝对地址，因此必须用**严格匹配的 ELF 文件**才能解析
- 不记录 load/store 的数据结果，仅记录跳转（当前 N-Trace 规范尚未支持 Data Trace）
- 多进程环境下，需要 Linux 内核在调度时写入 `scontext` CSR

## 与 JTAG Debugging 的区别

| 特性 | Trace Tool | JTAG Debugging |
|------|-----------|----------------|
| 功能 | 记录执行流程 | 暂停、断点、单步 |
| 性能影响 | 不影响实时性 | 暂停程序运行 |
| 适用场景 | 性能分析、热点定位 | 逐行调试 |

## 相关词条

- [[JTAG-Debugging]] — 断点调试工具（Trace 数据也常通过 JTAG 读取）
- [[SpacemiT-Studio]] — 集成开发环境
- [[../hardware/products/K3]] — K3 支持 RISC-V N-Trace

## 来源文档

- `docs-tool/zh/user_guide/trace_user_guide.md`（官方 Trace 使用手册）
