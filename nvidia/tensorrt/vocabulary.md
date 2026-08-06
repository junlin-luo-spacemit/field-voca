---
tags: [nvidia, tensorrt, vocabulary]
domain: tensorrt
last_updated: 2026-07-21
part: Overview ✅ / Inference Library ✅ / API Reference ✅
source: https://docs.nvidia.com/deeplearning/tensorrt/latest/
---

# TensorRT 词汇表

> 来源：https://docs.nvidia.com/deeplearning/tensorrt/latest/
> 版本：TensorRT 11.1.0 (Jun 2026)
> 格式：英文术语 → 中文译名 + 官方定义 + 例句

---

## 核心概念（Core Concepts）

### TensorRT / TensorRT 推理优化 SDK
NVIDIA 推理优化 SDK，将 PyTorch/TensorFlow/ONNX 等框架训练好的模型转换为针对 GPU 高度优化的推理引擎。支持混合精度（FP32/FP16/BF16/FP8/INT8/FP4/INT4）、动态形状、Transformer/LLM 专用优化。
> *NVIDIA TensorRT takes trained models from frameworks such as PyTorch, TensorFlow, and ONNX, and optimizes them for high-performance deployment.*

### inference / 推理
使用训练好的模型对新输入数据计算预测结果的过程（区别于训练）。TensorRT 专注于将推理阶段的延迟和吞吐优化到极致。
> *TensorRT is an SDK for optimizing and accelerating deep learning inference on NVIDIA GPUs.*

### build phase / 构建阶段
TensorRT 的第一阶段（通常离线执行）。Builder 接收模型定义，对其进行层融合、算子选择、精度校准等一系列优化，最终输出序列化的 engine（plan file）。
> *A build phase in which a builder compiles your network and selects the fastest available kernel for each layer on your target GPU.*

### runtime phase / 运行时阶段
TensorRT 的第二阶段。Runtime 反序列化 engine 并在 GPU 上执行推理。
> *A runtime phase in which a runtime loads the engine into your application and executes it on the GPU.*

### engine / 引擎（engine）
TensorRT 构建阶段的输出物，包含针对特定 GPU 优化后的网络图、编译好的 CUDA tactics、权重和执行调度。默认情况下 engine 绑定特定 TensorRT 版本和 GPU 型号。
> *The builder creates the engine in a serialized form called a plan, which can be deserialized immediately or saved to disk for later use.*

### plan / plan 文件
Engine 的序列化二进制格式，可保存到磁盘供后续部署使用。反序列化后即成为内存中的 engine 对象。
> *The output is a serialized binary called an engine (also referred to as a plan file).*

### Builder / 构建器
TensorRT 构建阶段的最高级接口。负责接收 NetworkDefinition + BuilderConfig，执行优化并生成 engine。
> *The highest-level interface for the build phase of TensorRT is the Builder.*

### Runtime / 运行时（Runtime）
TensorRT 运行时阶段的最高级接口。负责反序列化 plan 文件并创建 engine 对象。
> *The highest-level interface for the execution phase of TensorRT is the Runtime.*

### NetworkDefinition / 网络定义
描述模型结构的接口。最常见的填充方式是通过 ONNX parser 导入模型，也可用 `add...` 方法手动逐层构建。
> *The NetworkDefinition interface defines the model. The most common way to transfer a model to TensorRT is to export it from a framework in ONNX format.*

### BuilderConfig / 构建配置
指定 TensorRT 优化行为的接口：精度控制、workspace 大小限制、kernel 搜索策略、timing cache 等。
> *The BuilderConfig interface specifies how TensorRT should optimize the model.*

### ExecutionContext / 执行上下文
从 engine 创建的推理执行实例。包含推理过程中的所有状态（中间激活值等）。**一个 engine 可创建多个 context 并行执行**。
> *The ExecutionContext interface, created from the engine, is the main interface for invoking inference.*
> *You can have multiple contexts associated with a single engine and run them in parallel.*

### enqueueV3() / 异步推理入队
在 CUDA stream 上排队推理执行的 API 调用。是非阻塞的（asynchronous），控制权尽快返回给应用。完成需用 `cudaStreamSynchronize` 等待。
> *The required kernels are enqueued on a CUDA stream, and control is returned to the application as soon as possible.*

---

## 模型导入（Model Import）

### ONNX (Open Neural Network Exchange) / 开放神经网络交换格式
TensorRT 导入模型的主要中间格式。所有主流框架（PyTorch、TensorFlow、JAX）均可导出 ONNX。
> *TensorRT's primary means of importing a trained model from a framework is the ONNX interchange format.*

### ONNX parser / ONNX 解析器
TensorRT 自带的库，解析 ONNX 模型并填充 NetworkDefinition。支持 opset 9 及以上。
> *TensorRT ships with an ONNX parser library to assist in importing models.*

### opset / 算子集（opset）
ONNX 标准中对每个算子行为的版本化规范。ONNX 解析器向后兼容至 opset 9，推荐导出最新 opset。
> *Where possible, the parser is backward compatible to opset 9.*

### Torch-TensorRT / Torch-TensorRT 编译器
PyTorch-TensorRT 编译器，将 PyTorch 模块中可加速的子图编译为 TensorRT engine，其余部分由 PyTorch 原生执行。
> *PyTorch-TensorRT compiler that converts PyTorch modules into TensorRT engines. Subgraphs are accelerated through TensorRT while Torch executes the rest natively.*

