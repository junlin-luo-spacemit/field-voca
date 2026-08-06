---
tags: [nvidia, triton, vocabulary]
domain: triton
last_updated: 2026-07-21
part: Overview + User Guide ✅
source: https://docs.nvidia.com/deeplearning/triton-inference-server/user-guide/docs/
---

# Triton Inference Server 词汇表

> 来源：https://docs.nvidia.com/deeplearning/triton-inference-server/user-guide/docs/
> 版本：Triton 2.x (2026)
> 格式：英文术语 → 中文译名 + 官方定义 + 例句

---

## 核心概念（Core Concepts）

### Triton Inference Server / Triton 推理服务器
NVIDIA 开源的推理服务软件。支持从多种深度学习和机器学习框架（TensorRT、PyTorch、ONNX、OpenVINO、Python等）部署任意 AI 模型。提供 HTTP/REST 和 gRPC 接口，支持云、数据中心、边缘等部署场景。
> *Triton Inference Server is an open source inference serving software that streamlines AI inferencing.*

### model repository / 模型仓库
文件系统中存放模型的目录，Triton 启动时加载。可以是本地目录、GCS、S3 或 Azure Storage。
> *The model repository is a file-system based repository of the models that Triton will make available for inferencing.*

### backend / 后端
执行特定框架模型推理的库。不同框架（TensorRT、PyTorch、ONNX Runtime、OpenVINO 等）对应各自的 backend。
> *Each model is executed by a specific backend. The backend performs inferencing using the inputs provided in the batched requests.*

### inference request / 推理请求
客户端发送给 Triton 的请求，包含输入张量数据、模型名称、版本号等信息。
> *Inference requests arrive at the server via either HTTP/REST or GRPC.*

### scheduler / 调度器
负责将推理请求分发到模型实例的组件。Triton 为不同类型的模型提供不同的调度器。
> *Triton implements multiple scheduling and batching algorithms that can be configured on a model-by-model basis.*

### model instance / 模型实例
模型在某个计算设备上的一个执行副本。多个实例允许并发处理多个请求。
> *Each such enabled parallel execution is referred to as an instance.*

---

## 模型仓库（Model Repository）

### config.pbtxt / 模型配置文件
描述模型元数据和行为的 Protocol Buffer 文本文件。包含 platform、backend、max_batch_size、inputs/outputs 等必需字段，以及调度器、实例组等可选字段。
> *Each model in a model repository must include a model configuration that provides required and optional information about the model.*

### model version / 模型版本
模型目录下按数字命名的子目录。每个数字子目录对应模型的一个版本，Triton 可以同时服务多个版本。
> *Each model can have one or more versions available in the model repository. Each version is stored in its own, numerically named, subdirectory.*

### version policy / 版本策略
控制哪些模型版本对外可用。有三种策略：All（所有版本）、Latest（最新 n 个版本，默认 n=1）、Specific（指定版本列表）。
> *The ModelVersionPolicy property of the model configuration is used to set one of the following policies: All, Latest, Specific.*

### model.plan / TensorRT 模型文件
TensorRT 模型在 Triton 中的默认文件名，对应序列化的 TensorRT engine（plan 文件）。与特定 GPU 的 Compute Capability 绑定。
> *A TensorRT Plan is a single file that by default must be named model.plan.*

### model.onnx / ONNX 模型文件
ONNX 模型在 Triton 中的默认文件名，可以是单个文件或包含多个文件的目录。
> *An ONNX model is a single file or a directory containing multiple files. By default the file or directory must be named model.onnx.*

### model.pt / TorchScript 模型文件
TorchScript（PyTorch）模型在 Triton 中的默认文件名。
> *An TorchScript model is a single file that by default must be named model.pt.*

### model.py / Python 模型文件
Python backend 模型的默认文件名。允许在 Triton 中运行任意 Python 代码作为推理模型。
> *The Python backend allows you to run Python code as a model within Triton. By default the Python script must be named model.py.*

---

## 模型配置（Model Configuration）

### platform / 框架平台
指定模型所用框架的字段。例如 "tensorrt_plan"、"onnxruntime_onnx"、"pytorch_libtorch" 等。
> *The required values for platform and backend are described in the backend documentation.*

### max_batch_size / 最大批次大小
模型支持的最大批次大小。设置为大于 0 表示模型支持批处理，Triton 可以使用 dynamic batcher 或 sequence batcher。设置为 0 表示不支持批处理。
> *The max_batch_size property indicates the maximum batch size that the model supports for the types of batching that can be exploited by Triton.*

### dims / 张量维度
模型输入或输出张量的维度，不含批次维度。值 -1 表示可变大小维度。
> *An input shape indicates the shape of an input tensor expected by the model and by Triton in inference requests.*

