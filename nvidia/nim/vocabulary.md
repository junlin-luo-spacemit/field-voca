---
tags: [nvidia, nim, vocabulary]
domain: nim
last_updated: 2026-07-21
part: Overview + Concepts ✅
source: https://docs.nvidia.com/nim/ | https://developer.nvidia.com/blog/
---

# NVIDIA NIM 词汇表

> 来源：NIM 官方文档 + NIM Operator 文档 + NVIDIA Blog
> 格式：英文术语 → 中文译名 + 官方定义 + 例句

---

## 核心概念（Core Concepts）

### NIM (NVIDIA Inference Microservice) / NVIDIA 推理微服务
NVIDIA 将预训练 AI 模型封装成的容器化推理微服务。内置预优化推理引擎（TRT-LLM、vLLM 等），提供 OpenAI 兼容 API，可在云、数据中心、工作站上以单条命令部署。
> *NIM provides pre-packaged, state-of-the-art AI models that are optimized for deployment across NVIDIA GPUs.*

### NVIDIA AI Enterprise / NVIDIA AI 企业版
NVIDIA 企业级 AI 软件平台，NIM 是其核心组成部分。提供生产级运行时、持续安全更新和企业支持。
> *Part of NVIDIA AI Enterprise, NVIDIA NIM microservices are easy-to-use microservices for accelerating the deployment of foundation models.*

### NGC (NVIDIA GPU Cloud) / NVIDIA GPU 云服务
NVIDIA 的容器镜像仓库和模型/软件分发平台。NIM 容器镜像从 NGC 拉取（`nvcr.io/nim/...`）。
> *NIM containers are pulled from nvcr.io/nim/<vendor>/<model>:<version>.*

### NGC API Key / NGC API 密钥
访问 NGC 私有资源（包括 NIM 容器）所需的认证凭据。通过 `$oauthtoken` 用户名进行 Docker 认证。
> *You need an NVIDIA AI Enterprise license or NVIDIA Developer Program membership to access NIM containers via NGC API Key.*

### NVIDIA API Catalog / NVIDIA API 目录
NVIDIA 托管的 NIM API 云端演示平台（build.nvidia.com）。开发者可无需自行部署即可测试 NIM API。
> *Sample NVIDIA-hosted deployments of NIM are available on the NVIDIA API Catalog.*

---

## 模型与镜像（Models and Images）

### NIM container image / NIM 容器镜像
封装了模型权重、优化引擎、运行时配置的 Docker 镜像。格式：`nvcr.io/nim/<vendor>/<model>:<version>`。
> *Using a single optimized container, you can deploy a NIM microservice in under 5 minutes.*

### model profile / 模型 profile
NIM 容器内针对不同 GPU 类型、精度和推理后端预优化的配置集合。NIM 启动时自动检测 GPU 并选择最优 profile。
> *NIM microservices have many available profiles — model, precision, and backend combinations optimized for specific GPU types.*

### auto-detect profile / 自动选择 profile
NIM 启动时自动检测当前 GPU 的 Compute Capability，并从容器内置的 profile 中选择最优配置，无需用户手动指定。
> *The NIM Operator auto-detects the best model based on the GPUs available on the Kubernetes cluster.*

### TRT-LLM (TensorRT-LLM) backend / TRT-LLM 后端
NIM 内置的主要 LLM 推理后端，基于 TensorRT 优化，提供最高 GPU 吞吐量。适合批量推理和高吞吐场景。
> *NIM is built with pre-optimized inference engines from NVIDIA and the community, including NVIDIA TensorRT and TensorRT-LLM.*

### vLLM backend / vLLM 后端
NIM 可选的 LLM 推理后端，基于 PagedAttention 机制，灵活性更好，支持更多模型格式。
> *NIM includes support for community inference engines including vLLM.*

### model cache / 模型缓存
在 Kubernetes 或本地文件系统中预先下载并存储的模型权重和 profile 数据。避免每次启动时重新下载（可达数十 GB）。
> *The NIM Operator allows you to pre-cache models to avoid long bootstrapping times.*

### LOCAL_NIM_CACHE / 本地 NIM 缓存目录
存放下载的 NIM 模型权重的本地目录。通过 Docker volume 挂载到容器中。
> *Choose a path on your system to cache the downloaded models: export LOCAL_NIM_CACHE="~/.cache/nim"*

---

## API 接口（API Interface）