### TriPy / TriPy
TensorRT 的 Python 逐步网络构建接口，以 Pythonic 方式逐层定义网络。

### MLIR-TensorRT / MLIR-TensorRT
基于 MLIR 编译器基础设施的 TensorRT 前端，支持 TensorFlow/JAX 等 MLIR 生态的框架。

### Polygraphy / Polygraphy 工具集
NVIDIA 提供的推理调试工具集，支持：多后端对比（TensorRT vs ONNX-Runtime）、精度分析、ONNX 子图提取和简化、tactic 故障隔离。
> *Polygraphy is a toolkit designed to assist in running and debugging deep learning models in TensorRT and other frameworks.*

### ONNX-GraphSurgeon / ONNX-GraphSurgeon
用于编辑 ONNX 图结构的工具：提取子图、插入/删除节点、常量折叠。搭配 Polygraphy 解决 parser 兼容性问题。

### constant folding / 常量折叠
将模型中编译时可确定的计算提前求值，简化图结构。推荐导出 ONNX 后用 Polygraphy 执行常量折叠作为第一步。
> *After exporting a model to ONNX, run constant folding using Polygraphy as a good first step.*

---

## 精度与量化（Precision & Quantization）

### mixed precision / 混合精度
在同一网络中使用不同数值精度的策略。TensorRT 支持 FP32/FP16/BF16/FP8/INT8/FP4/INT4 混合使用，在精度和性能之间取得平衡。
> *TensorRT optimizes them for high-performance deployment with support for mixed precision (FP32/FP16/BF16/FP8/INT8/FP4/INT4).*

### FP8 (E4M3) / FP8 浮点
8 位浮点：1 位符号 + 4 位指数 + 3 位尾数。需通过显式 Q/DQ 层进行转换。
> *8-bit floating point type with 1-bit for sign, 4-bits for exponent, 3-bits for mantissa.*

### FP4 (E2M1) / FP4 浮点
4 位浮点：1 位符号 + 2 位指数 + 1 位尾数。每字节打包 2 个元素。需通过显式 Q/DQ 层转换。
> *4-bit floating point type with 1-bit for sign, 2-bits for exponent, 1-bit for mantissa.*

### INT4 / INT4 整型
4 位有符号整数。用于 **weight-only quantization**（仅量化权重），计算前需反量化。每字节打包 2 个元素。
> *INT4 is used for weight-only-quantization. Requires dequantization before computing is performed.*

### E8M0 / E8M0 指数型
8 位纯指数浮点（无符号、无尾数）。仅用作 **MXFP8 量化**的 scale 类型。动态范围 [2⁻¹²⁷, 2¹²⁷]。
> *E8M0 is supported only as a scale type for MXFP8 quantization.*

### Q/DQ (Quantize / Dequantize) / 量化 / 反量化层
在 TensorRT 中进行低精度量化的显式机制。Q 层将高精度值压缩为低精度，DQ 层将低精度恢复为高精度以参与计算。
> *Conversion to/from INT8 type requires an explicit Q/DQ layer.*

### PTQ (Post-Training Quantization) / 训练后量化
模型训练完成后再进行量化。使用校准数据集确定各层的动态范围，无需重新训练。

### QAT (Quantization-Aware Training) / 量化感知训练
在训练过程中模拟量化误差，使模型在量化后仍保持精度。TensorRT 可直接导入 QAT 模型的 Q/DQ 节点。

### calibration / 校准
PTQ 中通过运行代表性数据集确定每层激活值范围的过程，用于设定 Q/DQ 层的 scale 值。

### strongly typed network / 强类型网络
TensorRT 11.0+ 唯一支持的模式（移除了弱类型）。网络中每个张量的精度由其类型标注和 Q/DQ 节点精确决定，Builder 不会自动插入类型转换。
> *Weak typing was removed in 11.0.*

### NVFP4 / NVFP4
NVIDIA 的 4 位浮点格式。TensorRT 11.1 支持 NVFP4 dual-GEMM fusion，可在 MoE/MLP 模块中融合 gate 和 up projection。

---

## 动态形状（Dynamic Shapes）

### dynamic shapes / 动态形状
允许在运行时改变输入尺寸（batch size、序列长度、图像分辨率等）。需配合 optimization profile 使用。
> *The builder can be configured to adjust the input dimensions at runtime.*

### OptimizationProfile / 优化配置文件
定义每个输入张量的 **min/opt/max** 三组形状。TensorRT 在 [min, max] 范围内选择通用 kernel，并针对 opt 做最优调参。
> *Specify one or more instances of OptimizationProfile containing a minimum and maximum shape for each input and an optimization point.*

### shape tensor / 形状张量
值代表张量形状信息的特殊张量（如 reshape 的目标形状）。动态形状场景下需要特殊处理。

### data-dependent output / 数据依赖输出
输出形状取决于输入数据值（而非仅取决于输入形状）的情况。如 NMS 输出数量不固定。

---

## 性能优化（Performance Optimization）

### layer fusion / 层融合
Builder 将多个相邻层合并为一个优化 kernel 的过程。如 Conv + BN + ReLU 融合为单个 kernel，减少 kernel launch 开销和显存读写。
> *The builder eliminates dead computations, folds constants, reorders, and combines operations to run more efficiently on the GPU.*

### kernel auto-tuning / kernel 自动调参
Builder 对每层的多种实现方案逐一计时（timing），为目标 GPU 选择最快的 kernel 实现。
> *The builder times algorithms to determine the fastest kernel for the parameters provided.*

