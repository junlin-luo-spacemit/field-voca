# AI 软件栈学习笔记

> 文档入口：
> - TensorRT: https://docs.nvidia.com/deeplearning/tensorrt/index.html
> - cuDNN: https://docs.nvidia.com/cudnn/index.html
> - NeMo: https://docs.nvidia.com/nemo/index.html
> - NIM: https://docs.nvidia.com/nim/index.html
> - Dynamo: https://docs.nvidia.com/dynamo/index.html

---

## NVIDIA AI 软件栈架构

```
应用层 (Applications)
    ↓
AI Enterprise / NIM Microservices
    ↓
NeMo (Training & Alignment) | TensorRT (Inference Optimization)
    ↓
cuDNN | cuBLAS | CUTLASS (Math Libraries)
    ↓
CUDA Runtime & Driver
    ↓
GPU Hardware (H100 / B200 / ...)
```

---

## 各产品定位速记

| 产品 | 阶段 | 核心价值 |
|------|------|----------|
| NeMo | 训练/微调 | LLM 全流程训练套件，含 RLHF、评估 |
| TensorRT | 推理优化 | 模型编译优化，针对 NVIDIA GPU 最大化性能 |
| cuDNN | 底层库 | DL 原语（卷积、BN等）的 GPU 实现 |
| NIM | 推理部署 | 开箱即用的推理微服务，标准 REST API |
| Dynamo | 推理调度 | 数据中心级大规模推理集群调度框架 |
| CUTLASS | 底层库 | GEMM/矩阵乘法的 CUDA 模板库 |

---

## TensorRT 工作流程

```
训练框架 (PyTorch/TF)
    ↓ 导出
ONNX / TorchScript / TF SavedModel
    ↓ trtexec 或 Python API
TensorRT Engine (.trt / .engine)
    ↓ 部署
CUDA Runtime 推理
```

---

## 精度类型对比

| 精度 | 位宽 | 适用场景 | H100 Tensor Core 支持 |
|------|------|----------|----------------------|
| FP32 | 32-bit | 训练（高精度） | ✅ |
| TF32 | 19-bit | 训练加速（A100+） | ✅ |
| BF16 | 16-bit | 训练（更稳定） | ✅ |
| FP16 | 16-bit | 推理 / 训练混精度 | ✅ |
| FP8 | 8-bit | 推理（Hopper+） | ✅ |
| INT8 | 8-bit | 推理量化 | ✅ |
| INT4 | 4-bit | 极限压缩推理 | ✅ (Turing+) |

---

## 待深入阅读

- [ ] TensorRT Developer Guide - Precision and Quantization
- [ ] NeMo LLM Training Guide
- [ ] Dynamo Architecture Overview
- [ ] NIM Deployment Guide（本地部署 NIM 容器）
