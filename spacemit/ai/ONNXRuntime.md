---
tags: [ai, inference, onnxruntime, spacemit]
domain: ai
aliases: [ONNXRuntime, SpacemiT-ONNXRuntime, ONNX Runtime]
status: approved
created: 2026-11-16
updated: 2026-11-16
source: C:\Users\luojunlin\docs-ai\en\compute_stack\ai_compute_stack\onnxruntime.md
superseded_by:
---

# ONNXRuntime

## 定义

**SpacemiT-ONNXRuntime** 是把社区版 [ONNXRuntime](https://github.com/microsoft/onnxruntime) 的基础推理库与 **SpacemiT ExecutionProvider（`SpaceMITExecutionProvider`）** 加速后端集成后的推理引擎。整体架构保持解耦，因此用法与社区版几乎完全一致——已有的 ONNX 模型和脚本通常无需改动。

## 为什么重要

- **低迁移成本**：API 与社区版一致，现有 ONNX 工作流可直接复用
- **硬件加速**：通过 SpacemiT ExecutionProvider 调用芯片的 AI 算力（A60 / A100）
- **生态兼容**：ONNX 是模型交换的事实标准，可对接 PyTorch、TensorFlow 导出的模型

## 平台支持

| 平台 / 系统 | 支持 |
|---|---|
| K1 Buildroot | ✅ |
| K1 Bianbu LXQT/GNOME | ✅ |
| K1 OpenHarmony 5.0 | ❌ |
| K3 Buildroot | ✅ |
| K3 OpenHarmony | ❌ |
| K3 Bianbu LXQT/GNOME | ✅ |

## 与其他推理引擎的分工

| 引擎 | 定位 |
|---|---|
| **ONNXRuntime** | 通用模型推理（ONNX 格式），覆盖面最广 |
| [[Llama.cpp]] | 轻量大模型推理，主打 GGUF/GGML 量化模型 |
| [[vLLM]] | 高性能 LLM 服务框架，主打吞吐与并发 |

## 相关词条

- [[../ai/AI-Architecture|AI 架构]] — 整个 AI 软件栈的分层
- [[Llama.cpp]] — 轻量大模型推理引擎
- [[XSlim]] — 配套的量化工具链

## 来源文档

- `docs-ai/en/compute_stack/ai_compute_stack/onnxruntime.md`
