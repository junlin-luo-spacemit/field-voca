---
tags: [ai, inference, localai, spacemit]
domain: ai
aliases: [localAI, LocalAI]
status: needs_review
created: 2026-08-06
updated: 2026-10-29
source:
superseded_by:
---

# LocalAI

## 定义

LocalAI 是本地 AI 推理框架，兼容 OpenAI API，支持在无云或受限环境中运行模型推理。

## 为什么重要

- **OpenAI 兼容**：API 格式与 OpenAI 一致，易于迁移
- **模型支持广**：支持 LLM、语音、图像生成等多种模型
- **离线运行**：完全本地化，无需互联网
- **轻量部署**：Docker 一键启动

## 支持的模型类型

- 大语言模型（Llama, GPT-J, Falcon）
- 文本生成图像（Stable Diffusion）
- 语音识别与合成（Whisper, Piper）
- 嵌入模型（Sentence Transformers）

## 使用方式

```bash
# Docker 启动
docker run -p 8080:8080 -v $PWD/models:/models quay.io/go-skynet/local-ai

# API 调用（与 OpenAI 兼容）
curl http://localhost:8080/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{"model": "llama-2-7b", "messages": [{"role": "user", "content": "Hello"}]}'
```

## 在 SpacemiT 场景

- 边缘设备或开发板上的轻量推理
- 离线测试与验证
- 替代云端 API，降低成本
- K3 本地大模型推理

## 与 Ollama 对比

| 特性 | LocalAI | Ollama |
|------|---------|--------|
| 定位 | 多模型通用 | LLM 专用 |
| API | OpenAI 兼容 | OpenAI 风格 |
| 模型类型 | LLM + 图像 + 语音 | LLM 为主 |
| 部署方式 | Docker + 配置 | 单一命令 |

## 相关词条

- [[Ollama]] — 另一种本地 LLM 平台
- [[LLM-SDK]] — SpacemiT 大语言模型 SDK
- [[LangChain]] — LLM 应用开发框架
- [[../hardware/products/K3]] — K3 适合运行 LocalAI

## 来源文档

- LocalAI 官网：https://localai.io/
- SpacemiT 文档中心：AI / Application Software Stack / LocalAI
