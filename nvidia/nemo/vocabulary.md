---
tags: [nvidia, nemo, vocabulary]
domain: nemo
last_updated: 2026-07-21
part: Overview + Platform ✅
source: https://docs.nvidia.com/nemo/ | https://docs.nvidia.com/nemo/microservices/latest/
---

# NVIDIA NeMo 词汇表

> 来源：NeMo 官方文档 + NeMo Platform/Microservices 文档
> 格式：英文术语 → 中文译名 + 官方定义 + 例句

---

## 核心概念（Core Concepts）

### NeMo (NVIDIA NeMo) / NVIDIA NeMo
NVIDIA 模块化 AI 套件，帮助开发者管理 AI 智能体全生命周期：构建、部署和优化。由三大组成部分构成：NeMo Framework（训练）、NeMo Microservices/Platform（微服务化运维）、NeMo Agent Toolkit（智能体工具链）。
> *NVIDIA NeMo is a modular software suite of APIs and libraries that help developers manage the AI agent lifecycle—building, deploying, and optimizing AI agents at scale.*

### NeMo Framework / NeMo 框架
NeMo 的核心开源训练框架，基于 PyTorch，支持 LLM/VLM/Diffusion/Speech 模型的端到端预训练、微调和对齐。可从单 GPU 无缝扩展到多节点集群。
> *Flexible open-source framework for end to end training and development of generative AI models, scaling seamlessly from a single GPU to multi-node clusters.*

### NeMo Platform / NeMo Platform（微服务平台）
NeMo 的云原生微服务套件，为企业提供数据准备、模型微调、评估、安全防护等容器化 API 服务。也称 NeMo Microservices。
> *NeMo Platform gives you the infrastructure to build and deploy specialized AI agents with open source models.*

### NeMo Agent Toolkit / NeMo 智能体工具链
开源工具包，用于基于评估驱动的方法开发和优化智能体系统。
> *Open-source toolkit for evaluation-based development and optimization of an agentic system.*

### Megatron-LM / Megatron-LM
NVIDIA 开发的大规模分布式 Transformer 训练框架，是 NeMo Framework 的底层训练引擎。支持张量并行、流水线并行、序列并行的 3D 并行。
> *NeMo Framework is built on top of PyTorch and integrates Megatron-LM for large-scale distributed training.*

### Megatron-Bridge / Megatron-Bridge
从 NeMo Framework 26.02 起，负责桥接 Megatron-LM 训练能力的新组件，作为 NeMo Framework 容器中 LLM 训练的实现层。
> *For the NeMo Framework container, starting with 26.02, refer to Megatron-Bridge's documentation.*

---

## 训练技术（Training Techniques）

### pre-training / 预训练
在大规模未标注语料上从头训练模型，学习通用语言或多模态表征。需要海量算力（数千 GPU 运行数周到数月）。
> *NeMo Framework enables users to efficiently create and pre-train new generative AI models.*

### SFT (Supervised Fine-Tuning) / 监督微调
使用标注的（指令, 回答）对数据对预训练模型进行有监督训练，使模型学会遵循指令。NeMo Customizer 支持 Full SFT 和 LoRA 两种方式。
> *Start a Full SFT Customization Job — Learn how to start a SFT customization job using a custom dataset.*

### PEFT (Parameter-Efficient Fine-Tuning) / 参数高效微调
通过只训练少量参数来适配模型到新任务，大幅降低显存和计算需求。LoRA 是最常用的 PEFT 方法。
> *NeMo supports parameter-efficient fine-tuning techniques to reduce compute and memory requirements.*

### LoRA (Low-Rank Adaptation) / LoRA 低秩适配
PEFT 方法，在原始权重矩阵旁插入低秩分解矩阵（A×B），只训练这两个小矩阵。NeMo Customizer 原生支持 LoRA 微调和适配器管理。
> *LoRA jobs: An adapter attached to the original Model Entity.*

### full fine-tuning / 全参数微调
更新模型所有参数的微调方式。效果通常最好但显存和计算需求最高。NeMo Customizer 支持 Full SFT。
> *Full fine-tuning jobs: A new Model Entity with the customized weights.*

### RLHF (Reinforcement Learning from Human Feedback) / 人类反馈强化学习
使用人类偏好数据训练奖励模型，再通过 RL（通常是 PPO）优化 LLM 生成策略的对齐方法。ChatGPT 的核心训练方法之一。
> *NeMo Framework supports alignment techniques including RLHF.*

### alignment / 对齐
使 LLM 输出符合人类价值观、安全要求和指令遵循能力的训练过程。包括 SFT、RLHF、DPO、RLAIF 等方法。
> *NeMo enables users to align models to human preferences and safety requirements.*