### OpenAI-compatible API / OpenAI 兼容 API
NIM 提供与 OpenAI API 规范完全兼容的 REST 接口，使现有 OpenAI 应用无需修改代码即可切换到自托管 NIM。
> *NIM offers industry-standard APIs to simplify development.*

### /v1/completions / 文本补全端点
NIM LLM 的文本补全 API 端点，接受 prompt 并返回生成的文本。
> *curl -X POST http://0.0.0.0:8000/v1/completions -d '{"model": "meta/llama3-8b-instruct", "prompt": "...", "max_tokens": 64}'*

### /v1/chat/completions / 聊天补全端点
NIM LLM 的对话 API 端点，接受 messages 列表并返回助手回复。与 OpenAI Chat API 格式完全兼容。
> *client.chat.completions.create(model="meta/llama3-8b-instruct", messages=[{"role": "user", "content": "..."}])*

### streaming / 流式输出
NIM 支持将生成的 token 逐步流式返回，减少首 token 前等待时间，改善用户体验。设置 `stream: true` 启用。
> *To stream outputs, you should set stream to True.*

### base_url / 服务地址
NIM 服务的根 URL，用于 OpenAI 客户端库的 `base_url` 参数指向自托管 NIM 实例。
> *Make sure to update the base_url value to wherever your NIM microservice is running.*

---

## 模型类型（Model Types）

### reasoning LLM / 推理大语言模型
NIM 支持的模型类型之一，面向问答、代码生成、文本生成等通用场景（如 Llama 3、Nemotron）。
> *NIM supports deploying NIMs in various domains such as reasoning, retrieval, speech, and biology.*

### embedding NIM / 嵌入向量微服务
生成文本嵌入向量的 NIM，用于语义搜索、RAG、相似度比较等场景。
> *Retrieval — embedding, reranking, and other functions.*

### reranking NIM / 重排序微服务
对检索结果进行语义重排序的 NIM，提高 RAG 管道中的检索精度。
> *NIM microservices for retrieval including reranking.*

### speech NIM / 语音微服务
语音识别（ASR）和语音合成（TTS）的 NIM 微服务。
> *NIM supports deploying NIMs in speech domain.*

### biology NIM / 生物信息微服务
用于蛋白质结构预测（AlphaFold 等）等生物信息学任务的 NIM。
> *NIM supports deploying NIMs in biology domain.*

### multimodal NIM / 多模态微服务
处理图像+文本等多种模态输入的 NIM（如 LLaVA、CLIP）。
> *NIM microservices support vision models.*

---

## 定制化（Customization）

### LoRA (Low-Rank Adaptation) / LoRA 低秩适配
高效微调方法，只训练少量低秩参数。NIM 支持运行时动态加载多个 LoRA 适配器，无需重启服务。
> *NIM supports LoRA adapters trained using either HuggingFace or NVIDIA NeMo.*

### PEFT (Parameter-Efficient Fine-Tuning) / 参数高效微调
统称一类仅更新少量参数的微调方法，LoRA 是最常用的 PEFT 方法。NIM 原生支持 PEFT 适配器。
> *Store the LoRA adapters in /LOCAL_PEFT_DIRECTORY and serve using a script similar to the one used for the base container.*

### LOCAL_PEFT_DIRECTORY / LoRA 适配器本地目录
存放 LoRA 适配器文件的本地目录，通过 Docker volume 挂载到 NIM 容器中。
> *export LOCAL_PEFT_DIRECTORY=~/loras*

### LoRA adapter / LoRA 适配器
包含 LoRA 权重的模型文件。可以是 HuggingFace 格式或 NeMo 格式。可在推理请求中通过模型名称指定要使用的适配器。
> *Deploy using the name of one of the LoRA adapters in /LOCAL_PEFT_DIRECTORY.*

---

## Kubernetes 部署（Kubernetes Deployment）

### NIM Operator / NIM Operator
Kubernetes Operator，简化 NIM 微服务的生命周期管理、模型缓存、自动扩缩容等操作。
> *The NVIDIA NIM Operator enables Kubernetes cluster administrators to operate the software components and services necessary to deploy NVIDIA NIMs microservices in Kubernetes.*

### NIMCache (nimcaches.apps.nvidia.com) / NIM 模型缓存资源
Kubernetes 自定义资源，用于从 NGC 下载并持久化存储模型数据到网络存储，供多个 NIM 实例共享。
> *This custom resource enables downloading models from NVIDIA NGC and persisting them on network storage.*

