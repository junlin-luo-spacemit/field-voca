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
- 最大寻址空间 32 GB

**动态频率调节（DFS，Dynamic Frequency Scaling）**：K3 支持根据带宽需求实时调整内存频率，在低负载时降低频率以节省功耗。

## 二、带宽计算

**[带宽](GLOSSARY.md#带宽)计算公式**：通道数 × 每通道位宽（字节）× 数据速率

| 类型 | 计算 | 理论带宽 |
|---|---|---|
| LPDDR5 | 2 × 4 字节 × 6400 MT/s | **51.2 GB/s** |
| LPDDR4x | 2 × 4 字节 × 4266 MT/s | **34.1 GB/s** |

**关键点**：32 位 = 4 字节。[MT/s](GLOSSARY.md#lpddr5)（Million Transfers per Second）是每秒百万次传输次数，不等于 MHz。

## 三、为什么带宽是 AI 的瓶颈

AI 推理的速度由三个因素决定：**算力、带宽、容量**，取决于三者中最紧的那个。

```
算力足够但带宽不足  → A100 空转等待数据，快了也没用
带宽足够但容量不足  → 模型放不下内存，跑不起来
容量够但算力不足    → 计算太慢，带宽再大也无法弥补
```

**实际影响**：

| 场景 | LPDDR4x | LPDDR5 |
|---|---|---|
| AI 推理（带宽瓶颈） | 受限 | 明显提升 |
| AI 推理（算力瓶颈） | 无差异 | 无差异 |
| 通用计算 | 通常够用 | 略有提升 |

因此 LPDDR5 比 LPDDR4x 快，但配 LPDDR5 的产品 AI 不一定更快——要看瓶颈在哪里。

## 快速自测

1. LPDDR5 的理论带宽怎么算？给出计算过程。
2. 为什么 LPDDR5 比 LPDDR4x 快，但配 LPDDR5 的板子 AI 不一定快？
3. DFS 的作用是什么？

## 参考来源

| 来源 | 章节 | 内容 |
|---|---|---|
| docs-chip · key_stone/k3/k3_docs/k3_ds.md | 2.2.2 LPDDR4x/5 | 内存类型、速率、双通道、DFS |
