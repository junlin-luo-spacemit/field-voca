---
tags: [nvidia, nemo, notes]
domain: nemo
last_updated: 2026-07-21
source: https://docs.nvidia.com/nemo/ | https://docs.nvidia.com/nemo/microservices/latest/
---

# NVIDIA NeMo 精读笔记

> NeMo 2026 | 文档更新 Apr 2026

---

## 一句话总结

NeMo = **AI 模型全生命周期管理套件**。Framework 负责训练，Platform（Microservices）负责微调/评估/安全/数据的云原生运维，NIM 负责推理服务。三层配合覆盖从预训练到生产部署的完整流程。

---

## NeMo 产品组成

```
NVIDIA NeMo 套件
├── NeMo Framework         ← 开源训练框架（本文重点）
│   ├── LLM/VLM 预训练
│   ├── SFT / LoRA / RLHF / DPO
│   ├── 基于 Megatron-LM 的 3D 并行
│   └── Speech AI（ASR/TTS）
│
├── NeMo Platform          ← 云原生微服务平台
│   ├── NeMo Customizer    ← 微调服务（LoRA/Full SFT）
│   ├── NeMo Evaluator     ← 评估服务（100+ 基准）
│   ├── NeMo Guardrails    ← 实时安全防护
│   ├── NeMo Data Designer ← 合成数据生成
│   └── NeMo Auditor       ← 安全漏洞扫描
│
└── NeMo Agent Toolkit     ← 智能体开发工具
    └── 评估驱动的 Agent 优化
```

---

## NeMo Framework：LLM 训练全流程

### 1. 预训练（Pre-training）

```
海量语料（万亿 tokens）
    │
    ▼
数据处理（去重、质量过滤、tokenize）
    │
    ▼
分布式训练集群（数百~数千 GPU）
    │
    ├─ 数据并行（Data Parallel）
    ├─ 张量并行（Tensor Parallel, TP）
    ├─ 流水线并行（Pipeline Parallel, PP）
    └─ 上下文并行（Context Parallel, CP）
    │
    ▼
预训练 checkpoint（基础模型）
```

### 2. 监督微调（SFT）

```
预训练基础模型
    │
    ▼
指令微调数据集
(system prompt, user message, assistant response)
    │
    ├─ Full SFT（全参数，效果最好）
    └─ LoRA（低秩适配，显存低、速度快）
    │
    ▼
SFT 模型（指令遵循能力）
```

### 3. 对齐（Alignment）

```
SFT 模型
    │
    ├─ RLHF / PPO
    │   ├─ 训练奖励模型（人类偏好标注）
    │   └─ PPO 强化学习优化
    │
    ├─ DPO（直接偏好优化，更简单稳定）
    │   └─ 直接从偏好对比数据学习
    │
    └─ RLAIF（AI 反馈替代人类反馈）
    │
    ▼
对齐后模型（RLHF/DPO 模型）
    │
    ▼
NIM 部署 → Triton → GPU 推理
```

---

## NeMo Platform 微服务工作流

### 完整 AI Agent 开发流程

```
1. 数据准备
   [NeMo Data Designer]
   合成训练数据 → 领域特定 QA 对

2. 模型微调
   [NeMo Customizer]
   基础模型 + 合成数据 → LoRA 适配器 / SFT 模型

3. 模型评估
   [NeMo Evaluator]
   微调模型 vs 基础模型 → 性能对比报告

4. 安全测试
   [NeMo Auditor]
   红队测试 → 漏洞报告

5. 生产部署
   [NIM] → 通过 NeMo Guardrails 保护的推理端点

6. 持续优化（飞轮）
   生产流量 → 数据收集 → 再次微调 → 再次评估
```

### NeMo Customizer 工作流

```python
# 通过 NeMo Platform SDK 提交微调任务
from nemo_platform import NeMoPlatformClient

client = NeMoPlatformClient(base_url="http://localhost", api_key="...")

# 1. 创建/引用基础模型实体
model = client.models.get("meta/llama3-8b-instruct")

# 2. 上传训练数据集
dataset = client.files.upload("training_data.jsonl", workspace="my-team")

# 3. 提交 LoRA 微调任务
job = client.customizer.create_job(
    model=model,
    dataset=dataset,
    method="lora",           # 或 "sft" 全参数微调
    hyperparameters={
        "training_type": "sft",
        "num_epochs": 3,
        "learning_rate": 1e-4,
        "batch_size": 4,
        "lora_rank": 16,
        "lora_alpha": 32,
    }
)

# 4. 等待完成
job.wait()
print(f"Fine-tuned model: {job.output_model}")

# 5. 部署为 NIM
# output_model 可直接被 NIM 加载
```

### 数据集格式（Chat 模型）

```jsonl
{"messages": [
  {"role": "system", "content": "You are a helpful math tutor."},
  {"role": "user", "content": "What is 2+2?"},
  {"role": "assistant", "content": "The answer is 4."}
]}
{"messages": [
  {"role": "user", "content": "Explain gradient descent."},
  {"role": "assistant", "content": "Gradient descent is..."}
]}
```

---

## 分布式训练并行策略