### tactic / tactic（策略实现）
TensorRT 内部对同一层的不同 kernel 实现。Builder 在构建阶段尝试所有可用 tactic 并选最快的。
> *It also has multiple implementations for each layer with varying data formats. Then, it computes an optimal schedule to execute the model.*

### timing cache / 计时缓存
缓存 Builder 的 kernel 计时结果，加速后续构建。可序列化供不同构建复用。
> *You can also control how the builder searches for kernels and cached search results for use in subsequent runs.*

### dead computation elimination / 死计算消除
Builder 自动移除不影响最终输出的多余计算。

### trtexec / trtexec 命令行工具
TensorRT 自带的命令行工具：benchmark 网络、生成 engine、生成 timing cache。用于快速验证模型性能。
> *trtexec allows you to use TensorRT without developing your application.*

### Global Performance Tuner / 全局性能调参器
TensorRT 11.1 新增。自动化 `trtexec` 构建路线搜索，探索 Builder 内部调参旋钮，benchmark 候选 engine，验证精度后选择最快的有效路线。

### workspace / workspace（工作空间）
Builder/Runtime 执行层实现时的临时 GPU 显存。默认为设备全部显存，生产环境应通过 `setMemoryPoolLimit` 显式限制。
> *You can control the maximum amount of temporary memory through the memory pool limits of the builder config.*

---

## 内存管理（Memory Management）

### persistent memory / 持久内存
ExecutionContext 创建时分配的设备内存。某些层实现需要的状态数据（如卷积的 edge mask），其生命周期等于 context。

### enqueue memory / 入队内存（activation + scratch）
推理执行过程中存放中间结果（activation）和临时计算空间（scratch）的内存。TensorRT 通过生命周期不重叠的张量复用内存块来优化用量。
> *Enqueue memory is used to hold intermediate results while processing the network.*

### activation memory / 激活内存
存放网络中间层输出的内存。TensorRT 让生命周期不重叠的 activation 共享同一块显存。

### scratch memory / 临时计算内存
层实现内部需要的额外临时空间。可通过 `kWORKSPACE` 池限制上界。

### IGpuAllocator / GPU 分配器接口
用户可实现此接口接管 TensorRT 的所有设备内存分配，用于统一内存管理、预分配池或强制上限。
> *You can attach an implementation of TensorRT's IGpuAllocator interface to the builder or runtime and manage device memory yourself.*

### weight streaming / 权重流式加载
TensorRT 将网络权重从 host 内存按需流式传到 device 内存（而非启动时一次性加载）。允许运行超出 GPU 显存的模型，代价是延迟增加。
> *TensorRT can be configured to stream the network's weights from host memory to device memory during network execution.*

### CUDA lazy loading / CUDA 延迟加载
CUDA 特性（`CUDA_MODULE_LOADING=LAZY`）：按需加载 CUDA 模块，显著减少 TensorRT 初始化时的 GPU/host 内存峰值。
> *CUDA lazy loading can significantly reduce the peak GPU and host memory usage of TensorRT and speed up TensorRT initialization.*

### L2 persistent cache / L2 持久缓存
Ampere+ 架构支持。通过 `setPersistentCacheLimit` 为每个 ExecutionContext 保留 L2 cache 空间，减少 DRAM 访问。
> *TensorRT uses this to retain cache activations, reducing DRAM traffic and power consumption.*

---

## 插件与扩展（Plugins & Extensions）

### plugin / 插件
TensorRT 的自定义层机制。当 ONNX parser 遇到 TensorRT 不原生支持的算子时，可从 PluginRegistry 查找并使用用户注册的插件。
> *TensorRT has a Plugin interface that allows applications to implement operations that TensorRT does not support natively.*

### Plugin V3 / 插件 V3 接口
TensorRT 最新的插件 API 版本，提供更完善的注册、数据格式协商和动态形状支持。

### PluginRegistry / 插件注册表
全局注册表，存放所有可用的 plugin creator。注册/注销操作是线程安全的。
> *Registering and deregistering plugins (are thread-safe).*

---

## 高级功能（Advanced Features）

### refitting / 权重热更新（refit）
在不重新构建 engine 的情况下更新权重。适用于权重频繁变化但网络结构不变的场景（如强化学习、增量训练）。
> *When building an engine, you can specify that its weights can be updated later.*

### engine compatibility / 引擎兼容性
默认 engine 绑定特定 TensorRT 版本 + GPU。通过 version compatibility / hardware compatibility 机制可放宽限制。
> *By default, TensorRT engines are specific to both the TensorRT version and the GPU on which they were created.*

### version compatibility / 版本兼容性
让 engine 可在不同 TensorRT 版本间使用。需在构建时启用，并使用 lean runtime 或 dispatch runtime。

### hardware compatibility / 硬件兼容性
让 engine 可在不同 GPU 型号上运行（同架构或跨架构）。构建时需启用对应 flag。

### lean runtime / 精简运行时
较小的运行时库（`libnvinfer_lean`），仅包含执行 version-compatible engine 所需的最小代码集。部署包更小，但某些 operator 不支持。
> *The lean runtime library is smaller than the default library and contains only the code necessary to run a version-compatible engine.*

### dispatch runtime / 调度运行时
极小的 shim 库（`libnvinfer_dispatch`），可加载不同版本的 lean runtime 并转发调用，实现跨版本部署。
> *The dispatch runtime is a small shim library that can load a lean runtime and redirect calls.*

