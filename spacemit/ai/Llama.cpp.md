---
tags: [ai, inference, llama-cpp, spacemit]
domain: ai
aliases: [Llama.cpp, llama.cpp, llama-cpp]
status: approved
created: 2026-11-16
updated: 2026-11-16
source: C:\Users\luojunlin\docs-ai\en\compute_stack\ai_compute_stack\llama.cpp.md
superseded_by:
---

# Llama.cpp

## 定义

**llama.cpp** 是一个轻量级大模型推理框架，主要为本地 **GGUF/GGML** 格式模型推理设计。它是完全开源的，并与上游社区保持同步。在 SpacemiT 的 RISC-V 平台上，CPU 推理路径可通过 **RVV**（RISC-V 向量扩展）和 **IME** 等硬件能力优化，并可选集成 **SMT/Melange** 视觉扩展以支持多模态负载。

## 为什么重要

- **量化友好**：原生支持 GGUF 格式，非常适合部署量化后的 LLM
- **边缘友好**：轻量、无需 GPU，适合 K1/K3 这类边缘 AI 芯片
- **快速验证**：通过多线程 CPU 推理快速验证模型可用性与吞吐
- **社区同步**：与上游保持一致，能持续获得新模型支持

## 典型用途

- 在边缘设备上运行 LLM 与多模态模型
- 部署量化模型（INT4/INT8 等）以压缩内存占用
- 作为模型落地前的可行性验证工具

## 与其他推理引擎的分工

| 引擎 | 定位 |
|---|---|
| **Llama.cpp** | 轻量大模型推理，主打 GGUF/GGML 量化模型 |
| [[vLLM]] | 高性能 LLM 服务框架，主打吞吐与并发 |
| [[ONNXRuntime]] | 通用模型推理（ONNX 格式） |

## 相关词条

- [[../ai/AI-Architecture|AI 架构]] — 整个 AI 软件栈的分层
- [[../atoms/X100-Core|X100 核心]] — K3 的 CPU 核心，提供 RVV 向量能力
- [[../atoms/Matrix-Extension|矩阵扩展]] — IME 指令集扩展

## 来源文档

- `docs-ai/en/compute_stack/ai_compute_stack/llama.cpp.md`
- 上游项目：<https://github.com/ggml-org/llama.cpp>