```
模型大小 vs 并行策略选择

小模型（< 7B）：
├─ 单 GPU 或数据并行（DDP）
└─ 用 NeMo Customizer 即可

中等模型（7B ~ 70B）：
├─ LoRA：2~4 GPU，数据并行
├─ Full SFT：8 GPU，数据+张量并行
└─ TP=4 或 TP=8 + DP

大模型（70B ~ 405B）：
├─ 需要 3D 并行
├─ TP=8, PP=4, DP=N
└─ 数十到数百 GPU

超大模型（> 405B，如 MoE）：
└─ 专家并行（Expert Parallel）+ 3D 并行
   数千 GPU，DGX 集群
```

### 关键并行维度

| 并行类型 | 切分维度 | 典型场景 | NCCL 操作 |
|---------|---------|---------|----------|
| 数据并行 (DP) | 批次数据 | 所有训练 | AllReduce 梯度 |
| 张量并行 (TP) | 权重矩阵列/行 | 大型 Attention/FFN | AllReduce/AllGather |
| 流水线并行 (PP) | 层 | 超深模型 | Send/Recv |
| 上下文并行 (CP) | 序列长度 | 长上下文训练 | AllGather/ReduceScatter |

---

## 评估指标体系

```
NeMo Evaluator 支持的评估类型

行业基准（Industry Benchmarks）
├─ 代码生成：HumanEval, MBPP
├─ 安全：TruthfulQA, MMLU
├─ 推理：ARC, HellaSwag, GSM8K
└─ 工具调用：BFCL, ToolBench

RAG 评估
├─ 检索：recall@k, NDCG@k
├─ 生成：faithfulness（忠实度）
└─ 答案：answer-relevancy, exactmatch

Agent 评估
├─ topic-adherence    ← 是否偏题
├─ tool-call-accuracy ← 工具调用正确率
└─ goal-accuracy      ← 最终目标达成率

相似度指标
├─ ROUGE（文本重叠）
├─ BLEU（n-gram 精确度）
└─ F1（词级别 F1）

LLM-as-a-Judge
└─ 自定义评分 rubric，任意维度
```

---

## 与 NVIDIA AI 栈的关系

```
NeMo 在 NVIDIA AI 栈中的位置

┌─────────────────────────────────────────────────────┐
│              应用层（AI Agents / Apps）              │
└─────────────────────────────────────────────────────┘
            │ 训练                │ 推理
            ▼                     ▼
┌─────────────────┐    ┌─────────────────────────────┐
│  NeMo Framework │    │  NVIDIA NIM                 │
│  (训练/微调/对齐) │    │  (推理微服务容器)            │
└─────────────────┘    └─────────────────────────────┘
            │                     │
            ▼                     ▼
┌─────────────────────────────────────────────────────┐
│           NeMo Platform（微服务运维层）               │
│  Customizer | Evaluator | Guardrails | Data Designer │
└─────────────────────────────────────────────────────┘
            │                     │
            ▼                     ▼
┌─────────────────┐    ┌─────────────────────────────┐
│ TensorRT / cuDNN│    │  Triton Inference Server     │
│ NCCL (多GPU通信) │    │  (已学习)                   │
└─────────────────┘    └─────────────────────────────┘
            │
            ▼
┌─────────────────────────────────────────────────────┐
│              CUDA + GPU 硬件                         │
└─────────────────────────────────────────────────────┘

关键关系：
- NeMo Framework 训练模型 → checkpoint
- NIM 加载 checkpoint → 部署为推理服务
- NeMo Customizer 微调 → 新 checkpoint → 部署为 NIM
- NeMo Evaluator 评估 NIM 的输出质量
```

---

## NeMo vs HuggingFace 对比

| 对比项 | HuggingFace | NeMo Framework |
|-------|-------------|----------------|
| 专注点 | 通用模型生态 | 大规模生产训练 |
| 分布式训练 | FSDP / DeepSpeed | Megatron-LM 3D 并行 |
| 极大规模支持 | 有限（< 70B 为主） | 原生 1000B+ 支持 |
| 模型格式 | HF 格式 | NeMo 格式（可转 HF） |
| 推理部署 | 各种方案 | NIM（最优化） |
| 企业级支持 | 有限 | NVIDIA AI Enterprise |
| 开源 | ✅ | ✅（Framework） |

---

## 快速上手指南

### 本地 NeMo Platform（Docker）

```bash
# 安装 SDK
pip install nemo-platform-python-sdk

# 登录 NGC
export NGC_CLI_API_KEY=<your-key>
echo "$NGC_CLI_API_KEY" | docker login nvcr.io -u '$oauthtoken' --password-stdin

# 启动 NeMo Platform
docker pull nvcr.io/0857255566152269/external/nmp-api:26.03.1
nmp quickstart configure --auto
nmp quickstart up --image nvcr.io/0857255566152269/external/nmp-api:26.03.1

# 基本操作
nmp models list              # 查看可用模型
nmp workspaces list          # 查看工作空间
nmp chat --model llama3-8b   # 快速测试
```

### NeMo Framework（直接训练）

```python
# 使用 NeMo 2.0 API（Megatron-Bridge）
import nemo
from nemo.collections.llm import GPTModel, PreTrainingDataModule

# 配置模型和训练
model = GPTModel(config=GPTConfig7B())
data = PreTrainingDataModule(paths=["data/train.bin"])

trainer = nemo.lightning.Trainer(
    devices=8,
    strategy=nemo.lightning.MegatronStrategy(
        tensor_model_parallel_size=2,
        pipeline_model_parallel_size=1,
    )
)

trainer.fit(model, data)
```
