---
tags: [study, m3, memory, lpddr, bandwidth, spacemit]
domain: chip
aliases: [LPDDR 与带宽, LPDDR and Bandwidth]
status: needs_review
created: 2026-11-19
updated: 2026-11-19
---

# LPDDR 与内存带宽

> **本讲目标**：理解 K3 支持的内存类型（LPDDR4x/LPDDR5）的规格差异，掌握带宽计算公式，理解为什么内存带宽是 AI 推理的瓶颈。
>
> **📌 说明**：本模块以 **K3** 为例讲解内存规格，但 LPDDR 类型选择、带宽计算公式、瓶颈判断等概念适用于所有 SoC 芯片。

## 一、K3 支持的内存类型

K3 支持两种 [LPDDR](GLOSSARY.md#lpddr5) 内存类型：

| 类型 | 最高数据速率 | 适用场景 |
|---|---|---|
| LPDDR4x | 4266 MT/s | 低成本方案 |
| LPDDR5 | 6400 MT/s | 高性能方案 |

**两者的共同配置**：
- 双通道架构（Dual Channel）
- 每通道 32 位数据宽度
- 每通道支持两个 Rank
- 最大寻址空间 32 GB

**动态频率调节（DFS，Dynamic Frequency Scaling）**：K3 支持根据带宽需求实时调整内存频率，在低负载时降低频率以节省功耗。

## 二、带宽计算

**[带宽](GLOSSARY.md#带宽)计算公式**：

$$\text{带宽} = \text{通道数} \times \text{每通道位宽（字节）} \times \text{数据速率}$$

**LPDDR5 的理论带宽**：

$$2 \times 4\text{ 字节} \times 6400\text{ MT/s} = 51.2\text{ GB/s}$$

**LPDDR4x 的理论带宽**：

$$2 \times 4\text{ 字节} \times 4266\text{ MT/s} = 34.1\text{ GB/s}$$

**关键点**：32 位 = 4 字节，MT/s（Million Transfers per Second）是每秒百万次传输次数。

## 三、为什么带宽是 AI 的瓶颈

AI 推理的三个决定因素：**算力、带宽、容量**。

```
任务能跑多快？
  → 取决于三者中最紧的那个

算力足够但带宽不足  → A100 空转等待数据
带宽足够但容量不足  → 模型放不下内存
算力和带宽够但容量不足 → 推理跑不起来
```

**AI 推理的数据流**：
1. 模型权重存在内存（LPDDR5）
2. A100 核心从内存读取权重和输入数据
3. A100 计算，结果写回内存
4. 下一层继续读取

每一步 A100 都需要从内存读取数据。**如果内存带宽不足，A100 再强也发挥不出来。**

**LPDDR5 vs LPDDR4x 的实际差异**：

| 场景 | LPDDR4x | LPDDR5 |
|---|---|---|
| 理论带宽 | 34.1 GB/s | 51.2 GB/s |
| AI 推理（带宽瓶颈） | 受限 | 明显提升 |
| AI 推理（算力瓶颈） | 无差异 | 无差异 |
| 通用计算 | 通常够用 | 略有提升 |

## 四、Rank 的作用

**Rank** 是内存模组内的一组独立存储单元。每通道支持两个 Rank，可以在一个 Rank 刷新时访问另一个，提高内存利用率。

类比：两个收银台同时工作，一个结账时另一个接待下一位顾客。

## 快速自测

1. LPDDR5 的理论带宽怎么算？给出计算过程。
2. 为什么 LPDDR5 比 LPDDR4x 快，但配 LPDDR5 的板子 AI 不一定快？
3. DFS 的作用是什么？

## 参考来源

| 来源 | 章节 | 内容 |
|---|---|---|
| docs-chip · key_stone/k3/k3_docs/k3_ds.md | 2.2.2 LPDDR4x/5 | 内存类型、速率、双通道、DFS |
