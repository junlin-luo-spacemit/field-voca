---
tags: [ai, inference, ollama, spacemit]
domain: ai
aliases: [Ollama]
status: needs_review
created: 2026-08-06
updated: 2026-10-29
source:
superseded_by:
---

# Ollama

## 定义

Ollama 是本地大语言模型运行平台，提供简单的命令行接口，便于在本地机器上管理与运行 LLM 实例。

## 为什么重要

- **本地部署**：无需云端，数据不出设备
- **简单易用**：一条命令启动模型
- **模型管理**：类似 Docker，pull/run/rm 模型
- **API 兼容**：兼容 OpenAI API 格式

## 使用方式

```bash
# 拉取模型
ollama pull llama2

# 运行模型
ollama run llama2

# API 服务
ollama serve
```

## 支持的模型

- Llama 2 / Llama 3
- Mistral / Mixtral
- Phi / CodeLlama
- Qwen / ChatGLM

## 在 SpacemiT 场景

- 边缘设备本地推理
- 开发环境测试
- 无网络环境部署
- 配合 K3 进行 7B-13B 参数模型推理

## 与 LocalAI 对比

| 特性 | Ollama | LocalAI |
|------|--------|---------|
| 定位 | LLM 专用 | 多模型通用 |
| 易用性 | 极简 | 中等 |
| API | OpenAI 风格 | OpenAI 兼容 |
| 模型管理 | Docker 风格 | 配置文件 |

## 相关词条

- [[localAI]] — 另一种本地推理框架
- [[LLM-SDK]] — SpacemiT 大语言模型 SDK
- [[LangChain]] — LLM 应用开发框架
- [[../hardware/products/K3]] — K3 适合运行 Ollama

## 来源文档

- Ollama 官网：https://ollama.ai/
- SpacemiT 文档中心：AI / Application Software Stack / Ollama