### DLA (Deep Learning Accelerator) / 深度学习加速器
NVIDIA SoC（如 Jetson）上的专用推理处理器。支持 TensorRT 层的子集。**TensorRT 11.0/11.1 暂不支持 DLA**。
> *TensorRT supports NVIDIA's Deep Learning Accelerator (DLA). DLA is not supported in TensorRT 11.0 or 11.1.*

### ILoop / 循环网络接口
TensorRT 中实现循环（loop）子图的接口，用于 RNN 等迭代计算。

### IIfConditional / 条件分支接口
TensorRT 中实现数据依赖分支的接口。

### multi-device inference / 多设备推理
将单个网络拆分到多 GPU 执行。通过 DistCollective 层使用 NCCL 进行 AllReduce/AllGather 等集合通信。Ampere+ 支持。
> *Multi-device inference provides two capabilities: DistCollective and Multi-device attention.*

### DistCollective / 分布式集合通信层
TensorRT 中的分布式集合操作层：AllReduce、AllGather、Broadcast、Reduce、ReduceScatter、AllToAll 等。底层使用 NCCL。

---

## Transformer 优化（Transformer Optimizations）

### fused attention / 融合注意力
TensorRT 对 Multi-Head Attention 的整体融合实现，避免生成完整 attention matrix，显著降低显存占用和计算延迟。

### KV cache / KV 缓存
Transformer 自回归生成中缓存 Key/Value 向量的机制。TensorRT 提供专用支持以避免重复计算。

### MoE (Mixture of Experts) / 混合专家
Transformer 中的条件计算架构：每个 token 仅激活部分"专家"子网络。TensorRT 提供 MoE 专用融合优化。

### NVFP4 dual-GEMM fusion / NVFP4 双 GEMM 融合
TensorRT 11.1 在 SM121（DGX Spark）上的优化：将 MoE/MLP 模块的 gate projection 和 up projection 两个 GEMM 融合为一次执行。

---

## 线程与确定性（Threading & Determinism）

### thread safety / 线程安全性
TensorRT 对象通常**不是**线程安全的。推荐模式：一个 thread 对应一个 ExecutionContext；多个 thread 可各自从同一 engine 创建独立 context。
> *TensorRT objects are generally not thread-safe. The expected runtime concurrency model is that different threads operate in different execution contexts.*

### determinism / 确定性
构建好的 engine 对相同输入产生相同输出（IFillLayer RANDOM 和 IScatterLayer 重复索引除外）。不同构建间由于 timing noise 可能选不同 tactic 导致微小数值差异。
> *After an engine has been built, it is deterministic. Providing the same input in the same runtime environment produces the same output.*

### timing noise / 计时噪声
Builder 选 tactic 时的计时受 GPU 负载和时钟波动影响。不同构建可能选出不同实现，导致跨构建的微小精度差异。

### Editable Timing Cache / 可编辑计时缓存
允许强制 Builder 对指定层选择特定 tactic，确保不同构建间的可复现性。

### distributive independence / 分布独立性
`BuilderFlag::kDISTRIBUTIVE_INDEPENDENCE`：保证分布轴上相同的输入产生相同的输出。代价：不支持多 profile，可能性能下降。

---

## 对象生命周期（Object Lifetimes）

### factory pattern / 工厂模式
TensorRT 使用工厂模式：某些对象创建并拥有其他对象。工厂对象的生命周期必须覆盖其产物。
> *TensorRT uses a factory pattern where some objects create and own other objects.*

### Logger / 日志记录器
用户实现的接口，贯穿 TensorRT 全生命周期，接收诊断和信息消息。**必须线程安全**。
> *The logger is used for diagnostics and informational messages. The implementation must also be thread-safe.*

### ErrorRecorder / 错误记录器
用户实现的接口，附加到 API 对象以接收相关错误。子对象继承父对象的 recorder。

### trust boundary / 信任边界
Engine 文件等价于可执行代码——仅反序列化你自己构建的或来自可信渠道的 engine，不要加载不可信来源的 engine。
> *Deserializing an engine from an untrusted source is equivalent to executing untrusted native code.*

---

## 部署与生态（Deployment & Ecosystem）

### NVIDIA Triton Inference Server / Triton 推理服务器
更高级的推理服务框架，支持 CPU/GPU 跨平台推理、模型管理、REST/gRPC 端点。可集成 TensorRT engine。

### Model Optimizer (ModelOpt) / 模型优化器
NVIDIA 统一的量化/剪枝/蒸馏库。替代已弃用的 PyTorch/TensorFlow Quantization Toolkit。

### Nsight Systems / Nsight Systems 性能分析器
NVIDIA 性能分析工具，可与 TensorRT 集成进行端到端推理性能分析。

### Nsight Deep Learning Designer / Nsight 深度学习设计器
可视化 ONNX 模型编辑、性能分析和 TensorRT engine 构建的 IDE。

### NVIDIA DALI / DALI 数据加载库
高性能数据预处理流水线库（图像/音频/视频）。TensorRT 推理可作为 DALI pipeline 中的自定义算子集成。

### MIG (Multi-Instance GPU) / 多实例 GPU
Ampere+ 硬件级 GPU 分区。将物理 GPU 划分为多个隔离实例，每个有独立显存和计算切片。提供最强的多租户故障隔离。
> *MIG partitions the GPU into hardware-isolated instances. Each instance has its own memory slice, its own compute slice, and its own error domain.*

