---
tags: [nvidia, ai-software, vocabulary]
domain: ai-software
last_updated: 2026-06-29
source: https://docs.nvidia.com/
---

# AI 软件栈词汇表

> 涵盖：TensorRT、cuDNN、NeMo、NIM、AI Enterprise、Dynamo
> 来源：https://docs.nvidia.com/

---

## 推理优化（Inference Optimization）

### TensorRT / TensorRT
NVIDIA 高性能深度学习推理 SDK，将训练好的模型优化编译为高效的推理引擎。
> *TensorRT applies layer fusion, precision calibration, and kernel auto-tuning.*

### engine / 引擎
TensorRT 编译后的优化推理计划，针对特定 GPU 和 batch size 生成，不可跨 GPU 型号迁移。

### layer fusion / 算子融合
将多个计算层合并为一个，减少内存读写和 kernel 启动开销。
> e.g., Conv + BN + ReLU → 单个 fused kernel

### precision calibration / 精度校准
INT8 量化时，通过校准数据集统计激活值范围，在精度损失最小的前提下实现量化。

### quantization / 量化
将模型权重和激活从 FP32/FP16 降低到 INT8/FP8，减少显存占用并提升推理速度。
> *Post-training quantization (PTQ) vs Quantization-aware training (QAT)*
> 训练后量化 vs 量化感知训练

### dynamic shape / 动态形状
TensorRT 支持在运行时改变输入张量的维度（batch size、序列长度等）。

### plugin / 插件
TensorRT 中用于支持自定义算子的扩展机制。

---

## 模型格式与框架（Model Formats & Frameworks）

### ONNX (Open Neural Network Exchange) / 开放神经网络交换格式
框架无关的模型表示格式，用于 PyTorch/TensorFlow → TensorRT 的转换通道。

### checkpoint / 检查点
训练过程中保存的模型权重快照，用于恢复训练或导出推理模型。

### fine-tuning / 微调
在预训练基础模型上，使用特定任务数据继续训练以适配目标任务。

### LoRA (Low-Rank Adaptation) / 低秩适配
高效微调方法，只训练少量低秩参数矩阵，大幅降低显存和计算需求。

---

## 大模型服务（LLM Serving）

### NIM (NVIDIA Inference Microservice) / NVIDIA 推理微服务
封装了优化推理引擎的容器化微服务，提供标准化 API（兼容 OpenAI API 格式）。

### container / 容器
封装应用及其依赖的隔离运行环境（基于 Docker/OCI），NIM 以容器形式交付。

### throughput / 吞吐量
推理服务单位时间内处理的 token 数量（tokens/s）或请求数（req/s）。

### TTFT (Time To First Token) / 首 token 延迟
从请求发出到生成第一个 token 的时间，衡量用户感知响应速度。

### TBT (Time Between Tokens) / token 间延迟
相邻 token 生成之间的时间间隔，影响流式输出体验。

### KV cache / KV 缓存
保存注意力机制中 Key-Value 计算结果的缓存，避免重复计算历史 token，是 LLM 推理的核心内存消耗。

### continuous batching / 连续批处理
推理服务中动态将多个请求合并处理，相比静态批处理大幅提升 GPU 利用率。

### speculative decoding / 推测解码
使用小模型草稿 + 大模型验证的方式加速自回归生成，可提升 2-3× 速度。

### tensor parallelism / 张量并行
将单层的权重矩阵切分到多 GPU，适用于超大模型的推理部署。

### pipeline parallelism / 流水线并行
将模型的不同层分布到不同 GPU，形成流水线执行。

---

## NVIDIA NeMo 相关

### NeMo / NeMo
NVIDIA 模块化 AI 开发套件，覆盖 LLM 训练、对齐、评估和部署全流程。

### RLHF (Reinforcement Learning from Human Feedback) / 人类反馈强化学习
通过人类偏好标注训练奖励模型，再用 RL 优化 LLM 输出，ChatGPT 的核心技术之一。

### alignment / 对齐
使 LLM 的输出符合人类价值观、指令遵循和安全要求的训练过程。

### Megatron-LM / Megatron-LM
NVIDIA 开发的大规模 Transformer 模型训练框架，支持 3D 并行（数据+张量+流水线）。

---

## cuDNN 相关

### cuDNN (CUDA Deep Neural Network Library) / CUDA 深度神经网络库
提供卷积、池化、归一化、激活函数等深度学习基元的高度优化 GPU 实现。

### convolution / 卷积
CNN 的核心操作，cuDNN 提供多种高效算法（Winograd、FFT、implicit GEMM）。

### batch normalization / 批归一化（BN）
对每个 mini-batch 的激活值做归一化，加速训练并提高稳定性。

### fused operation / 融合操作
将多个独立操作（如 Conv+BN+ReLU）合并为单个 GPU kernel，减少内存带宽消耗。
