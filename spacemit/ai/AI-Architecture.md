---
tags: [ai, architecture, homogeneous-fusion, ime, matrix-extension, spacemit]
domain: ai
aliases: [AI架构, 同构融合, 智算核, AI CPU]
status: approved
created: 2026-10-29
updated: 2026-10-29
source: C:\Users\luojunlin\docs-ai\zh\architecture\
superseded_by:
---

# SpacemiT AI 架构与同构融合技术

## 核心创新：同构融合技术

### 定义

**同构融合技术**（Homogeneous Fusion Technology）是 SpacemiT 的核心 AI 架构创新：以标准 **RISC-V 指令集**为统一的软硬件接口，在 CPU 内集成 TensorCore，驱动 **Scalar 标量算力、Vector 向量算力和 Matrix AI 算力**，支持软件和 AI 模型同时在 RISC-V AI 核上运行，通过程序正常跳转实现软件和 AI 模型之间的事件和数据交互。

这种具有 AI 算力的 CPU 称为 **AI CPU** 或 **智算核**。

### 为什么重要：解决传统异构调度的复杂性

**传统 AI 加速器架构的挑战：**

传统 AI 加速器（GPGPU、NPU、TPU 等）在执行时需要主控 CPU 配合：
- **异构调度复杂**：CPU 与 XPU 之间需要额外的数据交互和同步
- **开发门槛高**：开发者需要理解异构编程模型和驱动管理
- **生态割裂**：不同厂商的 XPU 有不同的编程接口和工具链

```
传统架构：
┌─────────┐      ┌─────────┐
│   CPU   │◄────►│   XPU   │
│(调度器) │      │(AI加速) │
└─────────┘      └─────────┘
      ▲               ▲
      │               │
   复杂的异构调度和同步
```

**SpacemiT AI CPU 架构的优势：**

SpacemiT 在 CPU 内直接集成 TensorCore，保留 CPU 编程模型：
- **统一接口**：RISC-V 指令集统一驱动 Scalar/Vector/Matrix 算力
- **简化编程**：使用 Linux 线程即可驱动 AI 算力，无需异构调度
- **生态兼容**：便捷接入 RISC-V 开源生态，保留开源软件使用习惯
- **灵活计算**：兼具并行计算和逻辑计算能力，适配 MOE 模型推理

```
AI CPU 架构：
┌─────────────────────────────┐
│        AI CPU (智算核)       │
│  ┌────────┬────────┬────────┐│
│  │ Scalar │ Vector │ Matrix ││
│  │ (标量) │ (向量) │ (AI)   ││
│  └────────┴────────┴────────┘│
│   统一的 RISC-V 指令集接口   │
└─────────────────────────────┘
        ▲
        │
   Linux 线程直接驱动
```

## 三层算力架构

SpacemiT AI CPU 同时提供三种算力，通过 RISC-V 指令集统一驱动：

| 算力类型 | 指令集 | 用途 | 示例操作 |
|---------|--------|------|---------|
| **Scalar（标量）** | RISC-V 标准指令集 | 逻辑控制、分支判断、数据搬运 | if/else、循环、指针操作 |
| **Vector（向量）** | RISC-V Vector 1.0 (RVV) | SIMD 并行计算、数据预处理 | 图像处理、矩阵向量乘 |
| **Matrix（AI）** | RISC-V Matrix 扩展 (IME) | 深度学习、神经网络推理 | 矩阵乘、卷积、注意力机制 |

**三种算力的协同工作：**
1. **Scalar** 负责控制流、循环、条件判断
2. **Vector** 负责数据预处理、后处理、SIMD 加速
3. **Matrix** 负责神经网络核心计算（矩阵乘、卷积）

开发者无需手动管理三种算力的切换——编译器和运行时自动调度。

## K1 芯片架构实践

### K1 核心配置

**K1 芯片核心架构：**
- **4×X60 核心** — 通用 CPU 核心（Cluster 1）
- **4×A60 核心** — 智算核 / AI CPU（Cluster 0，带 IME 扩展）

**A60 核心技术规格：**
- **RISC-V Vector 位宽**：256-bit
- **Matrix 扩展**：IME 1.0（Integer Matrix Extension）
- **支持的 AI 模型**：0.5B-1B 参数量

### K1 A60 理论算力

**Matrix 算力（IME 扩展）：**
- 单核：0.5 TOPS (Int8)
- 4 核簇：**2 TOPS (Int8)**

