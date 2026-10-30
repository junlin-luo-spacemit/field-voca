---
tags: [atom, core, X60, spacemit]
domain: hardware
aliases: [X60-Core, X60 Core, X60核心]
status: needs_review
created: 2026-08-06
updated: 2026-10-29
source:
superseded_by:
---

# X60 Core

**中文译名**：X60 核心

## 定义

X60 Core 是 SpacemiT K1 芯片采用的 RISC-V CPU 核心架构，面向边缘泛 AI 应用设计，具备可选的 AI 扩展指令。

## 关键参数

- **架构**：RISC-V 64-bit (RV64GCVB / RVA22)
- **核心配置**：K1 采用双簇设计
  - Cluster0：4×X60（含 AI 扩展指令）
  - Cluster1：4×X60（标准配置）
- **缓存**：
  - L1：每核 32KB I-Cache + 32KB D-Cache
  - L2：每簇 512KB
  - TCM：512KB（Cluster0）
- **AI 能力**：Cluster0 AI 扩展约 2.0 TOPS

## AI 扩展指令

Cluster0 的 X60 核心包含 AI 自定义指令（如 `smt.vmadot`），用于加速矩阵运算。这是**指令集级别**的扩展，不同于 K3 的独立 A100 AI 核心。

## 与 X100 Core 的区别

| 特性 | X60 Core | X100 Core |
|------|----------|-----------|
| 定位 | 边缘泛 AI 应用 | 高性能 AI 计算 |
| 使用产品 | K1 | K3 |
| L1 Cache | 32KB/核 | 64KB/核 |
| AI 能力 | 指令集扩展（约 2 TOPS） | 配合 A100 AI 核心（约 60 TOPS） |

## 在 SpacemiT 产品中

- **K1**：8×X60 核心（双簇），Cluster0 带 AI 扩展
- 适用场景：单板机、教育套件、边缘 AI 推理

## 为什么重要

芯片的核心微架构决定了单核性能、指令集支持与功耗特性，是理解 SoC 性能的重要基础。

## 相关词条

- [[X100-Core]] — K3 的高性能核心
- [[Matrix-Extension]] — SpacemiT 矩阵加速技术
- [[SoC]] — 系统级芯片
- [[../hardware/products/K1]] — 使用 X60 核心的产品

## 来源文档

- K1 Datasheet
- K1 User Manual（Chapter 8: CPU System）
