---
tags: [study, m1, product]
domain: edge_robotics
aliases: [产品系列, 命名规律, Product Series, Naming Convention]
status: needs_review
created: 2026-11-18
updated: 2026-11-25
---

# M1 · 第一课：产品系列与命名规律

SpacemiT 的芯片分三个系列。命名有规律，看名字就知道属于哪一类。

## 一、芯片的三个系列

| 系列 | 名称 | 是什么 | 现有型号 |
|---|---|---|---|
| **K** | K 系列 AI CPU 芯片（Key Stone） | 带 AI 算力的 CPU | K1、K3 |
| **P** | P 系列电源芯片（Power Stone） | 电源管理芯片（PMIC） | P1、P3 |
| **V** | V 系列服务器芯片（Vital Stone） | 服务器级 CPU | V100 |

三个系列的定位不同。K 系列面向端侧和边缘，V 系列面向数据中心，P 系列是配套的电源芯片。

## 二、K 系列的命名规律

K 系列用一位奇数编号：**K1 → K3 → K5 → K7 → K9**。

现有的是 K1 和 K3。

K5、K7、K9 是后续规划的型号，目前没有产品资料。**不要在没有官方资料的情况下描述它们的规格。**

编号越大，通常代表越新的一代。但具体差异要看各型号的官方文档，不能只凭编号推断。

## 三、V 系列的命名规律

V 系列用三位整数：**V100 → V200 → V300**。

现有的是 V100。

V200、V300 同样是后续规划，目前没有产品资料。

V100 是系列名，也是芯片名。具体整机产品另有型号，比如 C2201、S5236、G5201。

## 四、产品型号和芯片型号是两回事

这一点容易混。

- **芯片型号**：K1、K3、V100、P1、P3
- **产品型号**：K1 MUSE Pi、K3 Pico-ITX、C2201、G5201

一颗芯片可以做成很多产品。

比如 K3 芯片，做成了 Pico-ITX、CoM260、Shelf、RV2768 四种产品。

反过来，C2201 和 S5236 都用 V100 芯片，但一个是计算服务器，一个是存储服务器。

## 五、K1 和 K3 的能力差别在哪

同样是 K 系列，两代的内核构成不同。

| 项 | K1 | K3 |
|---|---|---|
| 通用核 | 4 个 X60 核 | 8 个 X100 核，最高 2.4 GHz |
| AI 核 | 4 个 A60 核 | 8 个 A100 核 |
| 核总数 | 8 | 16 |
| AI 算力 | [2 TOPS](GLOSSARY.md#tops)（@INT8） | 60 TOPS（@INT4 sparse） |
| 内存接口 | 32 位 LPDDR4X | 双通道 LPDDR5，6400 MT/s |

### K1：通用核加 AI 核

K1 的 8 个核分成两[簇](GLOSSARY.md#簇)，两[簇](GLOSSARY.md#簇)不对称：

- [簇](GLOSSARY.md#簇) 0：4 个 **A60** 核，负责 AI 计算
- [簇](GLOSSARY.md#簇) 1：4 个 **X60** 核，负责通用计算

数据手册 2.1 节的原文是：

> - [簇](GLOSSARY.md#簇) 0：包含 4 个 X60 核，集成 2 TOPS AI 算力扩展单元
> - [簇](GLOSSARY.md#簇) 1：包含 4 个 X60 核，不带 AI 加速能力

**手册在这里只写了 X60 这个名字。** 但[簇](GLOSSARY.md#簇) 0 明确「集成 AI 算力扩展单元」，实际上就是 A60 智算核所在的位置。对外讲的时候，[簇](GLOSSARY.md#簇) 0 的核按 A60 称呼、[簇](GLOSSARY.md#簇) 1 的核按 X60 称呼，是团队内部统一的口径。

所以 K1 可以理解为：**4 个 X60 加 4 个 A60，共 8 核。**

> ⚠️ **对外宣传口径**：K1 对外只讲**8 核**，不细分 A60 和 X60 的数量。内部理解 4+4 配置即可，对外统一说「8 核处理器」。

### K3：两种核各 8 个

K3 是 8 个 X100 加 8 个 A100，共 16 核。

产品资料称「计算核和智算核」——计算核是 X100，智算核是 A100。

**和 K1 相比，K3 的变化是 AI 核从半簇扩到 8 个，并且换了新架构。**

### 名称对照

| 代际 | 通用核算 | AI 核算 | AI 指令扩展 |
|---|---|---|---|
| K1 | X60 | A60 | [IME](GLOSSARY.md#ime) 早期版本 |
| K3 | X100 | A100 | IME（VLEN 1024） |

命名规律是一致的：**X + 数字是通用核，A + 数字是 AI 核**，数字越大越新一代。

## 六、V100 没有 AI 核

V100 是服务器级 CPU，本身不含 AI 核。

AI 能力来自外接的 PCIe 加速卡。G5201 最多支持 8 张风冷加速卡。

所以 V100 整机的 AI 算力，取决于客户插几张、插什么卡。

## 快速自测

1. K、P、V 三个系列分别是什么芯片？
2. K 系列和 V 系列的编号规律是什么？
3. K1 的 8 个核里，X60 和 A60 各有几个？
4. V100 整机的 AI 算力从哪来？

答不上来就回看对应小节。

## 参考来源

| 小节 | 库 | 文件路径 | 章节 |
|---|---|---|---|
| K/P/V 三个芯片系列 | docs-chip | `key_stone/index.md`、`power_stone/index.md`、`vital_stone/index.md` | 全文 |
| K1 产品规格 | docs-product | `k1_series/k1_muse_pi/root_overview.md` | 简介、产品规格 |
| K1 簇架构与 X60 | docs-chip | `key_stone/k1/k1_docs/k1_ds.md` | 1.2 主要特性、2.1 CPU 子系统 |
| MUSE Book 规格 | docs-product | `k1_series/k1_muse_book/root_overview.md` | 特性 |
| K3 Pico 规格 | docs-product | `k3_series/k3_pico/root_overview.md` | 产品规格 |
| K3 CoM260 规格 | docs-product | `k3_series/k3_com260/root_overview.md` | 产品规格 |
| K3 芯片核数 | docs-chip | `key_stone/k3/k3_docs/k3_ds.md` | 2.1.1、2.1.2 |
| V100 G5201 | docs-product | `v100_series/v100_g5201/root_overview.md` | 概述、特性 |
| V100 产品清单 | docs-product | `v100_series/index.md` | 全文 |
| P 系列芯片 | docs-chip | `power_stone/p1/p1_docs/p1_ds.md`、`power_stone/p3/p3_docs/p3_ds.md` | 数据手册 |

> **注 1**：K5、K7、K9、V200、V300 是后续规划型号，本课不给出规格。早期产品资料中 K1 芯片曾写作 M1，对外统一使用 K1。
>
> **注 2**：命名规律（K 系列一位奇数、V 系列三位整数）已确认可对外讲。
>
> **注 3 — 关于 K1 的 A60 称呼**：K1 数据手册 V5.1 的 2.1 节只写「簇 0：4 个 X60 核，集成 2 TOPS AI 算力扩展单元」，没有出现 A60 这个型号字面。本课按团队口径把簇 0 的 4 个核称 A60、簇 1 的 4 个核称 X60，与 `spacemit/HOME.md`、`ai/AI-Architecture.md` 等既有笔记一致。若对外材料需要严格引用手册原文，用「簇 0 集成 AI 算力扩展单元」的表述更稳妥。
