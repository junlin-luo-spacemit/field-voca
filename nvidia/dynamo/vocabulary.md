---
tags: [nvidia, dynamo, vocabulary]
domain: dynamo
last_updated: 2026-07-21
part: Overview + Architecture ✅
source: https://docs.nvidia.com/dynamo/ | https://github.com/ai-dynamo/dynamo
---

# NVIDIA Dynamo 词汇表

> 来源：Dynamo 官方文档 + GitHub README (v1.2.1)
> 格式：英文术语 → 中文译名 + 官方定义 + 例句

---

## 核心概念（Core Concepts）

### Dynamo (NVIDIA Dynamo) / NVIDIA Dynamo
开源的数据中心级推理编排层。不替代 SGLang/TensorRT-LLM/vLLM 等推理引擎，而是将它们协调成一个跨多节点的统一推理系统。用 Rust 编写核心以保证性能，用 Python 实现可扩展性。
> *Dynamo is the orchestration layer above inference engines — it doesn't replace SGLang, TensorRT-LLM, or vLLM, it turns them into a coordinated multi-node inference system.*

### orchestration layer / 编排层
Dynamo 在推理引擎之上的协调层级，负责请求路由、资源调度、扩缩容等跨节点协同工作，而非执行具体的模型计算。
> *Most inference engines optimize a single GPU or a single node. Dynamo is the orchestration layer above them.*

### backend-agnostic / 后端无关
Dynamo 的设计原则：兼容多种推理后端（SGLang、TensorRT-LLM、vLLM），用户可自由选择底层引擎而不改变编排逻辑。
> *Dynamo is backend-agnostic and Kubernetes-native without being Kubernetes-only.*

### Kubernetes-native / Kubernetes 原生
Dynamo 提供 Operator、CRD、Gateway API 集成，支持自动扩缩容、调度和集群生命周期管理，同时也支持非 K8s 环境本地运行。
> *Use the Kubernetes path when you want the operator, CRDs, Gateway API integration, autoscaling, scheduling, and cluster lifecycle management.*

---

## 核心架构组件（Core Architecture Components）

### Frontend / 前端
Dynamo 提供 OpenAI 兼容 HTTP API 的入口组件。负责接收请求、tokenize、并转发给 Router。
> *python3 -m dynamo.frontend --discovery-backend file — starts the OpenAI-compatible frontend on port 8000.*

### Router / 路由器
Dynamo 核心组件，负责基于 worker 负载和 KV cache 重叠情况做出请求路由决策，将请求分发到最优的 worker。
> *The Dynamo Router makes worker-selection decisions.*

### Worker / 工作节点
运行实际推理引擎（SGLang/TRT-LLM/vLLM）实例的组件，处理分配给它的推理请求。
> *In another terminal, launch a worker for your backend.*

### Planner / 规划器
基于 SLA 驱动的自动扩缩容器。分析工作负载并动态调整资源池大小，以最低总拥有成本（TCO）满足延迟目标。
> *Planner: SLA-driven autoscaler that profiles workloads and right-sizes pools — meets latency targets at minimum total cost of ownership (TCO).*

### Grove / Grove
拓扑感知的 Kubernetes gang scheduling operator，用于在 NVL72 等大规模集群中跨机架、主机、NUMA 节点优化部署位置。
> *Grove: K8s operator for topology-aware gang scheduling (NVL72) — places workloads optimally across racks, hosts, and NUMA nodes.*

### KVBM (KV Block Manager) / KV 块管理器
将 KV cache 跨 GPU → CPU → SSD → 远程存储进行分层卸载的组件，突破单 GPU 显存限制以支持更长上下文。
> *KV Block Manager (KVBM): Offloads KV cache across GPU → CPU → SSD → remote storage — extends effective context length beyond GPU memory.*

### ModelExpress / ModelExpress
通过 NIXL/NVLink 实现 GPU 到 GPU 模型权重流式传输的组件，大幅加速新副本的冷启动。
> *ModelExpress: Streams model weights GPU-to-GPU via NIXL/NVLink — 7x faster cold-start for new replicas.*

