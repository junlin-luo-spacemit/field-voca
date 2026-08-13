---
tags: [home, index, nvidia]
domain: all
date: 2026-08-08
---

# NVIDIA 词汇知识库

> 芯片行业文档工程师学习用知识库 · 来源：[docs.nvidia.com](https://docs.nvidia.com/)
> 进度追踪：[[PROGRESS]] · 总览：[[README]] · 架构规范：[[../ARCHITECTURE]]

---

## 🌳 不知道从哪看起？

**[[LEARNING_GUIDE]]** — 专为技术小白设计的学习路径，从零开始理解 NVIDIA GPU 技术体系。

---

## 核心原子词条

> 最重要的概念，每个都有独立深度页面。点击进入，图谱中以**橙色**显示。

### 计算单元
- [[SM]] — Streaming Multiprocessor，GPU 的基本计算单元
- [[warp]] — 线程束，32线程，GPU 的基本调度单位
- [[CUDA-Core]] — 标量计算单元（FP32 / INT32）
- [[Tensor-Core]] — 矩阵计算单元，AI 训练/推理算力核心

### 内存
- [[HBM]] — 高带宽内存，H100 带宽 3.35 TB/s
- [[Unified-Memory]] — CPU/GPU 统一内存地址空间

### 互连
- [[NVLink]] — GPU 高速互连，H100 间带宽 900 GB/s

### 推理框架
- [[TensorRT]] — NVIDIA 推理优化引擎

---

## 领域词汇表

| 领域 | 词汇表 | 笔记 | 词条数 | 状态 |
|------|--------|------|--------|------|
| CUDA 编程 | [[cuda/vocabulary\|CUDA 词汇]] | [[cuda/notes\|CUDA 笔记]] | ~245 | ✅ 完成 |
| TensorRT | [[tensorrt/vocabulary\|TensorRT 词汇]] | [[tensorrt/notes\|TensorRT 笔记]] | ~235 | ✅ 完成 |
| cuDNN | [[cudnn/vocabulary\|cuDNN 词汇]] | [[cudnn/notes\|cuDNN 笔记]] | ~120 | ✅ 完成 |
| NCCL | [[nccl/vocabulary\|NCCL 词汇]] | [[nccl/notes\|NCCL 笔记]] | ~80 | ✅ 完成 |
| Triton 推理服务 | [[triton/vocabulary\|Triton 词汇]] | [[triton/notes\|Triton 笔记]] | ~90 | ✅ 完成 |
| NIM | [[nim/vocabulary\|NIM 词汇]] | [[nim/notes\|NIM 笔记]] | ~65 | ✅ 完成 |
| NeMo | [[nemo/vocabulary\|NeMo 词汇]] | [[nemo/notes\|NeMo 笔记]] | ~70 | ✅ 完成 |
| Dynamo | [[dynamo/vocabulary\|Dynamo 词汇]] | [[dynamo/notes\|Dynamo 笔记]] | ~55 | ✅ 完成 |
| AI Enterprise | [[ai-enterprise/vocabulary\|AI Enterprise 词汇]] | [[ai-enterprise/notes\|AI Enterprise 笔记]] | ~50 | ✅ 完成 |
| AI 软件栈 | [[ai-software/vocabulary\|AI软件 词汇]] | [[ai-software/notes\|AI软件 笔记]] | ~20 | 🔄 整理中 |
| 硬件平台 | [[hardware/vocabulary\|硬件 词汇]] | [[hardware/notes\|硬件 笔记]] | ~52 | ✅ 完成 |
| 网络互连 | [[networking/vocabulary\|网络 词汇]] | [[networking/notes\|网络 笔记]] | ~47 | ✅ 完成 |
| Omniverse | [[omniverse/vocabulary\|Omniverse 词汇]] | [[omniverse/notes\|Omniverse 笔记]] | ~53 | ✅ 完成 |
| 全局词汇 | [[glossary/master_glossary\|Master Glossary]] | — | ~1180 合计 | — |

---

## 学习进度

| 领域 | 文档 | 状态 | 完成日期 |
|------|------|------|----------|
| CUDA | Programming Guide Part 1–4 + Best Practices | ✅ | 2026-07-02 |
| CUDA | Runtime API Reference + Compute Capabilities | ✅ | 2026-07-02 |
| TensorRT | Overview + Architecture + Developer Guide + API Reference | ✅ | 2026-07-21 |
| cuDNN | Overview + Core Concepts + Graph API + Developer Guide | ✅ | 2026-07-21 |
| NCCL | Overview + Usage | ✅ | 2026-07-21 |
| Triton | Overview + User Guide | ✅ | 2026-07-21 |
| NIM | Overview + Concepts | ✅ | 2026-07-21 |
| NeMo | Overview + Platform | ✅ | 2026-07-21 |
| Dynamo | Overview + Architecture | ✅ | 2026-07-21 |
| AI Enterprise | Overview + Software + Lifecycle | ✅ | 2026-07-21 |
| Hardware | DGX H100/H200 + Jetson + vGPU | ✅ | 2026-07-21 |
| Omniverse | Kit SDK + Extension Dev + OmniGraph + Isaac Sim | ✅ | 2026-07-21 |
| Networking | NVLink/NVSwitch + InfiniBand/RoCE + GPUDirect + SHARP + BlueField DPU/DOCA + 集合通信 | ✅ | 2026-07-22 |

---

## 快速参考

**关键数字**
- warp = **32线程** · Block最大 = **1024线程**
- H100 SM数 = **132** · HBM3带宽 = **3.35 TB/s** · FP16算力 = **989 TFLOPS**
- Cluster 需要 CC 9.0+（Hopper）

**执行层次**
`thread` → `warp(32)` → `block(max 1024)` → `cluster(CC 9.0+)` → `grid`

**内存速度排序**
`register` > `shared memory` > `L1` > `L2` > `HBM` > `NVLink` > `PCIe` > `system DRAM`