### Green Context / 绿色上下文
CUDA 13.1+ 的轻量级上下文，可分配指定数量的 SM 和 Work Queue。比 MIG 轻量但不提供内存隔离。
> *Green contexts are a lighter-weight alternative to MIG to keep CUDA workloads from affecting each other's inference compute usage.*

### semantic versioning / 语义化版本
TensorRT 版本号遵循 SemVer 2.0.0：MAJOR 不兼容变更、MINOR 向后兼容新功能、PATCH 向后兼容修复。
> *TensorRT version number (MAJOR.MINOR.PATCH) follows Semantic Versioning 2.0.0.*

---

## 错误隔离（Error Isolation）

### sticky CUDA error / 粘性 CUDA 错误
一旦设备上发生异步 CUDA 错误，同进程同设备的后续 CUDA 调用持续失败，直到进程重启或设备 reset。单个 context 出错可毒害同进程所有 context。
> *Once an asynchronous error has occurred on a device, subsequent CUDA calls on the same device and process continue to fail.*

### process-level isolation / 进程级隔离
唯一可靠的软件恢复手段：终止并重启推理进程（由 systemd/K8s liveness probe/Triton 等监管）。
> *The only safe software-only recovery is to terminate and restart the inference process.*

### CUDA MPS / CUDA 多进程服务
允许多个进程共享单 GPU 的机制。可用作多租户间的故障边界，但隔离弱于 MIG。

---

## API Reference（Python/C++ API 核心类）

> 来源：TensorRT 11.1.0 Standard Python API Documentation

### 基础类型（Foundational Types）

### DataType / 数据类型枚举
表示张量/权重的数值类型（FLOAT/HALF/INT8/INT32/BOOL/FP8/BF16/INT4/FP4/E8M0 等）。`nptype()` 可转换为对应 NumPy 类型。

### Weights / 权重容器
包装原始权重数据（数组指针 + 数据类型 + 元素个数）的结构，配合 `WeightsRole` 标注角色（kernel/bias/shift/scale/constant 等）。

### Dims / 维度
描述张量形状的整数序列。`Dims2/DimsHW/Dims3/Dims4` 为常用维度的便捷子类，`volume()` 计算元素总数。

### IHostMemory / 主机内存句柄
Builder 序列化 engine 后返回的主机内存缓冲区封装，支持 buffer protocol 直接读取二进制内容。

### 核心类（Core）

### ILogger / Logger / 日志接口与默认实现
`ILogger` 是用户需要实现的抽象接口（含 `Severity` 分级：INTERNAL_ERROR/ERROR/WARNING/INFO/VERBOSE）；`Logger` 是官方提供的默认实现。贯穿 Builder/Runtime 全生命周期，**必须线程安全**。

### IProfiler / Profiler / 性能剖析接口
通过 `report_layer_time()` 回调收集每层执行耗时，用于逐层性能分析。

### IOptimizationProfile / 优化配置文件接口
`set_shape()`/`get_shape()` 设置/读取每个输入张量的 min/opt/max 三组形状；`set_shape_input()` 用于 shape tensor 输入。

### IBuilderConfig / 构建配置接口
核心配置项：`set_memory_pool_limit()`（workspace/DLA 等内存池上限）、`set_flag()`/`BuilderFlag`（如 FP16/INT8/REFIT/STRICT_TYPES 等开关）、`set_tactic_sources()`（cuBLAS/cuDNN 等 tactic 来源）、`add_optimization_profile()`、`set_timing_cache()`、`HardwareCompatibilityLevel`（硬件兼容级别）、`TilingOptimizationLevel`（tiling 优化级别）、`EngineCapability`（STANDARD/SAFETY/DLA_STANDALONE）。

### Builder / 构建器类
`create_network()` 创建 NetworkDefinition，`create_builder_config()` 创建配置，`create_optimization_profile()` 创建 profile，`build_serialized_network()` 一步构建出序列化 engine（推荐用法，取代旧版分步 `build_engine()`），`is_network_supported()` 预检查网络是否可构建。`NetworkDefinitionCreationFlag` 控制网络创建标志（如 STRONGLY_TYPED）。

### IProgressMonitor / 构建进度监视器
`phase_start()`/`step_complete()`/`phase_finish()` 回调，用于长时间构建过程中展示进度（如百分比条）。

### IDebugListener / 调试张量监听器
`process_debug_tensor()` 在推理时捕获被 `mark_debug()` 标记的中间张量值，用于调试。

### ICudaEngine / CUDA 引擎接口
构建完成的核心产物类。关键方法：`create_execution_context()`（创建执行上下文）、`get_tensor_shape()`/`get_tensor_dtype()`/`get_tensor_mode()`（查询 I/O 张量元信息）、`create_engine_inspector()`（引擎结构自检）、`serialize()`/`serialize_with_config()`（重新序列化）、`get_weight_streaming_automatic_budget()`（权重流式加载预算查询）。

### IExecutionContext / 执行上下文接口
`execute_async_v3()`（推荐的异步入队接口）、`set_input_shape()`（设置动态输入形状）、`set_tensor_address()`（绑定输入/输出设备指针）、`set_optimization_profile_async()`（切换 profile）、`IOutputAllocator` + `set_output_allocator()`（应对 data-dependent shape 的动态输出内存分配）、`infer_shapes()`（形状推理校验）。

