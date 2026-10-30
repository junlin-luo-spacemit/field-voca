---
tags: [atom, core, X100, spacemit]
domain: hardware
aliases: [X100-Core, X100 Core, X100核心]
status: approved
created: 2026-10-29
updated: 2026-10-29
source: C:\Users\luojunlin\docs-chip\zh\key_stone\
superseded_by:
---

# X100 Core

**中文译名**：X100 核心

## 定义

X100 Core 是 SpacemiT K3 芯片采用的高性能 RISC-V CPU 核心架构，定位为服务器与边缘 AI 计算的性能核心（大核）。

## 关键参数

- **架构**：RISC-V 64-bit (RV64GCVB)
- **核心数量**：K3 采用 8×X100 配置
- **主频**：最高 2.4 GHz
- **缓存**：
  - L1：每核 64KB I-Cache + 64KB D-Cache
  - L2：2MB 共享（8 核共享）
- **定位**：高性能大核，面向边缘服务器与计算密集型应用

## 与 X60 Core 的区别

| 特性 | X60 Core | X100 Core |
|------|----------|-----------|
| 定位 | 边缘泛 AI 应用 | 高性能 AI 计算 |
| 使用产品 | K1 | K3 |
| 主频 | 待补充 | 最高 2.4 GHz |
| L1 Cache | 32KB/核 | 64KB/核 |
| AI 扩展 | 部分型号可选 | 配合独立 A100 AI 核心 |

## 在 SpacemiT 产品中

- **K3**：8×X100 CPU + 8×A100 AI 核心，整机算力约 60 TOPS
- 与 A100 AI 核心协同工作，形成"同构融合计算架构"

## 相关词条

- [[X60-Core]] — K1 采用的核心
- [[A100-AI-Core]] — K3 的专用 AI 加速核心
- [[SoC]] — 系统级芯片概念
- [[../hardware/products/K3]] — 使用 X100 核心的产品

## 来源文档

- K3 Datasheet
- K3 User Manual（Chapter 8: CPU Subsystem）