### reshape / 张量重塑
用于在 Triton 接受的形状与模型期望的形状之间进行转换。解决接口与模型的形状不匹配问题。
> *The ModelTensorReshape property on a model configuration input or output is used to indicate that the input or output shape accepted by the inference API differs from the input or output shape expected or produced by the underlying framework model.*

### instance_group / 实例组配置
控制模型实例数量和运行位置的配置。可指定实例数量（count）、执行设备（KIND_GPU/KIND_CPU）和具体 GPU 编号。
> *The ModelInstanceGroup property is used to specify the number of execution instances that should be made available and what compute resource should be used for those instances.*

### KIND_GPU / GPU 实例
在 GPU 上运行的模型实例类型。
> *KIND_GPU: Execute the model instance on a GPU.*

### KIND_CPU / CPU 实例
在 CPU 上运行的模型实例类型。
> *KIND_CPU: Execute the model instance on a CPU.*

### response_cache / 响应缓存
为重复请求提供缓存加速的配置。启用后，相同输入的请求可直接返回缓存结果而不执行实际推理。
> *The model configuration response_cache section has an enable boolean used to enable the Response Cache for this model.*

### model warmup / 模型预热
避免首次推理延迟的机制。Triton 在模型准备好接受外部请求前，会先发送一系列预热推理请求以完全初始化模型。
> *Triton provides a configuration option that enables a model to be "warmed up" so that it is completely initialized before the first inference request is received.*

### cc_model_filenames / Compute Capability 模型文件映射
将 GPU Compute Capability 映射到对应 TensorRT plan 文件名的配置。TensorRT 模型绑定特定 CC，此字段用于同一模型支持多种 GPU。
> *The cc_model_filenames field maps the GPU's CUDA Compute Capability to a corresponding model filename at model load time.*

### auto-complete config / 自动补全配置
Triton 自动推断并填充 config.pbtxt 中必要字段的特性。TensorRT、ONNX、OpenVINO 模型支持自动配置，Python 模型可实现 auto_complete_config 函数。
> *Triton can derive all the required settings automatically for most of the TensorRT saved-model, ONNX models, and OpenVINO models.*

### model_transaction_policy / 模型事务策略
描述模型响应特性的配置。decoupled 为 true 时，响应数量可与请求数量不同，且可无序返回。
> *The model_transaction_policy property describes the nature of transactions expected from the model.*

### decoupled mode / 解耦模式
模型可以为单个请求生成多个响应，或不按请求顺序返回响应。常用于流式生成（LLM）。
> *Decoupled means the number of responses generated by the model may differ from number of requests issued, and the responses may be out of order relative to the order of requests.*

---

## 调度与批处理（Scheduling and Batching）

### dynamic batching / 动态批处理
Triton 将多个推理请求合并成一个批次执行的功能。适用于无状态模型。通常显著提高吞吐量。
> *Dynamic batching is a feature of Triton that allows inference requests to be combined by the server, so that a batch is created dynamically.*

### preferred_batch_size / 首选批次大小
动态批处理器尝试创建的目标批次大小。通常不需要设置；仅当特定批次大小有显著性能优势时才配置（如 TensorRT 多优化 profile）。
> *The preferred_batch_size property indicates the batch sizes that the dynamic batcher should attempt to create.*

### max_queue_delay_microseconds / 最大队列延迟
动态批处理器等待更多请求加入批次的最大延迟时间（微秒）。增大此值可提高吞吐量但会增加延迟。
> *The max_queue_delay_microseconds property setting changes the dynamic batcher behavior when a maximum size (or preferred size) batch cannot be created.*

### sequence batcher / 序列批处理器
用于有状态模型的调度器。确保同一序列的所有请求路由到同一模型实例，以正确维护模型状态。
> *The sequence batcher should be used for stateful models where a sequence of inference requests must be routed to the same model instance.*

### stateless model / 无状态模型
每次推理独立，不依赖之前请求的状态。CNN（图像分类、目标检测）等为典型无状态模型。使用 default scheduler 或 dynamic batcher。
> *A stateless model does not maintain state between inference requests. Each inference performed on a stateless model is independent of all other inferences.*

### stateful model / 有状态模型
维护跨请求状态的模型。同一序列的请求必须路由到同一模型实例。使用 sequence batcher。典型例子：RNN、LSTM。
> *A stateful model does maintain state between inference requests. The model is expecting multiple inference requests that together form a sequence.*

### correlation ID / 关联 ID
用于标识属于同一序列的所有请求的唯一标识符。客户端为一个序列中的所有请求设置相同的 correlation ID。
> *When making inference requests for a stateful model, the client application must provide the same correlation ID to all requests in a sequence.*

### control input / 控制输入
Triton 通过特殊输入张量与有状态模型通信的机制。包括 START（序列开始）、END（序列结束）、READY（请求就绪）、CORRID（关联ID）。
> *For a stateful model to operate correctly with the sequence batcher, the model must typically accept one or more control input tensors that Triton uses to communicate with the model.*

