---
tags: [nvidia, hardware, memory, atom]
domain: hardware
aliases: [HBM, High Bandwidth Memory, 高带宽内存]
date: 2026-06-29
source: https://docs.nvidia.com/
---

# HBM — High Bandwidth Memory / 高带宽内存

## 定义

**HBM**（High Bandwidth Memory）是一种通过硅中介层（interposer）与 GPU 紧密封装的高带宽 DRAM，是数据中心 GPU（A100、H100、B200）的主存储器，带宽远超传统 GDDR。

## 官方原文

> "H100 SXM5 features 80 GB HBM3 memory with 3.35 TB/s of memory bandwidth."

## 各代规格对比

| 版本 | 代表产品 | 带宽 | 容量 |
|------|----------|------|------|
| HBM2 | V100 | 900 GB/s | 32 GB |
| HBM2e | A100 | 2.0 TB/s | 80 GB |
| HBM3 | H100 SXM5 | **3.35 TB/s** | 80 GB |
| HBM3e | H200 | 4.8 TB/s | 141 GB |

## 关键特点

- 通过 **2.5D 封装**（硅中介层）与 GPU die 相连，极短的物理距离带来极高带宽
- 多个 HBM stack 堆叠在 GPU 旁边，H100 有 5 个 HBM3 stack
- 带宽是 [[Tensor-Core]] 算力的瓶颈之一，即 **memory wall** 问题
- [[Unified Memory]] 和 NVLink 的数据最终都要经过 HBM

## 与 GDDR 对比

| 指标 | HBM3 | GDDR6X |
|------|------|--------|
| 带宽 | 3.35 TB/s | ~1 TB/s |
| 功耗 | 低 | 高 |
| 成本 | 高 | 低 |
| 用途 | 数据中心 GPU | 消费级 GPU |

## 相关词条

- [[Tensor-Core]] — HBM 是其数据来源
- [[SM]] — 通过 L2 cache 访问 HBM
- [[NVLink]] — GPU 间通信带宽与 HBM 带宽互补
- [[Unified-Memory]] — 对 HBM 和系统内存的统一抽象
