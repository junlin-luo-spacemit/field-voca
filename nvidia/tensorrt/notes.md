---
tags: [nvidia, tensorrt, notes]
domain: tensorrt
last_updated: 2026-07-03
source: https://docs.nvidia.com/deeplearning/tensorrt/latest/
---

# TensorRT Overview 精读笔记

> TensorRT 11.1.0 | 文档更新 Jun 2026

---

## 一句话总结

TensorRT = **离线构建（优化）** + **在线推理（执行）** 的两阶段 SDK。Builder 把训练好的模型编译成针对目标 GPU 极致优化的 engine；Runtime 加载 engine 执行推理。

---

## 核心工作流

```
训练框架 (PyTorch/TF/JAX)
    │ export
    ▼
  ONNX 模型
    │ parse
    ▼
NetworkDefinition ──► BuilderConfig
    │                    │
    └──► Builder ◄───────┘
           │
           │ 优化: 层融合 / kernel auto-tune / 精度校准 / 常量折叠
           ▼
      Engine (plan file) ← 序列化到磁盘
           │
           │ deserialize
           ▼
       Runtime
           │
           │ createExecutionContext()
           ▼
   ExecutionContext ──► enqueueV3(stream) ──► 推理结果
```

---

## Builder 做了什么（构建阶段优化）

| 优化 | 说明 |
|------|------|
| Dead computation elimination | 移除不影响输出的节点 |
| Constant folding | 预计算编译时可确定的值 |
| Layer fusion | Conv+BN+ReLU → 单 kernel |
| Kernel auto-tuning | 对每层的多种 tactic 计时，选最快 |
| Precision calibration | 确定 INT8/FP8 scale |
| Format selection | 选择最优数据布局 (NCHW/NHWC/...) |
| Memory planning | 激活内存复用 |

**关键特征**：构建过程慢（分钟级），但只需做一次；结果 engine 是 GPU-specific 的。

---

## 精度支持矩阵

| 类型 | 位宽 | 特征 |
|------|------|------|
| FP32 | 32 | 默认精度 |
| FP16 | 16 | 最广泛的加速，Tensor Core |
| BF16 | 16 | 更大动态范围，Ampere+ |
| FP8 (E4M3) | 8 | 显式 Q/DQ，Hopper+ |
| INT8 | 8 | 显式 Q/DQ，需校准 |
| FP4 (E2M1) | 4 | 显式 Q/DQ，2 元素/字节 |
| INT4 | 4 | 仅用于 weight-only quant |
| E8M0 | 8 | 仅作为 MXFP8 scale 类型 |

**TensorRT 11.0 起只有强类型网络**（Weak typing 已移除）。

---

## 动态形状

- OptimizationProfile: 定义 min/opt/max 三组形状
- TensorRT 为 [min, max] 范围选通用 kernel，针对 opt 做极致优化
- 一个 engine 可有多个 profile，运行时按需切换
- data-dependent output：输出形状取决于输入值（如 NMS）

---

## 内存模型

### 构建阶段
- workspace: 层实现计时的临时 GPU 内存（默认=全部显存，**生产环境必须限制**）
- host 内存：ONNX 解析 + 权重副本（可能很大）

### 运行时阶段
| 类别 | 生命周期 | 说明 |
|------|----------|------|
| 权重内存 | engine 反序列化后常驻 | ≈ plan 文件大小 |
| persistent memory | context 全生命周期 | 某些层实现需要的状态 |
| enqueue memory | 推理执行时 | activation + scratch，可跨 context 复用 |

### 高级控制
- `IGpuAllocator`: 接管所有 GPU 分配
- `createExecutionContextWithoutDeviceMemory()`: 手动管理 enqueue memory
- Weight Streaming: 权重按需从 host 流式到 device

---

## 线程模型

| 操作 | 线程安全? |
|------|-----------|
| Runtime 只读操作 | ✅ |
| Engine 只读 + 创建 context | ✅ |
| Plugin 注册/注销 | ✅ |
| ExecutionContext 使用 | ❌ 一个 thread 一个 context |
| Builder 共享 | ⚠️ 技术上可以但计时互扰 |
| IRefitter | ❌ |

**推荐模式**: 一个 engine → 多个 thread 各创建独立 ExecutionContext。

---

## 确定性保证

- 同一 engine + 同输入 → 相同输出 ✅
- 不同构建之间 → 可能选不同 tactic → 微小差异
- 修复方法: Editable Timing Cache 强制选 tactic
- 例外: IFillLayer(RANDOM), IScatterLayer(重复索引)

---

## 工具链

