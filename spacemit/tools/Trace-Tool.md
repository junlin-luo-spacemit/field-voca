---
tags: [tools, trace, performance, spacemit]
domain: tools
aliases: [Trace Tool, 跟踪工具, Trace, 执行跟踪]
status: approved
created: 2026-08-06
updated: 2026-10-29
source: C:\Users\luojunlin\docs-buildroot\zh\k1_buildroot\device\
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

- **硬件跟踪**：ARM CoreSight ETM、RISC-V Trace
- **软件跟踪**：perf、ftrace、LTTng
- **可视化**：Chrome Tracing、Perfetto

## 分析流程

```
1. 配置跟踪选项（跟踪范围、事件类型）
2. 运行程序并记录
3. 停止跟踪，导出数据
4. 使用分析工具可视化与统计
5. 定位性能瓶颈
```

## 在 SpacemiT 场景

- 用于性能分析、瓶颈定位与系统行为验证
- K1/K3 芯片级跟踪支持
- AI 推理性能优化

## 与 JTAG Debugging 的区别

| 特性 | Trace Tool | JTAG Debugging |
|------|-----------|----------------|
| 功能 | 记录执行流程 | 暂停、断点、单步 |
| 性能影响 | 不影响实时性 | 暂停程序运行 |
| 适用场景 | 性能分析、热点定位 | 逐行调试 |

## 相关词条

- [[JTAG-Debugging]] — 断点调试工具
- [[SpacemiT-Studio]] — IDE 集成工具
- [[../hardware/products/K1]] — K1 跟踪支持
- [[../hardware/products/K3]] — K3 跟踪支持

## 来源文档

- SpacemiT 文档中心：Tools / Trace User Guide