### AIConfigurator / AI 配置器
在几秒钟内模拟上万种部署配置的工具，帮助在不消耗实际 GPU 时间的情况下找到最优服务配置。
> *AIConfigurator: Simulates 10K+ deployment configs in seconds — finds optimal serving config without burning GPU-hours.*

### Profiler / 性能分析器
分析工作负载特性（如 prefill/decode 比例、序列长度分布）的组件，为 Planner 的调度决策提供数据支持。
> *The Profiler component analyzes workload characteristics to support Planner scheduling decisions.*

---

## 分解式服务（Disaggregated Serving）

### disaggregated prefill/decode / 分解式预填充/解码
将 LLM 推理的 prefill（计算密集）和 decode（内存带宽密集）阶段拆分到独立可扩展的 GPU 资源池，各自使用最适合其负载特性的硬件配置。
> *Disaggregated Prefill/Decode: Separates prefill and decode into independently scalable GPU pools — maximizes GPU utilization.*

### prefill phase / 预填充阶段
LLM 推理中处理输入 prompt、计算初始 KV cache 的阶段。计算密集（compute-bound），受益于高算力 GPU。
> *Disaggregated serving allows independently scaling prefill and decode.*

### decode phase / 解码阶段
LLM 推理中逐 token 自回归生成的阶段。内存带宽密集（memory-bound），受益于高显存带宽 GPU。
> *Independently scale prefill and decode (disaggregated serving).*

### aggregated serving / 聚合式服务
prefill 和 decode 在同一 worker 上执行的传统服务模式，与分解式服务相对。适合简单场景或小规模部署。
> *Pre-built recipes show Llama-3-70B on vLLM using Aggregated deployment.*

### E/P/D (Encode/Prefill/Decode) disaggregation / 编码/预填充/解码分解
多模态推理中将编码（视觉/音频编码）、预填充、解码三阶段分别拆分到独立资源池，配合嵌入缓存加速处理。
> *Multimodal E/P/D: Disaggregated encode/prefill/decode with embedding cache — 30% faster TTFT on image workloads.*

---

## KV Cache 路由（KV Cache Routing）

### KV-aware routing / KV 感知路由
Router 根据各 worker 已缓存的 KV block 与新请求的重叠程度进行路由决策，避免重复计算相同的 prefix，显著降低 TTFT。
> *KV-Aware Routing: Routes requests based on worker load and KV cache overlap — eliminates redundant prefill computation — 2x faster TTFT.*

### Flash Indexer / Flash 索引器
Dynamo 的并发全局 KV 索引系统，跟踪所有推理 worker 上缓存的每个 KV block，支持每秒超过 1 亿次操作的查询速度。
> *Flash Indexer: Inter-Galactic KV Routing — tracks every cached KV block across all inference workers at over 100M ops/sec.*

### KV cache overlap / KV 缓存重叠
新请求的 prefix 与某个 worker 已有 KV cache 内容重合的程度。重叠度越高，该 worker 越适合处理该请求（可复用缓存，跳过重复 prefill）。
> *KV-aware routing places requests for prefix reuse.*

### prefix reuse / 前缀复用
利用已缓存的公共 prefix（如系统提示词、共享上下文）的 KV 值，避免对相同前缀重复计算。
> *KV Cache Aware Routing: How the router places requests for prefix reuse.*

### KV events / KV 事件
Worker 向 Router/Indexer 广播的 KV cache 状态变更通知（块创建、驱逐等），用于维护全局缓存视图。
> *Storage-tier KV offload: S3/Azure blob support + global KV events for cluster-wide cache visibility.*

### standalone KV indexer / 独立 KV 索引器
可作为独立 HTTP 服务运行的 KV 缓存索引器，用于查询块状态而无需完整 Router。
> *Standalone KV Indexer: Run the KV cache indexer as an independent HTTP service for querying block state.*

### topology-aware KV transfer / 拓扑感知 KV 传输
将分解式 prefill/decode 之间的 KV cache 传输限制在选定拓扑域内（如同机架），减少跨域网络开销。
> *Topology-Aware KV Transfer: Keep disaggregated prefill and decode KV-cache transfers within a selected topology domain.*

---

## 部署与扩缩容（Deployment and Scaling）