| 工具 | 用途 |
|------|------|
| **trtexec** | CLI benchmark / engine 生成 / timing cache |
| **Polygraphy** | 多后端对比 / 精度调试 / ONNX 编辑 |
| **ONNX-GraphSurgeon** | ONNX 图编辑 |
| **Model Optimizer** | 量化/剪枝/蒸馏 |
| **Nsight Systems** | 端到端性能分析 |
| **Triton** | 生产推理服务 |

---

## 部署注意事项

1. **Engine 绑定**: 默认绑定 TensorRT 版本 + GPU 型号 + OS
2. **Version Compatibility**: lean/dispatch runtime 实现跨版本
3. **Hardware Compatibility**: 跨 GPU 型号（同/跨架构）
4. **Trust Boundary**: engine = 可执行代码，绝不反序列化不可信来源
5. **Error Isolation**: sticky CUDA error → 进程级隔离 / MPS / MIG
6. **Multi-tenant**: workspace cap + IGpuAllocator + MIG 三层防护

---

## TensorRT 11.1 新增

- CUDA 13.3 基线升级
- Ubuntu 26.04 支持
- Python 3.14 bindings
- NVFP4 dual-GEMM fusion (SM121 / DGX Spark)
- Global Performance Tuner (自动化 trtexec 调参)


---

## TensorRT Developer Guide (Inference Library) 速查笔记

> 来源：TensorRT Inference Library Documentation（2026-06 更新）

---

### DG-1 量化工作流速查

#### 支持的量化精度

| 类型 | 位宽 | 用途 | 备注 |
|------|------|------|------|
| FP32 | 32 | 基线精度 | 默认 |
| FP16 | 16 | 通用加速 | Tensor Core 友好 |
| BF16 | 16 | 训练兼容 | 指数范围同 FP32 |
| FP8 (E4M3) | 8 | 低精度计算 | 需 Q/DQ 层 |
| FP4 (E2M1) | 4 | 权重压缩 | 需 Q/DQ，2元素/byte |
| INT8 | 8 | 整数量化 | 对称，需 Q/DQ |
| INT4 | 4 | 权重压缩 | weight-only，需 DQ |
| E8M0 | 8 | MXFP8 scale | 纯指数，无尾数 |

#### PTQ 工作流

```
训练好的 FP32 模型
    ↓ 导出 ONNX
使用 Polygraphy/trtexec + 校准数据集
    ↓ 确定 per-tensor/per-channel scale
Builder 自动融合 Q/DQ 节点
    ↓
输出 INT8/FP8 引擎
```

#### QAT 工作流

```
FP32 模型 + Q/DQ 伪量化节点
    ↓ 继续训练（前向模拟量化误差）
导出 ONNX（含 Q/DQ 节点）
    ↓ TRT Builder 识别并融合
输出高精度 INT8 引擎
```

#### 关键 API

```python
# PTQ 校准 (Python)
config.int8_calibrator = trt.IInt8EntropyCalibrator2(...)
config.set_flag(trt.BuilderFlag.INT8)

# 强类型网络 (11.0+，weak typing 已移除)
config.set_flag(trt.BuilderFlag.STRONGLY_TYPED)
# 或直接设置层输出精度：
layer.precision = trt.DataType.HALF
layer.get_output(0).dtype = trt.DataType.HALF
```

---

### DG-2 动态形状配置速查

#### 核心概念

- 动态维度用 `-1` 标记
- 必须为每个动态输入设置 Optimization Profile（min/opt/max）
- Builder 为每组 Profile 选择最优 kernel

#### 配置流程

```python
# 1. 设置网络输入为动态
network.get_input(0).shape = (-1, 3, -1, -1)  # batch, C, H, W

# 2. 创建 Optimization Profile
profile = builder.create_optimization_profile()
profile.set_shape("input",
    min=(1, 3, 224, 224),
    opt=(8, 3, 512, 512),
    max=(32, 3, 1024, 1024)
)
config.add_optimization_profile(profile)

# 3. 运行时设置实际形状
context.set_input_shape("input", (4, 3, 640, 640))
```

#### Shape Tensor vs Execution Tensor

| 类别 | Shape Tensor | Execution Tensor |
|------|-------------|-----------------|
| 内容 | 维度值（整数） | 实际计算数据 |
| 维度动态性 | 值可变 | 形状可变 |
| Profile 设置 | `set_shape_input()` | `set_shape()` |

#### Named Dimensions

```python
# 约束多个输入共享同一动态维度
profile.set_shape("encoder_input", min=(1,128), opt=(1,512), max=(1,1024))
profile.set_shape("decoder_input", min=(1,128), opt=(1,512), max=(1,1024))
# 使用命名维度确保 seq_len 一致
```