### iterative sequence / 迭代序列
单个请求被分解为多个调度迭代处理的模式。用于 LLM 推理中的 continuous batching / inflight batching。
> *The sequence batcher supports stateful execution of "iterative sequences" where a single request is processed over a number of scheduling iterations.*

### continuous batching / 持续批处理
在 LLM 推理中，每个迭代步骤都形成批次，允许新请求随时加入。也称 inflight batching 或 iteration-level batching。
> *Continuous batching, iteration level batching, and inflight batching are terms used in LLM inferencing to describe batching strategies that form batches of requests at each iteration step.*

### inflight batching / 在途批处理
LLM 推理中将处于不同生成步骤的多个请求组合到同一批次的技术。最大化 GPU 利用率。
> *By forming batches "continuously" inference servers can increase throughput by reusing batch slots as soon as they are free without waiting for all requests in a batch to complete.*

### priority levels / 优先级级别
动态批处理器中的多优先级队列配置。高优先级请求可优先被处理，低优先级请求等待。
> *The priority_levels property can be used to create multiple priority levels within the dynamic batcher so that requests with higher priority are allowed to bypass requests with lower priority.*

---

## 并发执行（Concurrent Execution）

### concurrent model execution / 并发模型执行
Triton 允许多个模型和同一模型的多个实例在同一系统上并行执行的特性。
> *The Triton architecture allows multiple models and/or multiple instances of the same model to execute in parallel on the same system.*

### instance_group count / 实例数量
每个模型配置的并发执行实例数量。增加实例数允许同时处理更多请求，但占用更多 GPU/CPU 资源。
> *Each model is given a single instance for each available GPU in the system by default. By using the instance_group field, the number of execution instances for a model can be changed.*

---

## 模型流水线（Model Pipelines）

### ensemble model / 集成模型（pipeline）
表示一条或多条模型组成的推理流水线。将预处理、推理、后处理等步骤串联，避免中间张量传输开销。
> *An ensemble model represents a pipeline of one or more models and the connection of input and output tensors between those models.*

### ensemble scheduler / 集成调度器
用于集成模型的专用调度器，按照配置中的步骤定义路由张量数据到各子模型。
> *The ensemble scheduler must be used for ensemble models, regardless of the scheduler used by the models within the ensemble.*

### step / 步骤（ensemble step）
集成模型中的一个子模型执行单元。包含模型名称、版本及输入输出张量的映射关系。
> *Each element in step section specifies the model to be used and how the inputs and outputs of the model are mapped to tensor names recognized by the scheduler.*

### input_map / 输入映射
集成步骤中将集成模型的张量名称映射到子模型输入张量名称的配置。
> *key: An input tensor name on the composing model. value: A tensor name on the ensemble model.*

### output_map / 输出映射
集成步骤中将子模型输出张量名称映射到集成模型张量名称的配置。
> *The output_map maps the composing model's output tensor to a name that is recognized within the ensemble.*

### BLS (Business Logic Scripting) / 业务逻辑脚本
允许在 Python backend 推理函数中直接调用其他 Triton 模型的机制。比 ensemble 更灵活，支持条件逻辑和循环。
> *BLS allows executing a sequence of Triton inference requests from within the execution of a Python backend model.*

### max_inflight_requests / 最大在途请求数
集成模型中限制每个步骤并发在途请求数量的配置，防止上游模型生产速度超过下游消费速度导致的内存溢出。
> *The max_inflight_requests field defines a limit on the number of concurrent in-flight requests allowed at each ensemble step.*

---

## 协议与接口（Protocols and APIs）

### KServe protocol / KServe 协议
Triton 基于社区开发的 KServe 预测 API v2 协议提供 HTTP/REST 和 gRPC 接口。
> *HTTP/REST and GRPC inference protocols based on the community developed KServe protocol.*

### HTTP/REST API / HTTP REST 接口
Triton 在端口 8000 上提供的推理接口。标准格式：`POST /v2/models/<model>/infer`。
> *Inference requests arrive at the server via HTTP/REST.*

### gRPC API / gRPC 接口
Triton 在端口 8001 上提供的高性能推理接口。相比 HTTP REST，延迟更低，适合高频调用。
> *Inference requests arrive at the server via GRPC.*

### metrics endpoint / 指标端点
Triton 在端口 8002 上提供 Prometheus 格式的系统指标，包含 GPU 利用率、推理吞吐量、延迟等信息。
> *Readiness and liveness health endpoints and utilization, throughput and latency metrics ease the integration of Triton into deployment frameworks such as Kubernetes.*

### health endpoint / 健康检查端点
用于确认 Triton 服务器和模型是否就绪的接口。`GET /v2/health/ready` 返回 200 表示服务器就绪。
> *Use Triton's ready endpoint to verify that the server and the models are ready for inference.*