**Vector 算力（RVV 1.0，256-bit）：**
- 单核：0.128 TOPS (Int8)，0.064 TOPS (FP16)，0.032 TOPS (FP32)
- 4 核簇：**0.5 TOPS (Int8)，0.25 TOPS (FP16)，0.13 TOPS (FP32)**

### K1 A60 实测性能

**测试工具**：[cpufp](https://github.com/pigirons/cpufp)（开源性能基准测试）

**单核实测（1×A60）：**

```
Instruction Set | Core Computation       | Peak Performance
----------------|------------------------|------------------
ime             | vmadot(s32,s8,s8)      | 511.53 GOPS
ime             | vmadotu(u32,u8,u8)     | 511.5 GOPS
ime             | vmadotus(s32,u8,s8)    | 511.53 GOPS
ime             | vmadotsu(s32,s8,u8)    | 511.51 GOPS
ime             | vmadotslide(s32,s8,s8) | 511.51 GOPS
vector          | vfmacc.vf(f16,f16,f16) | 66.722 GFLOPS
vector          | vfmacc.vv(f16,f16,f16) | 63.936 GFLOPS
vector          | vfmacc.vf(f32,f32,f32) | 33.36 GFLOPS
vector          | vfmacc.vv(f32,f32,f32) | 31.968 GFLOPS
```

**4 核簇实测（4×A60，Cluster 0）：**

```
Instruction Set | Core Computation       | Peak Performance
----------------|------------------------|------------------
ime             | vmadot(s32,s8,s8)      | 2.046 TOPS ✅
ime             | vmadotu(u32,u8,u8)     | 2.0462 TOPS
ime             | vmadotus(s32,u8,s8)    | 2.0461 TOPS
ime             | vmadotsu(s32,s8,u8)    | 2.0462 TOPS
ime             | vmadotslide(s32,s8,s8) | 2.0461 TOPS
vector          | vfmacc.vf(f16,f16,f16) | 266.88 GFLOPS ✅
vector          | vfmacc.vv(f16,f16,f16) | 255.75 GFLOPS
vector          | vfmacc.vf(f32,f32,f32) | 133.43 GFLOPS ✅
vector          | vfmacc.vv(f32,f32,f32) | 127.85 GFLOPS
```

**8 核全功率（8×A60，2 个 Cluster）：**

```
Instruction Set | Core Computation       | Peak Performance
----------------|------------------------|------------------
vector          | vfmacc.vf(f16,f16,f16) | 533.65 GFLOPS
vector          | vfmacc.vv(f16,f16,f16) | 511.45 GFLOPS
vector          | vfmacc.vf(f32,f32,f32) | 266.89 GFLOPS
vector          | vfmacc.vv(f32,f32,f32) | 255.75 GFLOPS
```

**性能总结：**
- ✅ **Matrix (IME)** 实测达到理论值：**2.046 TOPS (Int8)** per cluster
- ✅ **Vector (RVV)** 实测达到理论值：**266.9 GFLOPS (FP16)** per cluster
- ✅ 测试验证了 K1 A60 核心的三层算力架构性能

## K3 芯片架构升级

**K3 核心配置：**
- **8×X100 核心** — 高性能 CPU 核心（2.4GHz）
- **8×A100 核心** — 专用 AI 加速核心（**不是** AI CPU，是独立的 Matrix 专用核）

**关键区别：**
- **K1 A60**：AI CPU（CPU + Matrix 扩展指令），支持 Scalar/Vector/Matrix 三合一
- **K3 A100**：专用 AI 核心（独立硬件），约 **60 TOPS (Int8)** AI 算力

K3 的 A100 核心不是 "带 IME 扩展的 CPU"，而是独立的 Matrix 专用硬件加速器。

## SpacemiT AI 软件栈

基于同构融合架构，SpacemiT 构建了完整的 AI 软件栈：

### 第 1 层：端到端模型推理

| 工具 | 描述 | 链接 |
|-----|------|------|
| **ONNXRuntime** | SpacemiT ExecutionProvider 加速后端 | [[../ai/ONNXRuntime]] |
| **Llama.cpp** | 轻量大模型推理引擎，社区同步 | [[../ai/Llama.cpp]] |
| **vLLM** | 高性能 LLM 推理与服务框架 | [[../ai/vLLM]] |
| **XSlim** | 模型量化精简工具链 | [[../ai/XSlim]] |

### 第 2 层：AI 算子加速库

- TBD（开发中）

### 第 3 层：AI 编程语言

| 工具 | 描述 |
|-----|------|
| **Triton** | Python 交互的高性能 AI 算子编程 |

### 第 4 层：应用开发工具

| 工具 | 描述 | 链接 |
|-----|------|------|
| **Ollama** | 本地 LLM 部署工具 | [[Ollama]] |
| **LangChain** | LLM 应用开发框架 | [[LangChain]] |
| **LlamaIndex** | RAG 与数据框架 | [[LlamaIndex]] |
| **LocalAI** | 本地 AI 推理引擎 | [[LocalAI]] |
| **OpenWebUI** | Web UI for LLMs | [[OpenWebUI]] |
| **AI SDK** | SpacemiT AI 开发套件 | [[AI-SDK]] |
| **LLM SDK** | 大语言模型 SDK | [[LLM-SDK]] |
| **Speech SDK** | ASR/TTS 开发套件 | [[Speech-SDK]] |

## 应用场景

**同构融合技术特别适合：**

1. **边缘 AI 推理** — 无需复杂异构调度，功耗和延迟更优
2. **MOE 模型** — 兼具并行计算和逻辑计算，适合混合专家模型
3. **实时 AI 应用** — 软件与 AI 模型无缝切换，延迟可控
4. **嵌入式 AI** — Linux 线程直接驱动，开发门槛低
5. **开源生态集成** — RISC-V 原生支持，便捷接入开源工具链

**典型应用：**
- 本地大语言模型（Ollama + DeepSeek）
- 语音识别/合成（ASR/TTS）
- 计算机视觉（YOLO 目标检测）
- 机器人控制（ROS 2 + AI Robot SDK）
- 视觉语言模型（VLM 问答）

## 与传统架构对比

| 特性 | 传统异构架构（CPU + NPU） | SpacemiT 同构融合（AI CPU） |
|-----|------------------------|--------------------------|
| **编程模型** | 异构，需要驱动管理 | 同构，Linux 线程直接驱动 |
| **数据交互** | CPU ↔ NPU 需要显式搬运 | 共享内存，程序跳转即可 |
| **调度复杂度** | 高，需要复杂的异构调度 | 低，操作系统自动调度线程 |
| **开发门槛** | 高，需要学习专用 API | 低，标准 RISC-V + POSIX |
| **生态兼容** | 厂商锁定，工具链割裂 | RISC-V 开源生态通用 |
| **灵活性** | 仅并行计算 | 并行计算 + 逻辑计算 |
| **适用场景** | 纯 AI 推理 | AI + 控制逻辑混合 |

## 技术演进路线

| 代际 | 芯片 | AI 核心类型 | 算力 | 架构特点 |
|-----|------|-----------|------|---------|
| **第一代** | K1 | A60 (AI CPU) | 2 TOPS | CPU + IME 扩展指令 |
| **第二代** | K3 | A100 (专用核) | 60 TOPS | 独立 Matrix 硬件 |

**技术发展方向：**
1. K1：验证同构融合架构可行性
2. K3：提升 AI 算力至 60 TOPS
3. 未来：更高算力、更复杂模型支持（30B+ 参数）

## 相关词条

### 硬件架构
- [[../atoms/X60-Core]] — K1 通用 CPU 核心
- [[../atoms/X100-Core]] — K3 高性能 CPU 核心
- [[../atoms/A100-AI-Core]] — K3 专用 AI 加速核心
- [[../atoms/Matrix-Extension]] — RISC-V Matrix 扩展指令集

### 芯片产品
- [[../hardware/products/K1]] — K1 芯片（第一代 AI CPU）
- [[../hardware/products/K3]] — K3 芯片（60 TOPS AI 算力）

### AI 软件栈
- [[ONNXRuntime]] — SpacemiT ExecutionProvider
- [[Ollama]] — 本地 LLM 部署
- [[LangChain]] — LLM 应用框架
- [[AI-Platform]] — AI 平台概览

### 系统支持
- [[../software/Bianbu]] — 官方 Linux 发行版
- [[../software/ROS2]] — AI Robot SDK

## 来源文档

- SpacemiT AI 架构文档：C:\Users\luojunlin\docs-ai\zh\architecture\
- 设计理念：docs-ai/zh/architecture/concept.md
- IME 扩展：docs-ai/zh/architecture/ime_extension.md
- 计算软件栈：docs-ai/zh/compute_stack/ai_compute_stack.md
- cpufp 基准测试：https://github.com/pigirons/cpufp