---

### DG-3 Plugin V3 编写速查

#### 类继承关系

```
IPluginV3 (基类)
├── IPluginV3OneCore       → 通用元数据
├── IPluginV3OneBuild      → build 阶段逻辑
└── IPluginV3OneRuntime    → runtime 阶段逻辑

IPluginCreatorV3One        → 工厂类（实例化插件）
```

#### 最小实现模板 (C++)

```cpp
class MyPlugin : public IPluginV3,
                 public IPluginV3OneCore,
                 public IPluginV3OneBuild,
                 public IPluginV3OneRuntime {
public:
    // Core
    char const* getPluginName() override;
    char const* getPluginVersion() override;
    char const* getPluginNamespace() override;

    // Build
    int32_t getNbOutputs() override;
    int32_t configurePlugin(...) override;
    bool supportsFormatCombination(...) override;
    size_t getWorkspaceSize(...) override;

    // Runtime
    int32_t enqueue(...) override;  // 实际 CUDA kernel 调用
    IPluginV3* clone() override;
};
```

#### 注册与使用

```cpp
// 注册
REGISTER_TENSORRT_PLUGIN(MyPluginCreator);

// ONNX 中使用: op_type 匹配 plugin name
// 或代码中手动添加:
network->addPluginV3(inputs, nbInputs, shapeInputs, nbShapeInputs, *plugin);
```

#### Python Plugin (10.6+)

```python
@trt.plugin("MyOp", "1")
def my_op(inp: trt.ITensor) -> trt.ITensor:
    # 定义形状推断和 kernel
    ...
```

---

### DG-4 Transformer 引擎配置速查

#### Fused Attention 使用

```python
# IAttention layer 自动选择 fused kernel
attn_layer = network.add_attention(q, k, v)
attn_layer.scale = 1.0 / math.sqrt(head_dim)

# KV Cache 输入
attn_layer.set_input(3, past_key)    # past K
attn_layer.set_input(4, past_value)  # past V
```

#### 数据格式选择

| 格式 | 适用场景 | 说明 |
|------|---------|------|
| Padded | 定长批次 | 所有序列对齐至 max_seq_len |
| Packed/Ragged | 变长批次 | 紧凑拼接 + offset 数组，减少冗余计算 |

#### Mixture of Experts (MoE)

- 路由器选择 top-k 专家（通常 k=2）
- TRT 融合 gating + expert dispatch + combine
- 支持 TP (张量并行) 切分专家

#### Multi-Device Attention (Blackwell SM100+)

```
KV 序列按 chunk 分配到各 GPU
   → 每 GPU 计算局部 attention
   → AllReduce 合并结果
```

---

### DG-5 Engine 构建与部署流程

#### 完整流程图

```
ONNX 模型
    ↓ trt.OnnxParser
INetworkDefinition
    ↓ Builder + BuilderConfig
Serialized Engine (Plan)
    ↓ 保存到磁盘 (.engine/.plan)
    ↓ Runtime.deserialize_cuda_engine()
ICudaEngine
    ↓ create_execution_context()
IExecutionContext
    ↓ set_input_shape() + set_tensor_address()
    ↓ execute_async_v3(stream)
推理结果
```

#### BuilderConfig 关键设置

```python
config = builder.create_builder_config()

# 内存池限制
config.set_memory_pool_limit(trt.MemoryPoolType.WORKSPACE, 1 << 30)  # 1GB

# 精度标志
config.set_flag(trt.BuilderFlag.FP16)
config.set_flag(trt.BuilderFlag.INT8)

# 优化等级 (0=最快构建, 5=最优性能)
config.builder_optimization_level = 3

# 计时缓存
cache = config.create_timing_cache(b"")
config.set_timing_cache(cache, ignore_mismatch=False)

# 权重流式 (大模型)
config.set_flag(trt.BuilderFlag.WEIGHT_STREAMING)
```

#### 引擎兼容性矩阵

| 类型 | 说明 | 启用方式 |
|------|------|---------|
| 版本兼容 | 跨 TRT minor 版本 | `EXCLUDE_LEAN_RUNTIME` flag |
| 硬件兼容 | 同代不同 GPU | `HW_COMPATIBILITY_LEVEL` |
| 跨平台 | x86 ↔ ARM | `CROSS_PLATFORM_COMPATIBILITY` flag |

---

### DG-6 trtexec 常用命令速查

#### 基本用法