### Runtime / 运行时反序列化器
`deserialize_cuda_engine()` 从 plan 二进制还原为 `ICudaEngine`；`load_runtime()` 加载 lean runtime 用于跨版本兼容部署；`TempfileControlFlag` 控制临时文件生成行为。

### IRuntimeConfig / 运行时配置接口
TensorRT 较新引入的运行时级配置，`ExecutionContextAllocationStrategy` 控制 context 显存分配策略（静态预分配 vs 按需分配）。

### Refitter / 权重热更新器
实现 refitting 机制的操作类：`get_missing_weights()`（查询待更新权重清单）、`set_named_weights()`/`set_weights()`（提供新权重）、`refit_cuda_engine()`/`refit_cuda_engine_async()`（执行更新）、`get_weights_location()`（查询权重当前所在设备）。

### IErrorRecorder / 错误记录器接口
`report_error()`/`get_error_code()`/`get_error_desc()`/`has_overflowed()`，用于集中捕获 API 调用错误，可附加到 Builder/Runtime/Engine。

### ITimingCache / 计时缓存接口
`serialize()`/`combine()`（合并多份缓存）/`query()`（查询已有计时结果），用于跨构建复用 tactic 计时数据，加速重复构建。

### IGpuAllocator / GPU 分配器接口
`allocate()`/`allocate_async()`/`deallocate()`/`reallocate()`，用户实现后可接管 TensorRT 全部设备内存分配；`AllocatorFlag` 控制分配行为（如 RESIZABLE）。

### EngineInspector / 引擎自检器
`get_engine_information()`/`get_layer_information()` 以 JSON 形式导出 engine 内部结构、选中的 tactic、数据格式等，用于调试和性能分析。

### ISerializationConfig / 序列化配置接口
`SerializationFlag` 控制序列化行为（如 EXCLUDE_WEIGHTS，用于生成不含权重的 engine 供 refit 场景使用）。

### 网络与层（Network / Graph）

### INetworkDefinition / 网络定义接口
除 `add_input()`/`mark_output()` 基础方法外，提供 ~50 个 `add_*()` 方法逐层构建网络（如 `add_convolution_nd`、`add_attention`/`add_attention_v2`、`add_moe`、`add_rotary_embedding`、`add_kv_cache_update`、`add_dist_collective`、`add_dynamic_quantize`/`add_dynamic_quantize_v2`）。`mark_weights_refittable()` 标记权重可热更新；`mark_debug()` 标记调试张量。

### ITensor / 张量句柄
表示网络中的一条边（数据流）。`get_dimension_name()`/`set_dimension_name()` 为动态维度命名，便于多个张量间共享同一符号维度。

### ILayer / LayerType / 层基类与层类型枚举
所有具体层的公共基类，`get_input()`/`set_input()`/`get_output()` 访问层的输入输出张量；`LayerType` 枚举列出所有层种类。

### 关键层类型速查（部分）
| 层接口 | 用途 |
|--------|------|
| IConvolutionLayer / IDeconvolutionLayer | 卷积 / 反卷积 |
| IElementWiseLayer | 逐元素运算（加减乘除等，`ElementWiseOperation`） |
| IMatrixMultiplyLayer | 矩阵乘法（`MatrixOperation`：NONE/TRANSPOSE/VECTOR） |
| IShuffleLayer | reshape/transpose/permute 组合（`Permutation`） |
| ISliceLayer / IPaddingLayer | 切片 / 填充 |
| IAttention / IRotaryEmbeddingLayer / IKVCacheUpdateLayer | Transformer 专用：融合注意力 / RoPE / KV cache 更新 |
| IMoELayer | 混合专家层，`set_gated_weights()`/`set_swiglu_params()` 配置门控与激活 |
| IDistCollectiveLayer | 分布式集合通信（`CollectiveOperation`：AllReduce 等） |
| IQuantizeLayer / IDequantizeLayer / IDynamicQuantizeLayer | Q/DQ 量化层（动态量化为 11.x 新增） |
| ILoop / IRecurrenceLayer / IIteratorLayer | 循环子图构建 |
| IIfConditional / IConditionLayer | 条件分支子图构建 |
| INMSLayer / INonZeroLayer | 数据依赖输出的典型层（NMS 输出数量不固定） |
| IPluginV2Layer / IPluginV3Layer | 自定义插件层的网络内表示 |

### 插件系统（Plugin）

### IPluginCreator / IPluginCreatorV3One / PluginRegistry
`IPluginCreator`（V2 系）与 `IPluginCreatorV3One`（V3 系）负责根据 `PluginFieldCollection` 创建插件实例；`IPluginRegistry` 提供 `register_creator()`/`get_creator()`/`load_library()` 等全局注册管理，`get_plugin_registry()` 获取单例。

### IPluginV2DynamicExt / 插件 V2 动态形状扩展接口
早期支持动态形状的插件基类：`get_output_dimensions()`（形状推导）、`supports_format_combination()`（格式协商）、`enqueue()`（执行入口）、`get_workspace_size()`（临时显存声明）。

### IPluginV3 / 插件 V3 接口族
最新插件接口，拆分为多个 capability：`IPluginV3OneCore`（基础信息）、`IPluginV3OneBuild`（构建期：形状/数据类型推导、tactic 枚举 `get_valid_tactics()`）、`IPluginV3OneRuntime`（运行期：`enqueue()`、`on_shape_change()`、`set_tactic()`）。`IExprBuilder`/`IDimensionExpr` 用于以符号表达式描述输出形状（支持 data-dependent shape，如 `declare_size_tensor()`）。

