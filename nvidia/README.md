---
tags: [nvidia, readme, index]
domain: all
last_updated: 2026-06-29
---

# NVIDIA 文档学习与词汇整理

> 来源：[NVIDIA Documentation Hub](https://docs.nvidia.com/)
> 开始日期：2026-06-29
> 知识库首页：[[HOME]]

## 目录结构

```
nvidia/
├── HOME.md                    # 知识库首页（Obsidian 入口）
├── README.md                  # 本文件：总览与学习计划
├── PROGRESS.md                # 学习进度追踪
├── _templates/                # Obsidian 词条模板
│   └── term.md
├── atoms/                     # 核心词条（原子文件）
│   ├── SM.md
│   ├── warp.md
│   ├── Tensor-Core.md
│   ├── HBM.md
│   ├── NVLink.md
│   ├── TensorRT.md
│   ├── CUDA-Core.md
│   └── Unified-Memory.md
├── glossary/                  # 汇总词汇表（跨领域）
│   └── master_glossary.md     # 主词汇表（中英对照）
├── cuda/                      # CUDA 并行计算
│   ├── notes.md               # 学习笔记
│   └── vocabulary.md          # 词汇
├── ai-software/               # AI 软件栈（NeMo、NIM、TensorRT 等）
│   ├── notes.md
│   └── vocabulary.md
├── hardware/                  # 硬件平台（DGX、Jetson、vGPU）
│   ├── notes.md
│   └── vocabulary.md
├── networking/                # 网络互连（NVLink、InfiniBand、DOCA）
│   ├── notes.md
│   └── vocabulary.md
└── omniverse/                 # Omniverse 仿真平台
    ├── notes.md
    └── vocabulary.md
```

## NVIDIA 文档主要产品线

### 🔥 Most Popular（高频入口）

| 产品 | 说明 | 文档链接 |
|------|------|----------|
| NVIDIA CUDA | GPU 并行计算开发工具包 | https://docs.nvidia.com/cuda/doc/index.html |
| NVIDIA TensorRT | 高性能深度学习推理 SDK | https://docs.nvidia.com/deeplearning/tensorrt/index.html |
| NVIDIA cuDNN | GPU 加速深度神经网络库 | https://docs.nvidia.com/cudnn/index.html |
| NVIDIA NeMo | AI 智能体开发与部署套件 | https://docs.nvidia.com/nemo/index.html |
| NVIDIA NIM | AI 模型微服务部署平台 | https://docs.nvidia.com/nim/index.html |
| NVIDIA DGX Platform | 企业级 AI 训练平台 | https://docs.nvidia.com/dgx/index.html |
| NVIDIA Jetson | 边缘 AI 计算平台 | https://docs.nvidia.com/jetson/index.html |
| NVIDIA vGPU | GPU 虚拟化平台 | https://docs.nvidia.com/vgpu/index.html |
| NVIDIA Omniverse | 3D 仿真与协作平台 | https://docs.omniverse.nvidia.com/ |
| NVIDIA CUTLASS | CUDA C++ 高性能矩阵计算模板库 | https://docs.nvidia.com/cutlass/index.html |
| NVIDIA AIStore | 分布式对象存储（AI 工作负载） | https://docs.nvidia.com/aistore |
| NVIDIA Dynamo | 数据中心级推理服务框架 | https://docs.nvidia.com/dynamo/index.html |
| NVIDIA AI Enterprise | 端到端 AI 开发部署平台 | https://docs.nvidia.com/ai-enterprise/index.html |

## 学习策略

1. **优先级排序**：CUDA → TensorRT → cuDNN → NeMo/NIM（从底层到上层）
2. **每份文档先读**：Overview / Introduction / Glossary 三个部分
3. **词汇记录格式**：见各 `vocabulary.md` 文件头部说明
4. **周期**：每周至少精读 1 个产品文档的 Overview，整理 20 个词条