### DynamoGraphDeploymentRequest (DGDR) / Dynamo 图部署请求
零配置部署的 Kubernetes CRD。用户只需指定模型、硬件和 SLA，Dynamo 自动完成性能分析、拓扑优化和部署。
> *Zero-config deploy (DGDR): Specify model, HW, and SLA in one YAML — AIConfigurator auto-profiles the workload, Planner optimizes the topology, and Dynamo deploys.*

### DynamoGraphDeployment / Dynamo 图部署
描述完整推理拓扑（前端、路由器、多个 worker 池）的 Kubernetes 资源。
> *Understand DynamoGraphDeployments, DynamoComponentDeployments, DGDR, and recipes.*

### DynamoComponentDeployment / Dynamo 组件部署
描述单个 Dynamo 组件（如某个 worker 池）的部署配置。
> *DynamoComponentDeployments represent individual component deployments within the graph.*

### Dynamo Operator / Dynamo Operator
Kubernetes Operator，管理 Dynamo 资源的生命周期，包括 DynamoGraphDeployment 的创建、更新和状态同步。
> *Install the Dynamo Platform and deploy with a single manifest via the Dynamo Operator.*

### SLA (Service Level Agreement) / 服务级别协议
定义推理服务性能目标的协议，通常包含 TTFT（首token延迟）和 ITL（token间延迟）阈值，Planner 据此调整资源。
> *sla: {ttft: 200.0, itl: 20.0} — SLA-Based Planner meets latency SLAs at minimum TCO.*

### recipe / 配方
针对特定模型+后端+部署模式预制的配置模板（如 Llama-3-70B + vLLM + Aggregated），简化常见场景的部署。
> *Pre-built recipes for common models: Llama-3-70B (vLLM, Aggregated), DeepSeek-R1 (SGLang, Disaggregated).*

### model caching (Dynamo) / 模型缓存（Dynamo）
在 Kubernetes 集群中一次性下载模型并在所有 Pod 间共享的机制，避免重复下载。
> *Model Caching: Download models once and share across all pods in a Kubernetes cluster.*

---

## 请求路由拓扑（Request Routing Topologies）

### Dynamo-native Frontend routing / Dynamo 原生前端路由
Dynamo Frontend 直接服务 HTTP 并通过内置 Router 做出路由决策，无需外部网关。适用于本地开发和单集群部署。
> *The Dynamo Frontend serves HTTP and the integrated Dynamo Router makes worker-selection decisions. No external gateway is required.*

### Gateway API routing (GAIE) / 网关 API 路由
通过 Kubernetes Gateway API Inference Extension 网关调用 Dynamo Endpoint Picker Plugin (EPP)，再转发到目标 worker 的 Frontend sidecar。
> *A Kubernetes Gateway API Inference Extension gateway calls the Dynamo Endpoint Picker Plugin (EPP) before forwarding to the selected worker's Frontend sidecar.*

### Endpoint Picker Plugin (EPP) / 端点选择插件
Gateway API 路径中负责选择目标 worker 的插件组件，替代 Dynamo 原生 Router 在网关侧做决策。
> *The Dynamo Endpoint Picker Plugin (EPP) selects the target worker before forwarding.*

---

## 容错与可靠性（Fault Tolerance）

### request migration / 请求迁移
当 worker 发生故障时，将处于进行中（in-flight）的请求迁移到健康 worker 继续处理，避免用户请求失败。
> *Fault Tolerance: Canary health checks + in-flight request migration — workers fail; user requests don't.*

### canary health check / 金丝雀健康检查
持续监测 worker 健康状态的机制，尽早发现故障 worker 并触发请求迁移。
> *Canary health checks detect worker failures early to trigger request migration.*

### request cancellation / 请求取消
允许客户端主动取消正在处理的推理请求的机制，释放占用的计算资源。
> *Request Cancellation: cancel in-flight requests to free compute resources.*

### graceful shutdown / 优雅关闭
worker 下线时先完成或迁移进行中请求，再释放资源的关闭流程，避免请求丢失。
> *Graceful Shutdown: complete or migrate in-flight requests before releasing resources.*

---

## 智能体与多轮对话（Agentic Serving）

