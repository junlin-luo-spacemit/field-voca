---
tags: [spacemit, hardware, notes]
domain: hardware
date: 2026-08-08
---

# 硬件平台学习笔记

## K1 vs K3 核心对比

| 特性 | K1 | K3 |
|------|----|----|
| CPU 核心 | 待补充 | 8×X100 + 8×A100 |
| AI 算力 | 待补充 | 60 TOPS |
| 内存支持 | 待补充 | 待补充 |
| 目标场景 | 入门/教育/边缘 | 高性能 AI 计算 |
| 文档完整度 | ✅ 完整（Datasheet + User Manual + 硬件设计指南） | ✅ 完整（Datasheet + User Manual + 硬件设计指南） |

## 产品线分类

### 芯片（Chips）
- **K1, K3** — AI CPU，RISC-V 架构
- **P1, P1S** — 电源管理芯片

### 开发板（Dev Boards）
- **K1-MUSE-Pi, K1-MUSE-Pi-Pro** — 单板计算机
- **K3-Pico-ITX** — Pico-ITX 标准主板
- **K3-CoM260-Kit** — 计算模块套件

### 终端设备（End Devices）
- **K1-MUSE-BOOK** — 笔记本形态
- **K1-MUSE-PAPER** — 电子纸设备
- **K1-MUSE-BOX** — 盒子形态

### 集群/服务器（Clusters & Servers）
- **K1-MUSE-Shelf** — K1 机架式服务器
- **K3-Shelf** — K3 机架式服务器
- **RV2768** — RISC-V 集群服务器
- **N10-N48-Cluster** — 集群节点配置

## 硬件设计要点

（从官方硬件设计指南提取的关键点，待补充）

### 电源设计
- PMIC 配置要点
- 电源域划分
- 功耗预算

### 散热设计
- TDP 规格
- 散热方案推荐
- 温度监控

### PCB 布局
- 高速信号布线
- 电源层设计
- EMI 考量

## 常见问题（FAQ）

（从官方 FAQ 提取，待补充）

## 参考资料

- K1 Datasheet
- K1 User Manual（16章）
- K1 Hardware Design Guide
- K3 Datasheet
- K3 User Manual（17章）
- K3 Hardware Design Guide
