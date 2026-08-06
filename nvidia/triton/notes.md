---
tags: [nvidia, triton, notes]
domain: triton
last_updated: 2026-07-21
source: https://docs.nvidia.com/deeplearning/triton-inference-server/user-guide/docs/
---

# Triton Inference Server 精读笔记

> Triton 2.x | 文档更新 2026

---

## 一句话总结

Triton = **多框架、多设备、生产级推理服务框架**。将 TensorRT/PyTorch/ONNX 等模型包装成统一的 HTTP/gRPC 服务，内置动态批处理、并发执行、流水线编排，是 NVIDIA AI 栈的标准部署层。

---

## 整体架构

```
客户端（Python/C++/HTTP）
    │
    ├─ HTTP/REST (port 8000)  → KServe v2 协议
    ├─ gRPC       (port 8001)  → 高性能二进制协议
    └─ C API (in-process)      → 边缘/嵌入式场景

    │
    ▼
┌─────────────────────────────────────────┐
│         Triton Inference Server          │
│                                          │
│  Model Repository                        │
│  (local / GCS / S3 / Azure)             │
│                                          │
│  Per-Model Scheduler                     │
│  ├─ Default Scheduler (pass-through)     │
│  ├─ Dynamic Batcher (stateless)          │
│  └─ Sequence Batcher (stateful)          │
│                                          │
│  Model Instances                         │
│  ├─ TensorRT backend  → GPU             │
│  ├─ ONNX Runtime      → GPU/CPU         │
│  ├─ PyTorch (TorchScript) → GPU/CPU     │
│  ├─ OpenVINO          → CPU/Intel       │
│  ├─ Python backend    → any             │
│  └─ DALI backend      → GPU (preproc)   │
│                                          │
│  Metrics (port 8002) → Prometheus       │
└─────────────────────────────────────────┘
```

---

## 模型仓库结构

```
/model-repository/
├── my_trt_model/
│   ├── config.pbtxt           ← 模型配置
│   ├── 1/
│   │   └── model.plan         ← TensorRT engine
│   └── 2/
│       └── model.plan
│
├── my_onnx_model/
│   ├── config.pbtxt
│   └── 1/
│       └── model.onnx
│
├── my_python_model/
│   ├── config.pbtxt
│   └── 1/
│       └── model.py
│
└── my_ensemble/
    ├── config.pbtxt           ← platform: "ensemble"
    └── 1/
        └── (空目录，ensemble 无模型文件)
```

**云存储支持**：
```bash
# GCS
tritonserver --model-repository=gs://my-bucket/models

# S3
tritonserver --model-repository=s3://my-bucket/models

# Azure
tritonserver --model-repository=as://account/container/models
```

---

## 最小化 config.pbtxt

```protobuf
# TensorRT 模型示例
platform: "tensorrt_plan"
max_batch_size: 8

input [
  {
    name: "input0"
    data_type: TYPE_FP32
    dims: [ 16 ]        # 不含 batch 维度
  }
]

output [
  {
    name: "output0"
    data_type: TYPE_FP32
    dims: [ 4 ]
  }
]
```

**注意**：
- 全形状 = `[-1] + dims`（当 `max_batch_size > 0`）
- `-1` 表示可变维度（variable-size dimension）
- 大多数 TensorRT/ONNX 模型支持**自动配置**，可省略 config.pbtxt

---

## 实例组配置

```protobuf
# 默认：每个 GPU 一个实例
instance_group [
  { count: 1, kind: KIND_GPU }
]

# 指定 GPU，多实例
instance_group [
  {
    count: 2
    kind: KIND_GPU
    gpus: [ 0, 1 ]    # 在 GPU 0 和 1 上各放 2 个实例
  }
]

# CPU 实例
instance_group [
  { count: 4, kind: KIND_CPU }
]

# 混合部署
instance_group [
  { count: 1, kind: KIND_GPU, gpus: [ 0 ] },
  { count: 2, kind: KIND_CPU }
]
```

**并发模型执行原则**：
```
同一模型，4 实例：
请求 1 ─► 实例 0 → GPU
请求 2 ─► 实例 1 → GPU   ← 并发执行
请求 3 ─► 实例 2 → GPU
请求 4 ─► 实例 3 → GPU
请求 5 ─► 等待实例空闲...
```

---

## 动态批处理（Dynamic Batching）

### 适用场景
**无状态模型**（CNN、Transformer encoder/decoder 等），请求之间相互独立。

### 配置

```protobuf
# 最简配置：启用动态批处理，使用所有默认值
dynamic_batching { }

# 完整配置
dynamic_batching {
  preferred_batch_size: [ 4, 8 ]          # 首选批次大小
  max_queue_delay_microseconds: 100        # 最大等待时间（μs）
  preserve_ordering: true                  # 保持响应顺序
  priority_levels: 3                       # 3 个优先级
  default_priority_level: 2               # 默认优先级
}
```

### 调优流程

