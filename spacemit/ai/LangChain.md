---
tags: [ai, framework, langchain, spacemit]
domain: ai
aliases: [LangChain]
status: needs_review
created: 2026-08-06
updated: 2026-10-29
source:
superseded_by:
---

# LangChain

## 定义

LangChain 是用于构建基于大语言模型（LLM）的多模态应用的开发框架，提供链式调用、工具集成与记忆管理能力。

## 为什么重要

- **快速开发**：模块化组件，快速搭建 LLM 应用原型
- **工具集成**：连接 LLM、检索系统、API、数据库
- **链式编排**：多步骤推理与决策
- **记忆管理**：对话历史、上下文管理

## 主要功能

1. **Chains（链）**
   - 多步骤任务编排
   - LLM 串联调用

2. **Agents（智能体）**
   - 自主决策与工具选择
   - 目标导向的任务执行

3. **Memory（记忆）**
   - 对话历史管理
   - 上下文保持

4. **Retrieval（检索）**
   - 向量数据库集成
   - 文档检索增强生成（RAG）

## 使用场景

- 文档问答系统
- 智能客服与助手
- 数据分析与报告生成
- 多步骤推理任务

## 与 LlamaIndex 的关系

- **LangChain**：通用 LLM 应用框架，重在编排与集成
- **LlamaIndex**：专注文档索引与检索，常与 LangChain 配合使用

## 在 SpacemiT 中

- 可与本地 LLM（Ollama / LocalAI）配合
- 用于构建边缘 AI 应用
- 配合 K3 本地大模型推理

## 相关词条

- [[LlamaIndex]] — 文档检索工具
- [[Ollama]] — 本地 LLM 平台
- [[LLM-SDK]] — SpacemiT 大语言模型 SDK
- [[../hardware/products/K3]] — K3 支持本地 LLM 推理

## 来源文档

- LangChain 官网：https://www.langchain.com/
- SpacemiT 文档中心：AI / Application Software Stack / LangChain
