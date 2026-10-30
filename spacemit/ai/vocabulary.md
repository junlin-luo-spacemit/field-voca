---
tags: [spacemit, ai, vocabulary]
domain: ai
date: 2026-08-08
source: C:\Users\luojunlin\docs-ai\zh\
---

# AI 平台词汇表

> 涵盖：AI 平台、SDK、应用框架
> 来源：SpacemiT 文档中心 AI 分类

---

## AI 平台（AI Platform）

### SpacemiT RISC-V AI Computing Platform / SpacemiT RISC-V AI 计算平台
SpacemiT 的 AI 计算解决方案，整合硬件、软件栈和应用框架。详见 [[AI-Platform]]。

### AI Accelerator / AI 加速器
专用硬件单元，用于加速神经网络推理和训练。

---

## AI SDK

### AI SDK / AI 软件开发套件
面向模型部署与推理的开发工具集。详见 [[AI-SDK]]。

### LLM SDK / 大语言模型 SDK
专门用于部署和运行大语言模型的工具集。详见 [[LLM-SDK]]。

---

## 应用框架（Application Frameworks）

### LangChain / LangChain
构建基于 LLM 的多模态应用的开发框架。详见 [[LangChain]]。

### LlamaIndex / LlamaIndex
文档索引与检索工具，用于增强 LLM 应用的上下文能力。详见 [[LlamaIndex]]。

---

## 模型运行时（Model Runtime）

### Ollama / Ollama
本地大模型运行平台，支持多种开源模型。详见 [[Ollama]]。

### LocalAI / LocalAI
本地推理框架，兼容 OpenAI API。详见 [[localAI]]。

---

## 模型优化（Model Optimization）

### Model Quantization / 模型量化
降低模型精度（如 FP16 → INT8）以减少内存和提升速度。

### Model Pruning / 模型剪枝
移除神经网络中冗余的权重和连接。

### Knowledge Distillation / 知识蒸馏
用大模型训练小模型，保持性能同时降低规模。

---

## 推理引擎（Inference Engine）

### Inference / 推理
使用训练好的模型进行预测的过程。

### Batch Inference / 批量推理
一次处理多个输入样本，提升吞吐量。

### Real-time Inference / 实时推理
低延迟的单样本推理，适用于交互场景。

---

## AI 计算架构（Compute Architecture）

### Matrix Extension / 矩阵扩展
SpacemiT 的矩阵加速指令集，详见 [[../atoms/Matrix-Extension]]。

### Compute Software Stack / 计算软件栈
从底层硬件抽象到高层 AI 框架的软件层次。

---

## 参考资料

- [SpacemiT AI 文档](https://www.spacemit.com/community/document?lang=zh)
- AI SDK 使用指南
- 模型部署最佳实践