```
1. 设置合理的 max_batch_size（越大吞吐越高，但延迟也越高）
2. 启用 dynamic_batching {}（默认设置）
3. 用 Performance Analyzer 测量延迟和吞吐量
4. 若延迟在预算内，增加 max_queue_delay_microseconds 提高吞吐
5. 若有特定批次大小显著更优（如 TensorRT 多 profile），设置 preferred_batch_size
```

### 延迟 vs 吞吐量权衡

```
max_queue_delay = 0    → 最低延迟，最低吞吐（不等待）
max_queue_delay = 100μs → 稍高延迟，更好的批聚合
max_queue_delay = 1ms  → 高延迟，最大批聚合，最高吞吐
```

---

## 序列批处理（Sequence Batching）

### 适用场景
**有状态模型**（RNN/LSTM/GRU，流式 NLP 等），同一序列的多个请求必须在同一个模型实例上运行。

### 客户端协议

```python
# 客户端必须为同一序列设置相同的 correlation_id
# 并标记序列开始/结束

# 第一个请求（序列开始）
infer_input.set_data_from_numpy(data)
triton_client.infer(
    "my_stateful_model",
    inputs,
    sequence_id=1234,      # correlation ID
    sequence_start=True,   # 标记序列开始
)

# 中间请求
triton_client.infer(
    "my_stateful_model",
    inputs,
    sequence_id=1234,      # 相同的 correlation ID
)

# 最后一个请求（序列结束）
triton_client.infer(
    "my_stateful_model",
    inputs,
    sequence_id=1234,
    sequence_end=True,     # 标记序列结束
)
```

### 控制信号（Control Inputs）

```protobuf
sequence_batching {
  control_input [
    {
      name: "START"
      control [{ kind: CONTROL_SEQUENCE_START, fp32_false_true: [0, 1] }]
    },
    {
      name: "END"
      control [{ kind: CONTROL_SEQUENCE_END, fp32_false_true: [0, 1] }]
    },
    {
      name: "READY"
      control [{ kind: CONTROL_SEQUENCE_READY, fp32_false_true: [0, 1] }]
    },
    {
      name: "CORRID"
      control [{ kind: CONTROL_SEQUENCE_CORRID, data_type: TYPE_UINT64 }]
    }
  ]
}
```

---

## Continuous/Inflight Batching（LLM）

### 问题

传统批处理：
```
请求 A: [token1, token2, ..., token50]  → 50 步
请求 B: [token1, ..., token10]          → 10 步

静态批处理：必须等请求 B 执行 50 步后，批次才释放（浪费）
```

### Inflight Batching 解决方案

```
Step 1: [A, B, C, D]  → B完成2步，继续
Step 2: [A, B', C, D] → B完成，空出 slot
Step 3: [A, E, C, D]  → 新请求 E 立即填充 slot（不等待）
...
```

**核心**：每个 token 生成步骤都是独立调度，空出的 slot 立即被新请求占用。

### Triton 实现（Iterative Sequences）

```protobuf
sequence_batching {
  iterative_sequence: true  # 启用迭代序列模式
}
```

与 TRT-LLM backend 结合使用，实现生产级 LLM serving。

---

## 集成模型（Ensemble Pipeline）

### 经典流水线

```
客户端请求（图片字节）
    │
    ▼
[image_preprocess_model]   ← Python/DALI backend
  input:  RAW_IMAGE
  output: PREPROCESSED_IMAGE
    │
    ├─────────────────────┐
    ▼                     ▼
[classification_model]  [segmentation_model]  ← TensorRT
  input: PREPROCESSED     input: PREPROCESSED
  output: CLASS_SCORES    output: SEG_MASK
    │                         │
    └─────────┬───────────────┘
              ▼
        合并结果返回给客户端
```

### 配置示例

```protobuf
name: "ensemble_pipeline"
platform: "ensemble"
max_batch_size: 4

input  [{ name: "RAW_IMAGE",     data_type: TYPE_STRING, dims: [1] }]
output [
  { name: "CLASSIFICATION", data_type: TYPE_FP32, dims: [1000] },
  { name: "SEGMENTATION",   data_type: TYPE_FP32, dims: [3, 224, 224] }
]

ensemble_scheduling {
  step [
    {
      model_name: "image_preprocess_model"
      model_version: -1          # 始终用最新版本
      input_map  { key: "RAW_IMAGE"          value: "IMAGE" }
      output_map { key: "PREPROCESSED_OUTPUT" value: "preprocessed" }
    },
    {
      model_name: "classification_model"
      model_version: -1
      input_map  { key: "FORMATTED_IMAGE"       value: "preprocessed" }
      output_map { key: "CLASSIFICATION_OUTPUT"  value: "CLASSIFICATION" }
    },
    {
      model_name: "segmentation_model"
      model_version: -1
      input_map  { key: "FORMATTED_IMAGE"     value: "preprocessed" }
      output_map { key: "SEGMENTATION_OUTPUT"  value: "SEGMENTATION" }
    }
  ]
}
```

**关键**：
- `key`：子模型的张量名
- `value`：集成模型内部的共享张量名
- classification 和 segmentation 共享同一个 `preprocessed` 张量 → 并行执行

---

## BLS（Business Logic Scripting）

