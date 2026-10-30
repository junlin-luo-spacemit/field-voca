---
tags: [atom, memory, HBM, spacemit]
domain: hardware
aliases: [HBM, High Bandwidth Memory, 高带宽内存]
status: needs_review
created: 2026-08-06
updated: 2026-10-29
source:
superseded_by:
---

# HBM（High Bandwidth Memory）

**中文译名**：高带宽内存

## 定义

HBM 是一种高带宽、低功耗的堆叠式内存，常用于要求高内存带宽的 AI 加速器与高性能计算场景。

## 为什么重要

矩阵运算高度依赖内存带宽；使用 HBM 的 SoC 在执行大型模型和数据集时能显著减少内存瓶颈。

## 关注点

- 带宽（GB/s）
- 容量（GB）
- 接口与封装要求

## 在 SpacemiT 中

**当前状态**：SpacemiT K1/K3 采用 LPDDR4x / LPDDR5，未使用 HBM。

**HBM 为对比参考**：
- HBM 主要用于数据中心 GPU（如 NVIDIA H100 配备 HBM3，带宽 3.35 TB/s）
- K3 采用 LPDDR5-6400（带宽约 51.2 GB/s），成本与功耗更适合边缘场景
- 两者定位不同：HBM 追求极致带宽，LPDDR 平衡成本/功耗/性能

## 与 LPDDR 对比

| 特性 | HBM3 (典型) | LPDDR5 (K3) |
|------|-----------|------------|
| 带宽 | >3 TB/s | ~51 GB/s |
| 容量 | 24-80 GB | 最高 32 GB |
| 功耗 | 高 | 低 |
| 成本 | 极高 | 中 |
| 应用 | 数据中心 GPU | 边缘/移动 SoC |

## 相关词条

- [[SoC]] — 系统级芯片
- [[../hardware/products/K3]] — K3 内存规格