### C API / C 接口
允许将 Triton 作为库直接嵌入应用程序的接口。适合边缘部署等不需要独立服务进程的场景。
> *A C API allows Triton to link directly into your application for edge and other in-process use cases.*

### tritonserver CLI / tritonserver 命令行
启动 Triton 推理服务器的命令行工具。常用参数：--model-repository、--model-control-mode、--disable-auto-complete-config。
> *The Triton Inference Server is available as buildable source code, but the easiest way to install and run Triton is to use the pre-built Docker image.*

---

## 后端（Backends）

### TensorRT backend / TensorRT 后端
执行 TensorRT plan 文件（优化过的 engine）推理的后端。提供最优 GPU 推理性能。
> *A TensorRT Plan is a single file. TensorRT Plans are specific to the GPU's CUDA Compute Capability.*

### ONNX Runtime backend / ONNX Runtime 后端
执行 ONNX 模型推理的后端，基于 Microsoft ONNX Runtime。支持广泛的 ONNX opset。
> *Triton supports all ONNX models that are supported by the version of ONNX Runtime being used.*

### PyTorch backend / PyTorch（TorchScript）后端
执行 TorchScript 模型推理的后端。
> *A TorchScript model is a single file that by default must be named model.pt.*

### OpenVINO backend / OpenVINO 后端
执行 Intel OpenVINO 模型推理的后端。使用 .xml 和 .bin 文件对。
> *An OpenVINO model is represented by two files, a *.xml and *.bin file.*

### Python backend / Python 后端
允许将任意 Python 代码作为 Triton 模型运行的后端。灵活但性能不如原生后端。
> *The Python backend allows you to run Python code as a model within Triton.*

### DALI backend / DALI 后端
用于运行 NVIDIA DALI（数据加载和预处理）流水线的后端。加速图像、音频、视频的预处理。
> *The DALI backend allows you to run a DALI pipeline as a model within Triton.*

### custom backend / 自定义后端
通过 Backend C API 实现的扩展后端。可以添加新框架支持或自定义预后处理。
> *Triton supports a backend C API that allows Triton to be extended with new functionality such as custom pre- and post-processing operations.*

---

## 模型管理（Model Management）

### model control mode / 模型控制模式
控制 Triton 加载/卸载模型方式的设置。三种模式：NONE（静态，启动时加载所有模型）、EXPLICIT（显式 API 控制）、POLL（定期轮询模型仓库变化）。
> *While Triton is running, the models being served can be modified using model management.*

### model management API / 模型管理 API
通过 HTTP/REST 或 gRPC 动态加载/卸载/更新模型的接口。
> *The models being served by Triton can be queried and controlled by a dedicated model management API.*

### model analyzer / 模型分析工具
自动搜索最优模型配置（实例数、批处理设置等）的工具，帮助找到最佳性能配置。
> *The Model Analyzer automatically searches across different dynamic batching configurations.*

### performance analyzer / 性能分析工具
测量 Triton 推理延迟和吞吐量的客户端工具。用于分析不同配置下的性能表现。
> *Use the Performance Analyzer to determine the latency and throughput provided by the default dynamic batcher configuration.*

---

## 云存储支持（Cloud Storage）

### GCS (Google Cloud Storage) / 谷歌云存储
Triton 支持将模型仓库存放在 GCS，路径前缀 `gs://`。
> *For a model repository residing in Google Cloud Storage, the repository path must be prefixed with gs://.*

### S3 (Amazon S3) / 亚马逊 S3 存储
Triton 支持将模型仓库存放在 Amazon S3，路径前缀 `s3://`。
> *For a model repository residing in Amazon S3, the path must be prefixed with s3://.*

### Azure Storage / Azure 存储
Triton 支持将模型仓库存放在 Microsoft Azure Storage，路径前缀 `as://`。
> *For a model repository residing in Azure Storage, the repository path must be prefixed with as://.*

---

## 来源文档

- Triton Overview: https://docs.nvidia.com/deeplearning/triton-inference-server/user-guide/docs/index.html
- Model Repository: https://docs.nvidia.com/deeplearning/triton-inference-server/user-guide/docs/user_guide/model_repository.html
- Model Configuration: https://docs.nvidia.com/deeplearning/triton-inference-server/user-guide/docs/user_guide/model_configuration.html
- Batchers: https://docs.nvidia.com/deeplearning/triton-inference-server/user-guide/docs/user_guide/batcher.html
- Model Execution: https://docs.nvidia.com/deeplearning/triton-inference-server/user-guide/docs/user_guide/model_execution.html
- Ensemble Models: https://docs.nvidia.com/deeplearning/triton-inference-server/user-guide/docs/user_guide/ensemble_models.html