更灵活的流水线，支持条件逻辑和循环：

```python
# model.py
import triton_python_backend_utils as pb_utils

class TritonPythonModel:
    def execute(self, requests):
        responses = []
        for request in requests:
            # 获取输入
            input_data = pb_utils.get_input_tensor_by_name(request, "INPUT")
            
            # 条件调用不同模型
            if input_data.as_numpy()[0] > 0.5:
                infer_request = pb_utils.InferenceRequest(
                    model_name="model_A",
                    inputs=[input_data]
                )
            else:
                infer_request = pb_utils.InferenceRequest(
                    model_name="model_B",
                    inputs=[input_data]
                )
            
            # 执行内部推理
            infer_response = infer_request.exec()
            output = pb_utils.get_output_tensor_by_name(infer_response, "OUTPUT")
            
            responses.append(
                pb_utils.InferenceResponse(output_tensors=[output])
            )
        return responses
```

**vs Ensemble**：
| | Ensemble | BLS |
|--|--|--|
| 条件逻辑 | ❌ 固定 DAG | ✅ |
| 循环 | ❌ | ✅ |
| 性能 | 更高（无 Python 解释器） | 略低 |
| 调试 | 较难 | 容易 |

---

## 版本策略

```protobuf
# 仅最新版本（默认）
version_policy: { latest: { num_versions: 1 } }

# 最新 2 个版本（灰度发布）
version_policy: { latest: { num_versions: 2 } }

# 全部版本
version_policy: { all: {} }

# 指定版本（生产固定版本）
version_policy: { specific: { versions: [2, 4] } }
```

---

## 常用 REST API

```bash
# 检查服务器健康
GET /v2/health/ready

# 列出所有模型
GET /v2/models

# 查看模型状态
GET /v2/models/<model_name>

# 查看模型配置（含自动生成部分）
GET /v2/models/<model_name>/config

# 执行推理
POST /v2/models/<model_name>/infer

# 加载模型（EXPLICIT 模式）
POST /v2/repository/models/<model_name>/load

# 卸载模型
POST /v2/repository/models/<model_name>/unload
```

---

## 最佳实践

### ✅ 推荐做法

```
1. 优先使用 TensorRT backend（最佳 GPU 性能）
   模型 → ONNX → TensorRT → Triton

2. 为同一类 GPU 提前构建 TensorRT engine
   避免 Triton 启动时临时构建（耗时）

3. 启用动态批处理（无状态模型）
   dynamic_batching { }

4. 多实例并发（高并发场景）
   instance_group [{ count: 4, kind: KIND_GPU }]

5. 使用 Ensemble 减少数据往返
   避免客户端在预处理/推理/后处理之间多次 HTTP 调用

6. 模型预热避免冷启动延迟
   model_warmup { ... }

7. 使用响应缓存（重复请求场景）
   response_cache { enable: true }
```

### ❌ 常见陷阱

```
1. 忘记指定 cc_model_filenames（TensorRT 多 GPU）
   TensorRT engine 绑定 Compute Capability，不同架构需要不同 engine

2. max_batch_size 与实际模型不匹配
   - 模型支持批处理 但设置为 0 → 无法动态批处理
   - 设置过大 → 超出模型容量

3. 对 LLM 使用普通动态批处理
   → 应使用 inflight batching（iterative_sequence）
   
4. Ensemble 步骤 key/value 混淆
   key = 子模型的输入/输出名
   value = 集成模型内部的中间张量名
   
5. 有状态模型未设置 correlation_id
   → sequence batcher 无法识别序列，状态混乱
```

---

## 部署检查清单

```
模型准备：
□ 将模型转换为目标 backend 格式（TensorRT plan、ONNX 等）
□ 验证模型可在目标 GPU 上运行
□ 准备 config.pbtxt 或验证自动配置可用

性能配置：
□ 设置合理的 max_batch_size
□ 配置 dynamic_batching（无状态模型）
□ 设置 instance_group（并发请求数 = instances × max_batch_size）
□ 配置 model_warmup（避免冷启动）

高可用性：
□ 设置 version_policy（生产固定版本）
□ 配置健康检查集成到 Kubernetes liveness/readiness probe
□ 设置 Prometheus 指标监控

LLM 特殊配置：
□ 使用 TRT-LLM backend
□ 启用 iterative_sequence（inflight batching）
□ 配置 decoupled mode（流式生成）
```

---

## 与 NVIDIA AI 栈的关系

```
用户应用
    │
    ▼
NVIDIA NIM ────────────┐
(封装好的推理微服务)     │
    │                  ▼
    │           Triton Inference Server ← 标准部署层
    │                  │
    ├───────────────────┤
    │           ┌──────┴────────┐
    │       TRT-LLM backend  TensorRT backend
    │       (LLM 推理)        (CNN/Transformer)
    │           │                  │
    └───────────┴──────────────────┘
                │
              cuDNN  ←  CUDA  ←  NCCL（多卡）
                │
              GPU Hardware
```

**位置**：Triton 是 cuDNN/TensorRT 的上层服务化封装，NIM 是 Triton 的上层产品化封装。
