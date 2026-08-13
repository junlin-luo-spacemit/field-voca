---
tags: [nvidia, ai-software, inference, atom]
domain: ai-software
aliases: [TensorRT, TRT, 张量推理]
status: approved
created: 2026-06-29
updated: 2026-06-29
source: https://docs.nvidia.com/deeplearning/tensorrt/
superseded_by:
---

# TensorRT / TensorRT

## 定义

**TensorRT** 是 NVIDIA 的高性能深度学习**推理优化引擎**，接收训练好的模型并通过一系列优化生成在 NVIDIA GPU 上运行效率最大化的推理引擎。

## 官方原文

> "NVIDIA TensorRT is an SDK for high-performance deep learning inference, including a deep learning inference optimizer and runtime."

## 核心优化技术

| 技术 | 说明 |
|------|------|
| **Layer fusion** | 合并多个算子为一个 kernel，减少显存读写 |
| **Precision calibration** | FP32 → FP16 / INT8 / FP8，降低精度换速度 |
| **Kernel auto-tuning** | 对每个算子选择最优 CUDA kernel |
| **Dynamic shapes** | 支持可变 batch size / 序列长度 |
| **[[Tensor-Core]] 利用** | 自动调度矩阵运算到 Tensor Core |

## 工作流程

```
训练模型（PyTorch/TF/ONNX）
    ↓
trtexec / TensorRT Python API 解析
    ↓
图优化（fusion、量化、kernel tuning）
    ↓
生成 .engine 文件
    ↓
TensorRT Runtime 推理
```

## 关键术语

- **engine**：TensorRT 编译后的推理计划文件（与 GPU 型号绑定）
- **INetworkDefinition**：模型网络定义接口
- **IBuilder**：构建 engine 的核心类
- **trtexec**：命令行基准测试工具

## 相关词条

- [[Tensor-Core]] — TensorRT 自动利用的核心算力单元
- [[HBM]] — 推理过程中的显存
- NIM — 基于 TensorRT 的云原生推理微服务
- ONNX — TensorRT 的常用输入格式
