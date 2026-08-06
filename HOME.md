---
tags: [home, index, nvidia]
date: 2026-07-22
---

# 芯片行业文档词汇知识库

> 芯片行业文档工程师学习用知识库。来源：官方技术文档

---

## 知识域

| 域 | 主页 | 来源 | 状态 |
|----|------|------|------|
| NVIDIA | [[nvidia/HOME\|NVIDIA 知识库]] | [docs.nvidia.com](https://docs.nvidia.com/) | ✅ 活跃 |
| RISC-V | [[riscv/HOME\|RISC-V 知识库]] | [riscv.org](https://riscv.org/) | 🔄 建立中 |

---

## 领域词汇表

| 领域 | 词汇表 | 笔记 | 状态 |
|------|--------|------|------|
| CUDA 编程 | [[nvidia/cuda/vocabulary\|CUDA 词汇]] | [[nvidia/cuda/notes\|CUDA 笔记]] | ✅ Part 1 完成 |
| AI 软件栈 | [[nvidia/ai-software/vocabulary\|AI软件 词汇]] | [[nvidia/ai-software/notes\|AI软件 笔记]] | 🔄 整理中 |
| 硬件平台 | [[nvidia/hardware/vocabulary\|硬件 词汇]] | [[nvidia/hardware/notes\|硬件 笔记]] | 🔄 整理中 |
| 网络互连 | [[nvidia/networking/vocabulary\|网络 词汇]] | [[nvidia/networking/notes\|网络 笔记]] | 🔄 整理中 |
| Omniverse | [[nvidia/omniverse/vocabulary\|Omniverse 词汇]] | — | 🔄 整理中 |
| 全局词汇 | [[nvidia/glossary/master_glossary\|Master Glossary]] | — | 🔄 整理中 |

---

## 核心原子词条

> 最重要的概念，每个都有独立深度页面。

### 硬件 / 计算单元

- [[atoms/SM|SM — Streaming Multiprocessor 流多处理器]]
- [[atoms/warp|warp — 线程束（32线程执行单位）]]
- [[atoms/CUDA-Core|CUDA Core — 标量计算单元]]
- [[atoms/Tensor-Core|Tensor Core — 矩阵计算单元]]

### 内存

- [[atoms/HBM|HBM — 高带宽内存]]
- [[atoms/Unified-Memory|Unified Memory — 统一内存]]

### 互连

- [[atoms/NVLink|NVLink — GPU高速互连]]

### 软件 / 推理

- [[atoms/TensorRT|TensorRT — 推理优化引擎]]

---

## 学习进度

| 文档 | 来源 | 状态 | 更新 |
|------|------|------|------|
| CUDA Programming Guide Part 1 | [官方链接](https://docs.nvidia.com/cuda/cuda-programming-guide/) | ✅ 完成 | 2026-06-29 |
| CUDA Programming Guide Part 2–5 | 同上 | ⬜ 待学习 | — |
| TensorRT Developer Guide | [官方链接](https://docs.nvidia.com/deeplearning/tensorrt/) | ⬜ 待学习 | — |
| NeMo Framework | [官方链接](https://docs.nvidia.com/nemo-framework/) | ⬜ 待学习 | — |
| NVLink / NVSwitch | [官方链接](https://docs.nvidia.com/networking/) | ⬜ 待学习 | — |

---

## 快速参考

**关键数字**：warp = 32线程 · Block最大 = 1024线程 · H100 SM数 = 132 · HBM3带宽 = 3.35 TB/s · FP16算力 = 989 TFLOPS

**计算层次**：thread → warp(32) → block(max 1024) → cluster(CC 9.0+) → grid

**内存速度**：register > shared memory > L1 > L2 > HBM > NVLink > PCIe > system DRAM
