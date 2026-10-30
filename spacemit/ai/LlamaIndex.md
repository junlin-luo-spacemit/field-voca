---
tags: [ai, framework, llamaindex, spacemit]
domain: ai
aliases: [LlamaIndex]
status: needs_review
created: 2026-08-06
updated: 2026-10-29
source:
superseded_by:
---

# LlamaIndex

## 定义

LlamaIndex 是用于构建基于文档检索的 LLM 应用的工具集，管理文档索引、向量化与查询流程，实现检索增强生成（RAG）。

## 为什么重要

- **文档问答**：将自有文档转化为可问答知识库
- **长文本处理**：突破 LLM 上下文窗口限制
- **精准检索**：向量化 + 语义搜索
- **减少幻觉**：用真实文档作为依据

## 主要功能

1. **文档加载**
   - 支持 PDF、Markdown、HTML、API 等多种格式
   - 自动分块与预处理

2. **索引构建**
   - 向量索引（Vector Store Index）
   - 树形索引（Tree Index）
   - 关键词索引（Keyword Table Index）

3. **检索查询**
   - 语义相似度搜索
   - 混合检索（向量 + 关键词）
   - 重排序（Reranking）

4. **集成**
   - 支持多种 LLM（OpenAI、Ollama、LocalAI）
   - 多种向量数据库（Chroma、Pinecone、Qdrant）

## RAG 工作流程

```
文档 → 分块 → 向量化 → 存储到向量数据库
                         ↓
用户查询 → 向量化 → 检索相关文档 → 构造 Prompt → LLM 生成回答
```

## 使用场景

- 知识库问答
- 技术文档助手
- 法律/医疗文档分析
- 企业内部知识管理

## 与 LangChain 的关系

- **LlamaIndex**：专注文档索引与检索
- **LangChain**：通用 LLM 应用框架
- **常见组合**：LlamaIndex 负责检索，LangChain 负责编排

## 在 SpacemiT 中

- 配合本地 LLM（Ollama / LocalAI）
- 用于知识库问答等场景
- K3 本地大模型推理 + 文档检索

## 相关词条

- [[LangChain]] — LLM 应用开发框架
- [[Ollama]] — 本地 LLM 平台
- [[LLM-SDK]] — SpacemiT 大语言模型 SDK
- [[../hardware/products/K3]] — K3 支持本地 RAG 应用

## 来源文档

- LlamaIndex 官网：https://www.llamaindex.ai/
- SpacemiT 文档中心：AI / Application Software Stack / LlamaIndex
