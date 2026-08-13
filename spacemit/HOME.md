---
tags: [home, index, spacemit]
domain: all
date: 2026-08-08
---

# SpacemiT 文档中心词汇库

> 来源：SpacemiT 文档中心（中文 / English）
> 进度追踪：[[PROGRESS]] · 总览：[[README]] · 架构规范：[[../ARCHITECTURE]]

---

## 🌳 不知道从哪看起？先看这个

**[[概念地图]]** — 一张图讲清楚 SpacemiT 芯片与文档体系的整体框架，5 分钟建立全局认知。

**[[官方文档结构地图]]** — 1:1 对照 SpacemiT 官方文档中心的 Hardware/Software/AI/Tools 四大分类，标注本知识库已覆盖/待补充的具体条目，用作构建与补全的"骨架"。

**[[跨厂商术语对照表]]** — 学习 NVIDIA / RISC-V / AMD 等其它厂商术语时，用来对照"这个概念在 SpacemiT 里叫什么"的翻译字典，持续更新中。

---

## 核心原子词条

> 跨领域的核心概念，每个都有独立深度页面。详见 [[atoms/README]]。

### 硬件核心
- [[atoms/SoC]] — 系统级芯片，SpacemiT 产品的核心形态
- [[atoms/X60-Core]] — CPU 核心架构
- [[atoms/Matrix-Extension]] — SpacemiT 特色矩阵指令集扩展

### 内存与 I/O
- [[atoms/HBM]] — 高带宽内存
- [[atoms/IOMMU]] — I/O 内存管理单元

### 电源管理
- [[atoms/PMIC]] — 电源管理集成电路

---

## 领域词汇表

| 领域 | 词汇表 | 笔记 | 状态 |
|------|--------|------|------|
| 硬件 | [[hardware/vocabulary|硬件词汇]] | [[hardware/notes|硬件笔记]] | ✅ 已重构 |
| 软件 | [[software/vocabulary|软件词汇]] | [[software/notes|软件笔记]] | ✅ 已重构 |
| AI | [[ai/vocabulary|AI 词汇]] | [[ai/notes|AI 笔记]] | ✅ 已重构 |
| 工具 | [[tools/vocabulary|工具词汇]] | [[tools/notes|工具笔记]] | ✅ 已重构 |
| 云 | [[cloud/vocabulary|云词汇]] | [[cloud/notes|云笔记]] | ✅ 已重构 |

---

## 快速导航

**硬件产品**：
- K 系列芯片：[[hardware/products/K1]] | [[hardware/products/K3]]
- P 系列芯片：[[hardware/products/P1]] | [[hardware/products/P1S]]
- MUSE 系列：[[hardware/products/K1-MUSE-Pi]] | [[hardware/products/K1-MUSE-BOOK]]
- 集群服务器：[[hardware/products/RV2768]]

**软件平台**：
- 操作系统：[[software/Bianbu]] | [[software/OpenWrt]] | [[software/Buildroot]]
- 机器人：[[software/ROS2]]

**AI 平台**：
- SDK：[[ai/AI-SDK]] | [[ai/LLM-SDK]]
- 框架：[[ai/LangChain]] | [[ai/Ollama]] | [[ai/LlamaIndex]]

**开发工具**：
- 工具链：[[tools/Cross-Compilation-Toolchain]]
- 调试：[[tools/JTAG-Debugging]] | [[tools/Trace-Tool]]
- IDE：[[tools/SpacemiT-Studio]]

---

## 参考入口

- [SpacemiT 文档中心（中文）](https://www.spacemit.com/community/document?lang=zh)
- [SpacemiT 文档中心（English）](https://www.spacemit.com/community/document?lang=en)