### tensorrt.plugin（Python 原生插件模块）
TensorRT Python API 提供的轻量插件开发模块，无需实现完整 C++ 接口：
- `@tensorrt.plugin.register`：声明插件的形状/类型推导规则
- `@tensorrt.plugin.impl`：提供 JIT 编译的核函数实现
- `@tensorrt.plugin.aot_impl`：提供预编译（Ahead-of-Time）核函数实现，返回 `KernelLaunchParams`
- `@tensorrt.plugin.autotune`：定义多 tactic 自动调优候选（`AutoTuneCombination`）
- `Tensor`/`TensorDesc`：插件内访问张量数据指针、形状、dtype、stride 等
- `Shape`/`ShapeExpr`：符号化形状表达（`is_dynamic`/`is_size_tensor`/`numel()`）

### ONNX 解析器（Onnx Parser）

### OnnxParser / ONNX 解析器类
`parse()`/`parse_from_file()`（从内存或文件解析）、`supports_operator()`（查询算子是否受支持）、`is_subgraph_supported()`（子图支持性检查，用于混合执行场景）、`get_used_vc_plugin_libraries()`（查询 version-compatible engine 依赖的插件库）。

### OnnxParserRefitter / ONNX 层面权重热更新器
直接基于新的 ONNX 模型对已构建 engine 执行 refit（`refit_from_file()`/`refit_model_proto()`），无需手动枚举权重名称，简化 refitting 工作流。

### ParserError / OnnxParserFlag
`ParserError` 携带节点名/算子类型/文件/行号等详细诊断信息；`OnnxParserFlag` 控制解析行为（如 NATIVE_INSTANCENORM）。

---

*词条总计：~85（Overview + Inference Library）+ ~45（API Reference）= ~130*


---

## TensorRT Developer Guide (Inference Library) 词汇表

> 来源：TensorRT Inference Library Documentation（2026-06 更新）

### DG-1 量化与精度控制

| 英文 | 中文 | 说明 |
|------|------|------|
| Quantization | 量化 | 用低精度数据类型表示浮点值，减小模型体积 |
| Symmetric Quantization | 对称量化 | 以零为中心映射，仅需 scale 无需 zero-point |
| Post-Training Quantization (PTQ) | 训练后量化 | 模型训练完成后，用校准数据集确定 scale |
| Quantization-Aware Training (QAT) | 量化感知训练 | 训练中模拟量化误差，学习更鲁棒的权重 |
| Calibration | 校准 | 用代表性数据集统计激活值范围以确定量化 scale |
| Q/DQ Layer | 量化/反量化层 | 显式在网络中插入 Quantize/Dequantize 节点 |
| Explicit Quantization | 显式量化 | 用 Q/DQ 节点标注量化边界，TRT 据此融合 |
| Dynamic Quantization | 动态量化 | 运行时计算 scale（非提前校准） |
| Weight-Only Quantization | 仅权重量化 | 只量化权重（INT4/FP4），激活保持高精度 |
| Scale Factor | 缩放因子 | 浮点与量化值之间的映射系数 |
| E8M0 | E8M0 格式 | 8位纯指数格式，用作 MXFP8 的 scale 类型 |
| MXFP8 | 微缩放 FP8 | 使用 E8M0 scale 的分块 FP8 量化方案 |
| Strongly Typed Network | 强类型网络 | 每层精度由用户显式指定，TRT 不自动选择（11.0+ 唯一模式） |
| Precision Control | 精度控制 | Builder 级别设定允许的计算精度和约束 |
| Algorithm Selection | 算法选择 | Builder 从多个 kernel 实现中择优 |
| Reproducible Build | 可复现构建 | 固定算法选择以保证逐位一致 |
| Model Optimizer | 模型优化器 | NVIDIA 统一量化/剪枝/蒸馏工具库 |

### DG-2 动态形状

| 英文 | 中文 | 说明 |
|------|------|------|
| Dynamic Shape | 动态形状 | 推理时输入维度可变（batch/seq_len/H/W） |
| Optimization Profile | 优化配置 | 为动态维度指定 min/opt/max 三组值 |
| Shape Tensor | 形状张量 | 内容为维度值的特殊张量（非数据张量） |
| Execution Tensor | 执行张量 | 承载实际计算数据的普通张量 |
| Named Dimensions | 命名维度 | 为动态维度赋名以约束多输入间的关系 |
| IAssertionLayer | 断言层 | 在网络中声明维度约束条件 |
| Dynamically Shaped Output | 动态输出 | 输出形状依赖于输入数据内容 |
| Runtime Dimensions | 运行时维度 | 用 -1 作占位符指定的可变维度 |

### DG-3 自定义插件 (Plugin V3)

| 英文 | 中文 | 说明 |
|------|------|------|
| Plugin | 插件 | 扩展 TRT 的自定义层实现 |
| IPluginV3 | 插件 V3 接口 | 当前推荐的插件基类（取代 V2） |
| IPluginCreatorV3One | 插件创建器 V3 | 负责实例化插件对象的工厂类 |
| Plugin Registry | 插件注册表 | 全局注册表，ONNX Parser 从中查找插件 |
| initLibNvInferPlugins | 初始化标准插件 | 加载 TRT 自带插件库 |
| libnvinfer_plugin | 插件共享库 | TRT 标准插件动态库 |
| Timing Cache (Plugin) | 插件计时缓存 | 缓存插件 tactic 计时结果加速后续构建 |
| Custom Tactic | 自定义策略 | 插件提供多种 kernel 实现供 Builder 择优 |
| Plugin Shared Library | 插件共享库 | 独立 .so/.dll 形式分发的插件 |

