---
tags: [atom, pmic, power, spacemit]
domain: hardware
aliases: [PMIC, Power Management IC, 电源管理芯片, 电源管理集成电路]
status: needs_review
created: 2026-08-06
updated: 2026-10-29
source:
superseded_by:
---

# PMIC（电源管理集成电路）

**中文译名**：电源管理集成电路 / 电源管理芯片

## 定义

PMIC（Power Management IC，电源管理集成电路）负责为芯片和模块提供稳压、功率序列管理、功耗监控与电源保护功能。

## 为什么重要

良好的电源管理能保证 SoC 在不同负载下稳定工作并优化总系统能效，尤其在多模块或集群场景中至关重要。

## 在 SpacemiT 中

- SpacemiT 提供 **P 系列**电源管理芯片：
  - [[../hardware/products/P1]] — 配套 K1/K3 使用
  - [[../hardware/products/P1S]] — P1 的变体型号
- P 系列是系统设计的重要组成部分，与 K 系列 AI CPU 配套

## 相关词条

- [[SoC]] — 系统级芯片
- [[../hardware/products/K1]] — 配套使用 P1/P1S
- [[../hardware/products/K3]] — 配套使用 P1/P1S