### NIMService (nimservices.apps.nvidia.com) / NIM 服务资源
Kubernetes 自定义资源，代表一个 NIM 微服务实例。创建或更新此资源会在命名空间中创建相应的 Kubernetes Deployment。
> *This custom resource represents a NIM microservice. Adding and updating a NIM service resource creates a Kubernetes deployment for the microservice.*

### NIMPipeline (nimpipelines.apps.nvidia.com) / NIM 流水线资源
Kubernetes 自定义资源，代表一组 NIMService 资源，用于管理多个 NIM 微服务构成的推理流水线。
> *This custom resource represents a group of NIM service custom resources.*

### NIMBuild (nimbuild.apps.nvidia.com) / NIM 构建资源
Kubernetes 自定义资源，用于预先在目标 GPU 上构建 TRT-LLM engine，避免部署时的现场构建开销。
> *This custom resource generates optimized TRT-LLM engine builds based on predefined model profiles within a given LLM NIM.*

### autoscaling / 自动扩缩容
NIM Operator 支持根据 DCG 指标或 NIM 特定指标自动调整 NIM 服务实例数量。
> *You can scale based on different metrics, either based on DCG metrics or NIM-specific metrics.*

### air-gapped environment / 隔离网络环境
与外部互联网断开连接的安全部署环境。NIM Operator 的模型预缓存功能支持在隔离网络中部署 NIM。
> *Deploying models into a cache will also unblock deploying into air-gapped environments by providing a place to store your models within your clusters.*

---

## 集成与生态（Integration and Ecosystem）

### LangChain integration / LangChain 集成
通过 `langchain-nvidia-ai-endpoints` 库，在 LangChain 应用中直接使用 NIM 端点，替代 OpenAI 调用。
> *NIM is integrated into application frameworks like Haystack, LangChain, and LlamaIndex.*

### LlamaIndex integration / LlamaIndex 集成
LlamaIndex RAG 框架原生支持 NIM 作为推理后端。
> *NIM microservices are integrated into LlamaIndex, bringing secure, reliable, accelerated model inferencing to developers.*

### Haystack integration / Haystack 集成
Haystack AI 框架原生支持 NIM 作为 LLM 和嵌入模型后端。
> *NIM is integrated into application frameworks like Haystack.*

### RAG (Retrieval-Augmented Generation) / 检索增强生成
将外部知识库检索结果注入 LLM 上下文的技术。NIM 是 NVIDIA RAG Blueprint 的核心推理组件。
> *Multimodal RAG is a featured blueprint using NVIDIA NIM.*

### NVIDIA AI Blueprints / NVIDIA AI 蓝图
基于 NIM 构建的端到端应用参考方案，包括 Multimodal RAG、Digital Human、Video Search 等。
> *Featured Blueprints include Multimodal RAG, Digital Human, and Video Search and Summarization.*

### Dynamo / Dynamo
NVIDIA 开源的大规模分布式和分解式推理服务框架。NIM Operator 支持通过 Dynamo CRD 进行大规模部署。
> *The NIM Operator experimentally supports deploying Dynamo CRDs enabling large-scale distributed and disaggregated inference serving.*

---

## 性能指标（Performance Metrics）

### TTFT (Time To First Token) / 首 token 延迟
从发送请求到收到第一个生成 token 的时间。衡量用户感知响应延迟。
> *TTFT is a key metric for measuring user-perceived latency in LLM inference.*

### TBT / ITL (Time Between Tokens / Inter-Token Latency) / Token 间延迟
连续 token 生成之间的时间间隔，影响流式输出的流畅程度。
> *TBT/ITL determines how smooth the streaming output appears to users.*

### throughput (tokens/s) / 吞吐量
单位时间内生成的 token 总数。衡量服务端处理能力，通常以 tokens/second 计量。
> *NIM delivers high throughput through optimized backends like TRT-LLM.*

### latency / 延迟
从请求发出到收到完整响应的时间。TTFT + (output_tokens × ITL)。
> *NIM provides optimized latency and throughput for production deployments.*

---

## 来源文档

- NIM 主页: https://docs.nvidia.com/nim/index.html
- NIM Operator: https://docs.nvidia.com/nim-operator/latest/index.html
- NIM 部署博客: https://developer.nvidia.com/blog/a-simple-guide-to-deploying-generative-ai-with-nvidia-nim/
- NIM on WSL2: https://docs.nvidia.com/nim/wsl2/latest/index.html
