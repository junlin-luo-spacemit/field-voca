---
tags: [atoms, index, spacemit]
domain: all
status: needs_review
created: 2026-08-08
updated: 2026-10-29
---

# 原子词条索引（Atoms Index）

## 什么是原子词条（Atoms）？

原子词条是 SpacemiT 知识库中**最核心**的技术概念，具有以下特征：
1. **跨领域引用**：在多个领域（硬件/软件/AI/工具）中被频繁引用
2. **基础性**：是理解其他概念的前置知识
3. **SpacemiT 特色**：SpacemiT 独有或重点强调的技术
4. **深度解析**：需要独立页面进行完整阐述

**其他概念请查看各领域的 vocabulary.md 和详细页**。

---

## 核心原子词条列表

### 硬件核心
- [SoC](SoC.md) — 系统级芯片，SpacemiT 产品的核心形态
- [X60 Core](X60-Core.md) — K1 的 CPU 核心架构（8×X60，双簇，Cluster0 含 AI 扩展）
- [X100 Core](X100-Core.md) — K3 的高性能 CPU 核心（8×X100 @2.4GHz）
- [A100 AI Core](A100-AI-Core.md) — K3 的专用 AI 加速核心（8×A100，约 60 TOPS）
- [Matrix Extension](Matrix-Extension.md) — SpacemiT 的矩阵加速技术（K1 指令集扩展 + K3 A100 硬件核心）

### 内存与 I/O
- [HBM](HBM.md) — 高带宽内存（对比参考，SpacemiT 采用 LPDDR）
- [IOMMU](IOMMU.md) — I/O 内存管理单元，虚拟化关键技术

### 电源管理
- [PMIC](PMIC.md) — 电源管理集成电路（P 系列）

---

## 其他概念索引

- **硬件产品与架构** → [[../hardware/vocabulary]]
- **软件与 SDK** → [[../software/vocabulary]]
- **AI 平台与框架** → [[../ai/vocabulary]]
- **开发工具** → [[../tools/vocabulary]]
- **云服务** → [[../cloud/vocabulary]]
