---
tags: [atom, soc, hardware, spacemit]
domain: hardware
aliases: [SoC, System on Chip, 系统级芯片]
status: needs_review
created: 2026-08-06
updated: 2026-10-29
source:
superseded_by:
---

# SoC（系统级芯片）

**中文译名**：系统级芯片

## 定义

SoC（System on Chip，系统级芯片）将 CPU、GPU/AI 加速单元、内存控制器、I/O 控制器等多个子系统集成到一颗芯片上，减少系统体积并提高数据吞吐效率。

## 为什么重要

SoC 是嵌入式与边缘设备的核心：选择不同 SoC 会影响性能、功耗、接口和开发生态。

## 关键属性（常见关注点）

- CPU 架构与核数
- 专用加速单元（Matrix/Vector 引擎）
- 内存类型与带宽（LPDDR / HBM）
- 外设与 I/O（PCIe、Ethernet、USB、M.2 等）
- 功耗与散热（TDP、热设计）

## 在 SpacemiT 中

- 典型产品：
  - [[../hardware/products/K1]] — 轻量级 AI SoC，8×X60 CPU + AI 扩展，约 2 TOPS
  - [[../hardware/products/K3]] — 高性能 AI SoC，8×X100 CPU + 8×A100 AI Core，约 60 TOPS
- 文档位置：参考 SpacemiT 产品页与芯片概述

## 相关词条

- [[X60-Core]] — K1 的 CPU 核心
- [[X100-Core]] — K3 的 CPU 核心
- [[A100-AI-Core]] — K3 的 AI 加速核心
- [[Matrix-Extension]] — SpacemiT 矩阵加速技术
