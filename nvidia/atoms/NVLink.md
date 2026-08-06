---
tags: [nvidia, networking, hardware, atom]
domain: networking
aliases: [NVLink, NV-Link]
date: 2026-06-29
source: https://docs.nvidia.com/
---

# NVLink / NVLink

## 定义

**NVLink** 是 NVIDIA 专有的高速 GPU-to-GPU 互连技术，用于在多 GPU 系统（NVLink Switch、DGX）中实现极高带宽的点对点通信，带宽远超 PCIe。

## 官方原文

> "NVLink provides high-speed, high-bandwidth peer-to-peer data transfers between GPUs within a node."

## 各版本规格

| 版本 | 架构 | 双向带宽/Link | GPU间总带宽 |
|------|------|--------------|------------|
| NVLink 3.0 | Ampere (A100) | 100 GB/s | 600 GB/s（12 links） |
| NVLink 4.0 | Hopper (H100) | 100 GB/s | 900 GB/s（18 links） |
| NVLink 5.0 | Blackwell (B200) | 200 GB/s | 1.8 TB/s（18 links） |

## NVLink Switch（NVSwitch）

- 用于连接节点内多个 GPU 的交换芯片
- DGX H100 含 4 个 NVSwitch 芯片，8块 H100 之间全互连带宽 = 900 GB/s × 8 = 7.2 TB/s（全双工）
- NVLink 网络（NVLink Switch System）可跨节点扩展到 256 GPU

## 与 PCIe 对比

| 指标 | NVLink 4.0 | PCIe 5.0 × 16 |
|------|-----------|---------------|
| 双向带宽 | 900 GB/s | 128 GB/s |
| 延迟 | 低 | 相对高 |
| 用途 | GPU间 | CPU-GPU |

## 相关词条

- [[HBM]] — NVLink 传输的数据来自/到 HBM
- [[SM]] — 数据经 NVLink 后到达 SM
- [[Tensor-Core]] — NVLink 为多 GPU 训练的 Tensor Core 喂数据
- NCCL — 利用 NVLink 执行集合通信（AllReduce 等）
