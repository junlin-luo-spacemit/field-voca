---
tags: [study, index, spacemit]
domain: all
status: approved
created: 2026-11-16
updated: 2026-11-16
---

# SpacemiT 学习材料索引

> 所有学习材料统一放在 `spacemit/study/` 目录下。
> 配套测验在 `spacemit/study/quiz/` 目录下。
> 同事可以直接从这里开始，按阶段顺序学习。

---

## 学习路径

```
Day 1 → Day 2 → Day 3（计划中）
  ↓         ↓
产品全景   软件栈
  ↓         ↓
测验      测验（待建）
```

---

## 材料清单

### Day 1：SpacemiT 产品全景（硬件）

| 文件 | 内容 | 预计时间 |
|------|------|---------|
| [day1-products.md](./day1-products.md) | 两大芯片系列、三颗芯片规格、产品线、RV2768、同构融合技术、选型决策树 | 45 分钟阅读 |
| [quiz/day1-recap.html](./quiz/day1-recap.html) | 22 题互动选择题，点击即时判分，分组统计 | 20 分钟 |

**学完 Day 1 你应该能回答**：
- RV2768 是由多少颗什么芯片组成的？
- K1 的 8 个核心怎么分配？
- G5201 出厂有没有 GPU 卡？
- 同构融合技术是谁的技术？

---

### Day 2：软件栈基础

| 文件 | 内容 | 预计时间 |
|------|------|---------|
| [day2-boot-chain.md](./day2-boot-chain.md) | 启动链六层、OpenSBI 解释、刷机 vs 启动、K3 vs K1 差异 | 30 分钟阅读 |
| day2-operating-systems.md | 四种 OS 的定位与选型（Bianbu/Buildroot/OpenWrt/OpenHarmony） | 待建 |
| day2-sdk-toolchain.md | SDK/BSP 结构、为什么需要交叉编译 | 待建 |
| quiz/day2-recap.html | Day 2 互动测验 | 待建 |

**学完 Day 2 你应该能回答**：
- 通电到程序运行经过哪六层固件？
- BROM 和 FSBL 有什么区别？
- OpenSBI 是什么，x86 有没有？
- 刷机和启动有什么区别？

---

### Day 3（计划中）：AI 栈

| 文件 | 内容 |
|------|------|
| day3-ai-stack.md | 推理引擎（ONNXRuntime / Llama.cpp / vLLM）、量化（XSlim）、应用工具 | 待建 |
| quiz/day3-recap.html | Day 3 测验 | 待建 |

---

## 给同事的快速上手建议

1. **先读 [day1-products.md](./day1-products.md)**，大约 45 分钟。建立全局认知。
2. **做 [day1-recap.html](./quiz/day1-recap.html) 测验**，确认理解。85 分（19/22）以上再继续。
3. **读 [day2-boot-chain.md](./day2-boot-chain.md)**，理解固件链。
4. 遇到不确定的技术术语，查 [[../HOME|HOME.md]] 里的"5 分钟快速理解"部分。
5. 深入某个概念，查 [[../atoms/README|原子词条]]。

---

## 参考文档来源

所有材料均依据以下官方文档库核实：

| 文档库 | 路径 | 覆盖内容 |
|--------|------|---------|
| docs-chip | `C:\Users\luojunlin\docs-chip` | K1/K3 芯片规格 |
| docs-product | `C:\Users\luojunlin\docs-product` | 产品简介 |
| docs-bianbu | `C:\Users\luojunlin\docs-bianbu` | Bianbu OS |
| docs-buildroot | `C:\Users\luojunlin\docs-buildroot` | Buildroot SDK、启动链 |
| docs-ai | `C:\Users\luojunlin\docs-ai` | AI 软件栈 |
| docs-tool | `C:\Users\luojunlin\docs-tool` | 工具链、JTAG、Trace |