### DPO (Direct Preference Optimization) / 直接偏好优化
无需显式奖励模型的对齐方法，直接使用人类偏好对比数据优化 LLM。比 RLHF 更简单稳定。
> *NeMo Framework supports DPO for model alignment.*

### sequence packing / 序列打包
将多个短序列连接成一个长序列以最大化 GPU token 利用率的训练优化技术，避免填充 token 浪费算力。
> *Optimize for Tokens/GPU Throughput — Learn how to optimize the token-per-GPU throughput for a LoRA optimization job.*

### checkpoint / 检查点
训练过程中保存的模型权重快照。NeMo 支持将检查点转换为 HuggingFace 格式或直接部署到 NIM。
> *NeMo Framework enables users to leverage existing code and pre-trained model checkpoints.*

---

## 分布式训练并行（Distributed Training Parallelism）

### data parallelism / 数据并行
在多 GPU 上复制模型，每个 GPU 处理不同数据，梯度通过 AllReduce（NCCL）同步。NeMo 支持全分片数据并行（FSDP）。
> *Data parallelism replicates the model across GPUs and synchronizes gradients via AllReduce.*

### tensor parallelism (TP) / 张量并行
将单个层的权重矩阵切分到多 GPU，适用于超大模型。NeMo 通过 Megatron-LM 实现。
> *Tensor parallelism splits individual layer weight matrices across GPUs.*

### pipeline parallelism (PP) / 流水线并行
将模型的不同层组分配到不同 GPU 节点，形成流水线执行。适用于层数极多的超大模型。
> *Pipeline parallelism distributes model layers across different GPU nodes forming a pipeline.*

### context parallelism (CP) / 上下文并行
沿序列长度维度将输入切分到多 GPU，允许处理超长上下文（如 128K+ tokens）。
> *Context parallelism splits the input sequence across GPUs to handle very long contexts.*

### 3D parallelism / 3D 并行
数据并行 + 张量并行 + 流水线并行的组合，是训练千亿参数模型的标准配置。NeMo 通过 Megatron-LM 全面支持。
> *NeMo supports 3D parallelism combining data, tensor, and pipeline parallelism for training very large models.*

---

## NeMo Platform 组件（NeMo Platform Components）

### NeMo Customizer / NeMo 定制器（微调服务）
NeMo Platform 的模型微调微服务，提供 LoRA 和 Full SFT API。输入：基础模型 + 数据集；输出：LoRA 适配器或新模型实体。
> *Learn how to fine-tune models by making requests to NVIDIA NeMo Customizer through the API. Fine-tuned models you have created can be deployed using NVIDIA NIMs.*

### NeMo Evaluator / NeMo 评估器
NeMo Platform 的模型评估微服务，支持超过 100 个行业基准测试、LLM-as-a-Judge 评分、RAG 和 Agent 专用指标。
> *Evaluation API provides automated workflows for over 100 industry benchmarks, LLM-as-a-judge scoring, and specialized metrics for RAG and agent systems.*

### NeMo Guardrails / NeMo 护栏
提供实时安全防护的微服务，阻断不安全输入/输出的 LLM 应用。
> *Real-time protection with guardrails.*

### NeMo Data Designer / NeMo 数据设计师
合成训练数据生成微服务，帮助创建特定领域高质量训练数据集。
> *Synthetic data generation — Generate synthetic training data for fine-tuning workflows.*

### NeMo Auditor / NeMo 审计器
安全漏洞扫描微服务，检测 LLM 应用中的潜在安全漏洞。
> *Test and protect AI agents — Scan for vulnerabilities with Auditor.*

### customization job / 微调任务
NeMo Customizer 中表示一个微调工作单元的对象，包含模型实体引用、数据集、超参数配置等。
> *Create a customization job referencing the Model Entity.*

### model entity / 模型实体
NeMo Platform 中表示模型的数据对象，关联到 FileSet（模型权重的存储位置）。
> *Create a Model Entity pointing to your base model checkpoint (stored as a FileSet).*

### FileSet / 文件集
NeMo Platform 中存储模型权重、数据集等大文件的集合。
> *A Model Entity pointing to your base model checkpoint stored as a FileSet.*

### workspace / 工作空间
NeMo Platform 的组织和授权边界单元。所有平台资源（模型、数据集、任务、评估结果）都属于某个工作空间。
> *Workspaces provide organizational and authorization boundaries — create separate workspaces to isolate teams, users, environments, or clients.*

### project / 项目（NeMo Platform）
工作空间内用于组织相关资源的标签。用于整合微调实验、评估活动等相关工作。
> *Group related resources with projects. Projects are organizational tags within a workspace.*

---

## 评估（Evaluation）

### benchmark / 基准测试
将一个或多个评估指标与数据集配对的标准化评估套件。用于跨模型比较和端到端评估。
> *Benchmarks: Evaluation suites that pair one or more metrics with a dataset.*