### agentic inference / 智能体推理
针对 Agent 工作负载优化的推理模式，支持逐请求优先级、预期输出长度提示、推测性预填充等。
> *Agentic inference: Per-request hints for priority, expected output length, and speculative prefill.*

### agent hints / Agent 提示
客户端为每个请求附加的元数据提示（优先级、预期长度等），帮助 Dynamo 更优地调度该请求。
> *Agent Hints: Per-request serving hints for agentic workloads.*

### priority scheduling / 优先级调度
Router 和后端引擎根据请求优先级进行调度的机制，确保高优先级请求（如实时交互）获得更快响应。
> *Priority Scheduling: Request priority across the Dynamo router and backend engines.*

### session metadata / 会话元数据
跟踪多轮对话/Agent 会话的元数据信息，用于追踪和 SGLang 子 Agent 的 KV 隔离。
> *Session metadata for tracing and SGLang subagent KV isolation.*

### tool calling / 工具调用
LLM 生成结构化工具调用请求的能力。Dynamo 内置解析器将模型输出解析为 OpenAI 兼容的 tool_calls 格式。
> *Tool Calling & Reasoning Parsing: Parse tool calls and reasoning out of model output into OpenAI-compatible tool_calls.*

### reasoning parsing / 推理内容解析
从模型输出中解析思维链（thinking/reasoning）内容的功能，分离最终答案和推理过程。
> *Reasoning Parsing: Configure Dynamo's built-in reasoning parsers for models that emit thinking content.*

---

## 多模态与扩展场景（Multimodal and Extended Scenarios）

### embedding cache (Dynamo) / 嵌入缓存
缓存视觉编码器生成的嵌入向量，避免对重复图像重新编码，加速多模态推理。
> *Embedding Cache: Cache vision encoder embeddings to skip re-encoding repeated images.*

### encoder disaggregation / 编码器分解
将视觉/音频编码阶段拆分到独立 worker 池，可与 prefill/decode 分开独立扩展。
> *Encoder Disaggregation: Separate vision encoding into a dedicated worker for independent scaling.*

### multimodal KV routing / 多模态 KV 路由
将多模态请求路由到具有最佳 KV 缓存重叠的 worker，考虑图像/文本混合上下文的缓存复用。
> *Multimodal KV Routing: Route multimodal requests to workers with the best KV cache overlap.*

### diffusion serving / 扩散模型服务
Dynamo 对文生图、文生视频等扩散模型的原生支持，集成 FastVideo 和 SGLang Diffusion。
> *Video generation: Native FastVideo + SGLang Diffusion support — real-time 1080p on single B200.*

---

## 性能与仿真（Performance and Simulation）

### DynoSim / DynoSim
Dynamo 服务栈的工作负载驱动离散事件仿真工具，用于在真实集群验证前绘制帕累托前沿（Pareto frontier）。
> *DynoSim: Simulating the Pareto Frontier — a workload-driven discrete-event simulation of the NVIDIA Dynamo serving stack.*

### Pareto frontier / 帕累托前沿
在延迟和成本/吞吐量等多个目标之间的最优权衡曲线，DynoSim 用于在部署前找到该曲线。
> *DynoSim maps Pareto frontiers before real-cluster validation.*

### TTFT (Time To First Token) / 首 token 延迟
从请求发出到生成第一个 token 的时间，KV-aware routing 可将其降低约 2 倍。
> *2x faster time to first token — KV-aware routing, Qwen3-Coder 480B (Baseten benchmark).*

### ITL (Inter-Token Latency) / token 间延迟
连续生成 token 之间的时间间隔，是 SLA 定义中的关键延迟指标之一。
> *sla: {ttft: 200.0, itl: 20.0}*

### AIPerf / AIPerf
用于比较不同部署拓扑性能的基准测试工具。
> *Benchmarking Guide: Compare deployment topologies with AIPerf.*

---

## 来源文档

- Dynamo GitHub README: https://github.com/ai-dynamo/dynamo/blob/main/README.md
- Dynamo 文档索引: https://docs.nvidia.com/dynamo/latest/llms.txt
- Dynamo Quickstart: https://docs.nvidia.com/dynamo/getting-started/quickstart.html
