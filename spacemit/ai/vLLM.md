---
tags: [ai, inference, vllm, spacemit]
domain: ai
aliases: [vLLM]
status: approved
created: 2026-11-16
updated: 2026-11-16
source: C:\Users\luojunlin\docs-ai\en\compute_stack\ai_compute_stack\vllm.md
superseded_by:
---

# vLLM

## 定义

**vLLM** 是一个面向大语言模型（LLM）推理与服务的高性能框架，以高吞吐量和内存效率著称。SpacemiT 正在适配 vLLM 以支持 RISC-V 平台上的 LLM 原生部署。

> ⚠️ SpacemiT 的 vLLM 适配仍在开发中（官方文档标注：*vLLM for SpacemiT is still under construction*）。

## 为什么重要

- **高吞吐**：通过 PagedAttention 技术大幅提升并发推理效率
- **服务化**：内置 OpenAI 兼容的 HTTP 服务接口
- **主流生态**：是当前 LLM 生产部署的主流方案之一

## 与其他推理引擎的分工

| 引擎 | 定位 |
|---|---|
| **vLLM** | 高并发 LLM 服务，面向数据中心/集群部署 |
| [[Llama.cpp]] | 轻量本地推理，面向边缘设备 |
| [[ONNXRuntime]] | 通用 ONNX 模型推理 |

## 典型场景

- RV2768 集群上的大规模 LLM 推理服务（100B+ 模型）
- 对吞吐量和并发要求高的 API 服务

## 相关词条

- [[../ai/AI-Architecture|AI 架构]] — 整个 AI 软件栈的分层
- [[../hardware/products/RV2768|RV2768]] — K3 集群，2880 TOPS

## 来源文档

- `docs-ai/en/compute_stack/ai_compute_stack/vllm.md`
- 上游项目：<https://github.com/vllm-project/vllm>