### DG-4 Transformer 优化

| 英文 | 中文 | 说明 |
|------|------|------|
| Fused Attention | 融合注意力 | 将 QKV matmul + softmax + V matmul 合为单一 kernel |
| IAttention Layer | 注意力层 | TRT 原生 Attention 算子（含 fused 路径） |
| KV Cache | KV 缓存 | 存储历史 Key/Value 避免 LLM 重复计算 |
| Paged KV Cache | 分页 KV 缓存 | 按页管理 KV，减少内存碎片 |
| Rotary Position Embedding (RoPE) | 旋转位置编码 | 相对位置编码方式，LLaMA/GPT-NeoX 使用 |
| Mixture of Experts (MoE) | 混合专家 | 稀疏激活架构，每 token 仅路由到 k 个专家 |
| Multi-Device Attention | 多设备注意力 | KV 序列跨 GPU 分割的上下文并行方案 |
| Padded Format | 填充格式 | batch 内所有序列填充至统一长度 |
| Packed (Ragged) Format | 紧凑格式 | 变长序列紧密拼接，附带偏移数组 |
| Context Parallelism | 上下文并行 | 长序列拆分到多 GPU 并行处理注意力 |

### DG-5 引擎高级特性

| 英文 | 中文 | 说明 |
|------|------|------|
| Engine Compatibility | 引擎兼容性 | Plan 文件的版本/硬件/跨平台兼容规则 |
| Version Compatibility | 版本兼容 | 允许 Runtime 跨 TRT minor 版本加载引擎 |
| Hardware Compatibility | 硬件兼容 | 允许引擎在同代不同 GPU 间移植 |
| Cross-Platform Compatibility | 跨平台兼容 | x86 ↔ ARM 引擎互通 |
| Refitting | 权重刷新 | 运行时更新引擎权重而不重新构建 |
| Weight Stripping | 权重剥离 | 构建时不嵌入权重，部署时再 refit |
| Weight Streaming | 权重流式 | 从主机内存按需流入 GPU，支持超大模型 |
| Timing Cache | 计时缓存 | 缓存 kernel 择优结果加速后续构建 |
| Builder Optimization Level | 构建优化等级 | 控制 Builder 搜索深度（0-5），影响构建时间与性能 |
| Lean Runtime | 精简运行时 | 面向部署的最小化 TRT runtime 库 |
| Engine Inspector | 引擎检查器 | 查看引擎内层、精度、内存等详细信息 |
| Debug Tensors | 调试张量 | 标记中间张量为输出用于数值调试 |

### DG-6 性能优化

| 英文 | 中文 | 说明 |
|------|------|------|
| Layer Fusion | 层融合 | Builder 将多层合并为单一 kernel 减少内存开销 |
| Pointwise Fusion | 逐元素融合 | 连续逐元素操作合并为一个 kernel |
| Q/DQ Fusion | Q/DQ 融合 | 将量化/反量化节点融入相邻计算层 |
| Tensor Core Alignment | 张量核对齐 | I/O 维度对齐（8/16/32）以启用 TC 加速 |
| CUDA Graphs (TRT) | CUDA 图 | 将推理调用链录制为图减少启动开销 |
| Multi-Stream Inference | 多流推理 | 多 CUDA Stream 并行执行不同 batch/context |
| trtexec | TRT 命令行工具 | 基准测试/引擎构建/计时缓存生成 |
| Polygraphy | 调试工具包 | 多后端对比推理、ONNX 编辑、精度诊断 |
| ONNX-GraphSurgeon | ONNX 图编辑器 | 程序化修改 ONNX 计算图 |
| Nsight Deep Learning Designer | DL 设计器 | ONNX 可视编辑 + TRT 性能分析 IDE |
| Global Performance Tuning | 全局性能调优 | 自动化 builder route 搜索进一步压榨性能 |

### DG-7 控制流与 DLA

| 英文 | 中文 | 说明 |
|------|------|------|
| ILoop Layer | 循环层 | 支持 RNN 等迭代子图 |
| IIfConditional Layer | 条件层 | 数据依赖的 if-else 分支 |
| Deep Learning Accelerator (DLA) | 深度学习加速器 | NVIDIA SoC 上的专用推理引擎 |
| DLA Standalone Mode | DLA 独立模式 | 不依赖 GPU 独立执行的 DLA 工作模式 |
| Capture and Replay | 捕获与回放 | 录制 TRT API 调用序列用于调试复现 |

### DG-8 多设备推理

| 英文 | 中文 | 说明 |
|------|------|------|
| Multi-Device Inference | 多设备推理 | 单模型跨多 GPU 并行执行 |
| IDistCollectiveLayer | 分布式集合层 | AllReduce/AllGather/ReduceScatter 等通信原语 |
| NCCL Communicator | NCCL 通信器 | NVIDIA 集合通信库的通信器对象 |
| Rank | 秩 | 多设备场景中每个 GPU 的编号 |
| Tensor Parallelism | 张量并行 | 将算子内部矩阵分割到多 GPU |
| Pipeline Parallelism | 流水线并行 | 模型不同层分配到不同 GPU 流水执行 |