```bash
# ONNX → Engine
trtexec --onnx=model.onnx --saveEngine=model.engine

# 指定精度
trtexec --onnx=model.onnx --fp16 --int8 --saveEngine=model.engine

# 动态形状
trtexec --onnx=model.onnx \
    --minShapes=input:1x3x224x224 \
    --optShapes=input:8x3x512x512 \
    --maxShapes=input:32x3x1024x1024

# 性能基准
trtexec --loadEngine=model.engine --iterations=1000 --avgRuns=100

# 计时缓存
trtexec --onnx=model.onnx --timingCacheFile=cache.bin --saveEngine=model.engine

# 权重流式
trtexec --onnx=model.onnx --weightStreaming --saveEngine=model.engine
```

#### 关键输出指标

| 指标 | 含义 |
|------|------|
| Throughput | 吞吐量 (infer/sec) |
| GPU Compute Time (mean) | 平均 GPU 计算耗时 |
| H2D/D2H Latency | 主机↔设备数据传输延迟 |
| Total Host Wall-time | 端到端总延迟 |
| Peak GPU Memory | GPU 峰值显存占用 |

---

### DG-7 Polygraphy 调试速查

#### 多后端对比

```bash
# TRT vs ONNX-Runtime 精度对比
polygraphy run model.onnx \
    --trt --onnxrt \
    --atol 1e-3 --rtol 1e-3

# 保存各层输出
polygraphy run model.onnx --trt \
    --trt-outputs mark all \
    --save-outputs trt_outputs.json
```

#### ONNX 模型编辑

```bash
# 提取子图
polygraphy surgeon extract model.onnx \
    --inputs input0:[1,3,224,224]:float32 \
    --outputs conv1_out:auto \
    -o subgraph.onnx

# 常量折叠
polygraphy surgeon sanitize model.onnx --fold-constants -o folded.onnx
```

#### 精度诊断流程

```
1. polygraphy run → 发现 TRT 输出偏差大
2. --trt-outputs mark all → 逐层比对
3. 定位问题层 → 尝试锁定该层为 FP32
4. polygraphy surgeon extract → 隔离子图单独测试
5. 确认修复后重新构建完整引擎
```

---

### DG-8 Weight Streaming / Refitting 速查

#### Weight Streaming（超大模型）

```python
# Build 时启用
config.set_flag(trt.BuilderFlag.WEIGHT_STREAMING)

# Runtime 设置预算（GPU 上缓存多少权重）
engine.weight_streaming_budget_v2 = 2 * (1 << 30)  # 2GB on GPU
# 0 = 全部流式（最慢），-1 = 全部驻留（最快，需足够显存）
```

- 仅支持强类型网络
- 权重按需从 Host→Device，推理延迟会增加
- 适用场景：模型 > GPU 显存但可接受延迟折损

#### Refitting（热更新权重）

```python
# Build 时启用 refit
config.set_flag(trt.BuilderFlag.REFIT)

# Runtime refit
refitter = trt.Refitter(engine, logger)
refitter.set_named_weights("conv1.weight", new_weights)
assert refitter.refit_cuda_engine()
```

#### Weight Stripping（轻量部署）

```python
# Build 时剥离权重
config.set_flag(trt.BuilderFlag.STRIP_PLAN)

# 部署时 refit 注入权重
refitter = trt.Refitter(engine, logger)
refitter.set_named_weights(...)  # 逐个注入所有权重
refitter.refit_cuda_engine()
```

- 用途：Plan 文件极小，权重独立存储/更新
- 必须与 REFIT flag 配合使用

---

### DG-9 多设备推理速查

#### 分布式集合操作

| 操作 | 含义 |
|------|------|
| AllReduce | 所有 GPU 上的张量求和/均值后广播 |
| AllGather | 收集各 GPU 片段拼接为完整张量 |
| ReduceScatter | 先 Reduce 再按 rank 切分 |
| Broadcast | 一个 GPU → 所有 GPU |
| AllToAll | 全交换（每对 rank 互发不同片段） |
| Gather / Scatter | 单点收集 / 分发 |

#### API 用法

```python
# 创建 NCCL Communicator
comm = trt.IDistCommunicator(nccl_unique_id, rank, num_ranks)

# 添加 AllReduce 层
layer = network.add_dist_collective(input_tensor, trt.DistCollectiveOp.ALL_REDUCE)
layer.set_communicator(comm)
```

#### 支持条件

- DistCollective: Ampere (SM 80)+ GPU
- Multi-Device Attention: Blackwell (SM 100)+
- 需要 NCCL 库
- 仅特定 build 配置支持

---

### DG-10 数据格式与 I/O 速查

#### 数据格式