### metric / 评估指标（NeMo）
对模型输出进行评分的逻辑单元。可复用于不同数据集和任务特定标准。
> *Metrics: Scoring logic that evaluates model outputs.*

### LLM-as-a-Judge / 大模型评判
使用另一个 LLM 对生成输出进行评分的评估方法。支持自定义评分标准（rubric）和数值范围。
> *Use another LLM to evaluate outputs with flexible scoring criteria. Define custom rubrics or numerical ranges.*

### live evaluation / 实时评估
同步提交请求并立即获得评估结果的模式。适合快速迭代和指标开发。
> *Live evaluation (synchronous): Submit a request and get results immediately.*

### offline evaluation / 离线评估
对已有的模型输出数据集进行评分，不需要重新生成。
> *Offline evaluation: Score existing dataset rows (for example, model outputs already generated).*

### online evaluation / 在线评估
作为评估的一部分从模型生成输出，然后进行评分。
> *Online evaluation: Generate outputs from a model as part of evaluation, then score them.*

### RAG evaluation / RAG 评估
专门评估检索增强生成流水线质量的指标，包括 recall@k、faithfulness（忠实度）、answer-relevancy（答案相关性）。
> *RAG Evaluation Metrics: recall@k, faithfulness, answer-relevancy.*

### agentic evaluation / 智能体评估
评估 Agent 和多步推理模型的指标，包括 topic-adherence（主题遵循度）、tool-call-accuracy（工具调用精度）、goal-accuracy（目标达成率）。
> *Agentic Evaluation Metrics: topic-adherence, tool-call-accuracy, goal-accuracy.*

---

## 模型与架构（Models and Architecture）

### Nemotron / Nemotron
NVIDIA 研发的 LLM 系列模型，基于 Llama 架构并经过 NVIDIA 特定训练优化。包括 Llama Nemotron Nano/Super/Ultra 等。
> *Llama Nemotron Models — available Llama Nemotron models from NVIDIA, including Nano and Super variants.*

### LLM (Large Language Model) / 大语言模型
基于 Transformer 的大规模自回归语言模型，如 Llama、Mistral、GPT 等。NeMo 是 NVIDIA LLM 训练的主要框架。
> *NeMo Framework enables efficient training and customization of Large Language Models.*

### VLM (Vision-Language Model) / 视觉-语言模型
处理图像和文本双模态输入的大模型，如 LLaVA、CLIP。NeMo 支持 VLM 训练。
> *NeMo Framework supports training of Large Language Models, Multimodal (VLM), and Speech AI models.*

### ASR (Automatic Speech Recognition) / 自动语音识别
将语音转换为文本的任务，NeMo Framework Speech AI 组件的核心功能。
> *NeMo Framework provides support for training and customization of Speech AI models, including Automatic Speech Recognition.*

### TTS (Text-to-Speech) / 文本转语音
将文本合成为语音的任务，NeMo Framework Speech AI 的另一核心功能。
> *NeMo Framework provides support for TTS synthesis.*

### Riva / NVIDIA Riva
NVIDIA 的端到端语音 AI 框架，NeMo 训练/定制的语音模型可通过 Riva 部署。
> *NeMo models can be optimized and deployed with NVIDIA Riva.*

---

## 生产部署（Production Deployment）

### RBAC (Role-Based Access Control) / 基于角色的访问控制
NeMo Platform 的多租户权限控制机制。用户在特定工作空间内被赋予 Viewer、Editor 或 Admin 角色。
> *Production-grade features include RBAC and observability.*

### observability / 可观测性
NeMo Platform 的监控和追踪能力，支持 OpenTelemetry 集成，提供服务健康状态和性能指标。
> *Production-grade features include RBAC and observability.*

### NeMo Platform CLI (nmp) / NeMo Platform 命令行工具
与 NeMo Platform 交互的命令行接口，支持模型列出、聊天、工作空间管理等操作。
> *nmp models list, nmp chat --help, nmp workspaces list*

### air-gapped deployment / 隔离网络部署
在与外部互联网断开连接的安全环境中部署 NeMo Platform，适用于高安全要求场景。
> *NeMo Platform supports deployment in air-gapped environments.*

---

## 来源文档

- NeMo 主页: https://docs.nvidia.com/nemo/
- NeMo Framework Overview: https://docs.nvidia.com/nemo-framework/user-guide/latest/overview.html
- NeMo Platform (Microservices): https://docs.nvidia.com/nemo/microservices/latest/
- NeMo Customizer: https://docs.nvidia.com/nemo/microservices/latest/customizer/index.html
- NeMo Evaluator: https://docs.nvidia.com/nemo/microservices/latest/evaluator/index.html