| 格式 | 布局 | 说明 |
|------|------|------|
| CHW (线性) | Channel-Height-Width | TRT 默认解释 |
| HWC | Height-Width-Channel | 某些输入格式 |
| CHW vectorized | CHW + vector stride | Tensor Core 内部格式 |

#### I/O 注意事项

- 输入/输出张量必须命名（`network.get_input(0).name = "input"`）
- UINT8 仅可作为网络 I/O，内部需 CastLayer 转换
- 空张量（zero-volume）：维度含 0，跳过执行但保持管线正确
- 可复用输入 buffer 作为输出 buffer（需 Builder 不重排地址）
- Sparsity（稀疏）：结构化稀疏 2:4 可被 TRT 自动利用

---

### DG-11 生态集成速查

| 组件 | 用途 |
|------|------|
| Triton Inference Server | 生产级推理服务（HTTP/gRPC + 动态 batching） |
| DALI | GPU 数据预处理流水线 |
| Torch-TensorRT | PyTorch → TRT 编译（torch.compile 后端） |
| MLIR-TensorRT | TF/JAX → TRT 编译 |
| TriPy | Python 逐层构建 TRT 网络的 DSL |
| NVIDIA DRIVE | 车载部署平台（含 DLA 支持） |
| MIG | Multi-Instance GPU 隔离 |
| Green Contexts | CUDA 绿色上下文，多租户 GPU 共享 |

---

## API Reference 精读笔记

> 来源：TensorRT 11.1.0 Standard Python API Documentation

### API 模块结构

```
tensorrt (顶层模块)
├── Foundational Types: DataType, Weights, Dims, IHostMemory
├── Core: Logger/Profiler/OptimizationProfile/BuilderConfig/
│         Builder/ProgressMonitor/DebugListener/Engine/
│         ExecutionContext/Runtime/RuntimeConfig/Refitter/
│         ErrorRecorder/TimingCache/GpuAllocator/
│         EngineInspector/SerializationConfig
├── Network (Graph): NetworkDefinition, ITensor, ILayer,
│                    ~40种具体Layer接口
├── Plugin: IPluginCreator(V2/V3), PluginRegistry,
│           IPluginV2DynamicExt, IPluginV3系列
├── tensorrt.plugin: register/impl/aot_impl/autotune (Python原生插件DSL)
└── parsers.Onnx: OnnxParser, OnnxParserRefitter, ParserError
```

### 与 Overview/Inference Library 笔记的映射关系

API Reference 是前两部分概念在代码层面的落地：

| 概念（Overview/DG笔记） | 对应 API |
|------|------|
| Builder 两阶段工作流 | `Builder` → `IBuilderConfig` → `build_serialized_network()` |
| OptimizationProfile (min/opt/max) | `IOptimizationProfile.set_shape()` |
| workspace 限制 | `IBuilderConfig.set_memory_pool_limit()` |
| Q/DQ 层 | `INetworkDefinition.add_quantize()`/`add_dequantize()`/`add_dynamic_quantize_v2()` |
| refitting | `Refitter` 类 + `INetworkDefinition.mark_weights_refittable()` |
| weight streaming | `ICudaEngine.get_weight_streaming_automatic_budget()` |
| Plugin V3 | `IPluginV3OneCore`/`OneBuild`/`OneRuntime` 三段式接口 |
| fused attention / KV cache | `INetworkDefinition.add_attention_v2()` / `add_kv_cache_update()` |
| MoE 融合 | `INetworkDefinition.add_moe()` + `IMoELayer.set_swiglu_params()` |
| DistCollective | `INetworkDefinition.add_dist_collective()` + `IDistCollectiveLayer` |
| data-dependent output | `INMSLayer`/`INonZeroLayer` + `IExprBuilder.declare_size_tensor()` |
| trust boundary（不可信 engine） | `Runtime.deserialize_cuda_engine()` 的安全前提 |

### Python 原生插件（tensorrt.plugin）新工作流

TensorRT 较新版本提供了无需写 C++ 的插件开发路径：

```python
import tensorrt.plugin as trtp

@trtp.register("my_ns::my_op")
def _(x: trtp.TensorDesc) -> trtp.TensorDesc:
    return x.like()  # 形状/类型推导规则

@trtp.impl("my_ns::my_op")
def _(x: trtp.Tensor, y: trtp.Tensor):
    # JIT 核函数实现（可用 Triton/CUDA Python 等）
    ...
```

相比传统 `IPluginV2DynamicExt`/`IPluginV3` C++ 实现，这条路径显著降低了自定义算子的开发门槛，适合快速原型和研究场景；生产级高性能插件仍推荐 `aot_impl`（预编译）或完整 C++ IPluginV3 实现。
