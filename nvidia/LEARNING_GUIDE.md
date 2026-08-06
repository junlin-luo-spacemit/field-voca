# NVIDIA 技术文档学习指南
## 专为技术小白设计的入门路径

> 本指南基于本仓库已整理的 ~1150 个词条，帮助你从零开始，按正确顺序理解 NVIDIA 的整套技术体系。

---

## 🧠 先建立这个基础认知：GPU 和 CPU 有啥不一样？

**一句话版本**：CPU 是"全能选手"，很聪明但人数少；GPU 是"流水线工人"，每个人很简单但人数极多。

| | CPU | GPU |
|---|---|---|
| 核心数 | 几个到几十个 | 几千到几万个 |
| 擅长 | 复杂逻辑、分支判断 | 大量重复的数学计算 |
| 比喻 | 一位天才博士 | 一万个小学生同时算加法 |

**为什么 AI 需要 GPU？**
训练一个 AI 模型需要做海量的矩阵乘法（比如"1000 × 1000 的矩阵乘以另一个 1000 × 1000 的矩阵"），这种活 GPU 做起来比 CPU 快 100 倍以上。

---

## 📚 整个知识体系地图

把 NVIDIA 的技术想象成一栋楼：

```
┌─────────────────────────────────────────────┐
│  第 5 层：应用层（你实际使用的产品）         │
│  NIM（一键部署 AI）/ Isaac Sim（机器人仿真）  │
│  Omniverse（3D 协作平台）                    │
├─────────────────────────────────────────────┤
│  第 4 层：AI 生产工具                        │
│  NeMo（训练框架）/ Triton（推理服务器）       │
│  Dynamo（LLM 推理编排）                      │
├─────────────────────────────────────────────┤
│  第 3 层：优化推理（让模型跑得快）            │
│  TensorRT（推理加速）/ cuDNN（神经网络算子）  │
├─────────────────────────────────────────────┤
│  第 2 层：多 GPU 协作                        │
│  NCCL（多 GPU 通信）                         │
├─────────────────────────────────────────────┤
│  第 1 层：GPU 编程基础                       │
│  CUDA（让程序在 GPU 上运行）                 │
├─────────────────────────────────────────────┤
│  地基：硬件                                  │
│  DGX（AI 服务器）/ Jetson（边缘设备）        │
│  vGPU（虚拟化 GPU）                          │
└─────────────────────────────────────────────┘
```

**学习建议：从地基往上学**。每一层都建立在下一层之上，但你不一定要学完每一层才能继续——根据你的目标选择合适的入口。

---

## 🗺️ 三种学习路径（根据你的目标选一条）

### 路径 A：我想做 AI 推理部署（最常见的工作场景）
```
硬件了解（30 分钟） → CUDA 基础（1 周） → TensorRT（1~2 周） → Triton（1 周）
```
**终点**：能把一个 AI 模型从 PyTorch 导出、压缩优化、部署成服务。

### 路径 B：我想做大模型训练和微调
```
硬件了解 → CUDA 基础 → cuDNN → NCCL → NeMo
```
**终点**：能用多 GPU 训练、微调大语言模型。

### 路径 C：我想做机器人/3D 仿真
```
硬件了解 → Omniverse Kit SDK → Isaac Sim
```
**终点**：在虚拟环境中仿真机器人、生成训练数据。

---

## 📖 各领域学习指南

### 🔴 第一站：硬件（必看，30 分钟，读词汇表即可）

**参考文件**：`hardware/vocabulary.md`、`hardware/notes.md`

**你需要知道的 3 件事**：

1. **GPU 的型号怎么看**：H100 > A100 > V100（数字越大越新越强）。"SXM" 版本比 "PCIe" 版本带宽更高（价格也更贵）。

2. **DGX 是什么**：就是 NVIDIA 出的"AI 专用服务器"，8 块顶级 GPU 插在一起，内部用 NVLink 高速连接，买来即用不需要自己拼装。

3. **Jetson 是什么**：边缘 AI 计算模块，体积像手掌，功耗只有几十瓦，用在机器人、无人机、摄像头等设备上。

**初学者只需要记住的关键词**：
- `GPU 架构世代`：Volta → Turing → Ampere → Hopper → Blackwell（越新越强）
- `HBM`：显卡上的高速内存，带宽极高
- `NVLink`：多个 GPU 之间的高速数据通道（比 PCIe 快 7 倍以上）
- `MIG`：把一块大 GPU 切分成多个小 GPU，让多个用户共用

---

### 🟠 第二站：CUDA 基础（核心中的核心）

**参考文件**：`cuda/notes.md`、`cuda/vocabulary.md`

**先理解这个模型（用工厂类比）**：

```
你写的 CUDA 程序
    │
    ▼
GPU 上的 Grid（整个工厂）
    │
    ├── Block（一个车间）——可以共享高速暂存内存（Shared Memory）
    │       │
    │       └── Thread（一个工人）——每人有自己的寄存器
    │
    └── Block（另一个车间）
```

**最重要的 4 个概念**：

| 概念 | 白话解释 |
|------|----------|
| `kernel` | 你写的函数，但它在 GPU 上被成千上万个线程同时执行 |
| `thread/block/grid` | 线程 → 线程块 → 网格，三层组织结构 |
| `shared memory` | 同一个 Block 内的线程可以共享的超快内存（比显存快 10 倍） |
| `warp` | 32 个线程的最小执行单位，必须执行相同指令（否则"warp divergence"，性能下降） |

**内存类型速查（从快到慢）**：
```
寄存器（最快，每个线程私有）
    ↓
共享内存 Shared Memory（快，Block 内共享）
    ↓
L2 Cache（自动管理）
    ↓
显存 Global Memory（慢，但容量大，所有线程可访问）
    ↓
主机内存 Host Memory（最慢，在 CPU 那边）
```

**初学者常见误区**：
- ❌ 以为 GPU 比 CPU 快是因为它"更聪明" → ✅ 实际是因为它能"同时做很多事"
- ❌ 以为程序直接搬到 GPU 就变快 → ✅ 需要明确告诉 GPU 哪些数据/计算才能加速
- ❌ 随意访问内存 → ✅ 内存访问要"对齐"（coalescing），否则性能暴跌

**推荐学习顺序**：
1. `cuda/notes.md` 的 Part 1：编程模型（理解 kernel、thread、block）
2. `cuda/notes.md` 的 Part 2：内存类型（理解什么内存放什么）
3. 实验：把一个向量加法从 CPU 改写成 CUDA 版本

---

### 🟡 第三站：cuDNN（神经网络的加速库）

**参考文件**：`cudnn/notes.md`、`cudnn/vocabulary.md`

**一句话理解**：cuDNN 就是一个"高度优化的数学函数库"，里面预先实现了深度学习里最常用的操作（卷积、矩阵乘法、注意力机制等），你不需要自己用 CUDA 手写，直接调用即可。

**类比**：你不需要自己手写排序算法，直接调用 `sort()` 函数；同理，你不需要手写卷积，直接调用 cuDNN 的卷积函数。

**最关键的设计**：Graph API（操作图）

```
传统方式：一次调用 = 一个操作 = 一次显存读写
    卷积 → 写入显存 → 激活函数 → 写入显存 → 归一化 → 写入显存...

cuDNN Graph API：把多个操作组成图 = 一次执行 = 只读写一次显存
    卷积 + 激活函数 + 归一化 → 合并成一个 kernel → 一次显存读写
                                    （这叫"操作融合"）
```

**你需要理解的核心流程**（11 步，但其实不用全背）：
1. 创建图
2. 添加输入张量（Tensor）
3. 添加操作节点
4. 验证图
5. 查询最优执行方案（启发式算法）
6. 执行

**Flash Attention 是什么**：
这是一种"注意力机制的高效实现"，让 Transformer 模型（GPT、BERT 等）的注意力计算消耗的显存从 O(N²) 降到 O(N)。cuDNN 内置了 Flash Attention 的实现，现代 LLM 几乎都在用。

---

### 🟢 第四站：TensorRT（让模型推理更快）

**参考文件**：`tensorrt/notes.md`、`tensorrt/vocabulary.md`

**先理解"推理"和"训练"的区别**：
- **训练**：喂给模型大量数据，反复调整参数，很慢，只做一次
- **推理**：拿训练好的模型，给它一个输入，得到一个输出，要求很快，反复做

TensorRT 解决的是**推理阶段**的速度问题。

**TensorRT 的两个阶段（记住这个！）**：

```
阶段一：构建（离线，慢，只做一次）
    ONNX 模型文件
        ↓ TensorRT Builder 分析
        ↓ 层融合、算子选择、精度校准
        ↓ 针对你的具体 GPU 编译
        ↓
    Engine 文件（.engine/.plan）← 保存到磁盘

阶段二：推理（在线，极快，反复做）
    加载 Engine 文件
        ↓ 创建执行上下文
        ↓ 输入数据 → 异步推理 → 输出结果
```

**精度这件事（重要！）**：

| 精度 | 每个数字占用空间 | 速度 | 精度损失 |
|------|-----------------|------|----------|
| FP32 | 32 位（4 字节）| 基准 | 无 |
| FP16 | 16 位（2 字节）| 2x | 极小，可忽略 |
| INT8 | 8 位（1 字节）| 4x | 小，需校准 |
| FP8  | 8 位（1 字节）| 4x | 更小，Hopper 专属 |

实际生产中，**FP16 是最常用的选择**——速度翻倍，精度几乎没有损失。

**新手可以用这个工具快速上手**：
```bash
# 把 ONNX 模型转成 TensorRT engine 并测性能
trtexec --onnx=model.onnx --fp16 --saveEngine=model.engine
```

---

### 🔵 第五站：NCCL（多 GPU 通信）

**参考文件**：`nccl/notes.md`、`nccl/vocabulary.md`

**什么时候需要 NCCL**：当你有多块 GPU 需要协作训练一个大模型时。

**最重要的概念：AllReduce**

想象 4 台电脑（4 块 GPU）分别计算了梯度，现在需要把梯度"合并"后分发给每台电脑：

```
GPU 0: 梯度 = [1, 2, 3]
GPU 1: 梯度 = [4, 5, 6]
GPU 2: 梯度 = [7, 8, 9]
GPU 3: 梯度 = [10, 11, 12]
                ↓ AllReduce（求和）
所有 GPU 都得到: [22, 26, 30]
```

这就是 `AllReduce`——所有人贡献，所有人得到同样的结果。深度学习训练里每一步都要做这个操作。

**NCCL 的其他操作（白话版）**：
- `Broadcast`：一个人说给所有人听
- `Reduce`：所有人说给一个人听（汇总）
- `AllGather`：所有人把自己的那一份拼成完整的，每个人都有完整版
- `ReduceScatter`：先汇总再各取一段（用于大模型并行）

**新手只需记住**：单机单卡不需要 NCCL；多卡训练大模型时，NCCL 自动在后台工作，PyTorch 和 TensorFlow 都默认集成了它。

---

### 🟣 第六站：Triton Inference Server（部署 AI 服务）

**参考文件**：`triton/notes.md`（注意：这是 Triton 推理服务器，不是 NVIDIA Triton 编程语言）

**类比**：你训练好了一个模型，就像做好了一道菜；Triton 就是餐厅——负责接订单、调度厨师、送餐，让这道菜能服务很多人。

**Triton 解决的问题**：
- 模型保存在哪里？→ 统一的"模型仓库"（Model Repository）
- 怎么接收用户请求？→ HTTP/gRPC 接口
- 怎么高效利用 GPU？→ 动态批处理（Dynamic Batching）
- 怎么处理多个模型的流水线？→ Ensemble 模型

**最简单的部署流程**：

```
1. 把模型文件按规定格式放好
   model_repository/
   └── my_model/
       ├── config.pbtxt        ← 配置文件（说明输入输出格式）
       └── 1/
           └── model.plan      ← TensorRT 引擎文件

2. 启动 Triton 服务器
   tritonserver --model-repository=/path/to/model_repository

3. 发送请求
   curl -X POST http://localhost:8000/v2/models/my_model/infer ...
```

**动态批处理是什么**：多个用户同时发请求，Triton 把他们的请求"打包"成一批一起送给 GPU 处理，而不是一个一个处理——GPU 利用率大幅提高。

---

### 🟤 第七站：NIM、NeMo、Dynamo（高级 AI 工具）

**参考文件**：`ai-software/` 目录下各文件

**快速区分这三个**：

| 工具 | 比喻 | 用途 |
|------|------|------|
| **NeMo** | 厨师培训学校 | 训练/微调大模型 |
| **NIM** | 外卖平台 | 一键部署大模型成 API 服务 |
| **Dynamo** | 快递调度中心 | 高并发 LLM 推理的编排引擎 |

**NIM（NVIDIA Inference Microservice）最好理解**：
```
以前：自己搭模型 + 配置 TensorRT + 写 API 接口 → 需要几天
现在：docker pull nvcr.io/nim/meta/llama3... → 几分钟跑起来 LLaMA
```
NIM 把模型容器化，内置了最优的推理配置，暴露 OpenAI 兼容的 API——你甚至可以把 OpenAI 的客户端代码直接对准 NIM 使用。

**NeMo 适合谁**：需要用自己的数据微调（Fine-tune）大模型的团队，支持 LoRA、全量微调、RLHF 等训练方式。

**Dynamo 适合谁**：需要服务超高并发 LLM 请求的大型部署场景，自动做 Prefill/Decode 拆分（P/D 分解）以提高 GPU 利用率。

---

### ⚪ 第八站：Omniverse 与 Isaac Sim

**参考文件**：`omniverse/notes.md`、`omniverse/vocabulary.md`

**Omniverse 是什么**：NVIDIA 的"虚拟协作平台"，让多个用户实时编辑同一个 3D 场景，核心是 USD（一种 3D 场景文件格式）+ RTX 光追渲染。

**Isaac Sim 是什么**：Omniverse 上的机器人仿真器，特别之处：
- 物理仿真用的是真实物理引擎（PhysX/Newton），不是游戏引擎简化版
- 传感器仿真（摄像头、激光雷达）基于 RTX 渲染，非常逼真
- 和 ROS 2 无缝集成，可以直接对接真实机器人软件栈

**为什么机器人公司需要 Isaac Sim**：
```
真实世界训练机器人 → 昂贵、危险、缓慢（一次碰撞 = 损失）
Isaac Sim 仿真训练 → 便宜、安全、1000 倍速（并行仿真）
```

**OmniGraph 是什么**：Isaac Sim 里的"可视化编程"，用连线方式写逻辑，不需要写代码就能让场景里的物体有行为（比如：检测到碰撞 → 发送信号 → 停止马达）。

---

## 🎯 推荐的"最小学习集"（30 分钟能理解的版本）

如果你只有 30 分钟，先搞懂这 7 件事：

1. **GPU 的核心优势** = 并行计算（同时做很多简单操作）
2. **CUDA** = 让你写代码"命令 GPU 干活"的编程接口
3. **TensorRT** = 把 AI 模型压缩优化，让推理速度快几倍
4. **cuDNN** = 神经网络常用操作的高速函数库（TensorRT 底层用到它）
5. **NCCL** = 多块 GPU 一起训练时的通信工具
6. **Triton** = AI 模型的"外卖服务器"，负责接请求、调度 GPU
7. **NIM** = 预先打包好的 AI 微服务，拉一个 Docker 就能用

---

## 📋 学习检查清单

完成一个阶段后用这个自测：

### CUDA 阶段
- [ ] 我能解释 thread/block/grid 的区别
- [ ] 我知道 shared memory 和 global memory 的速度差异
- [ ] 我理解为什么要避免 warp divergence
- [ ] 我知道什么是 memory coalescing

### TensorRT 阶段
- [ ] 我能解释 build phase 和 runtime phase 的区别
- [ ] 我知道 FP32/FP16/INT8 各自的权衡
- [ ] 我能用 `trtexec` 把一个 ONNX 模型转成 engine
- [ ] 我理解 dynamic shapes 是什么以及为什么需要它

### 生产部署阶段
- [ ] 我能搭建一个 Triton 推理服务
- [ ] 我知道 dynamic batching 如何提高 GPU 利用率
- [ ] 我能解释 NIM 和自己部署模型的区别

---

## 📁 本仓库文件结构说明

```
field_voca/nvidia/
├── PROGRESS.md         ← 整体学习进度（你现在看的）
├── LEARNING_GUIDE.md   ← 本文件
├── cuda/
│   ├── notes.md        ← 精读笔记（详细）
│   └── vocabulary.md   ← 词汇表（术语解释）
├── tensorrt/           ← 同上结构
├── cudnn/
├── nccl/
├── triton/
├── hardware/
├── omniverse/
└── ...（其他领域）
```

**建议使用方式**：
1. 先看某个领域的 `notes.md`（有流程图、对比表、代码示例）
2. 遇到不认识的术语，查 `vocabulary.md`（有中英文对照和官方定义）
3. 想了解更多，参考 `notes.md` 顶部的文档链接去官网深入

---

## 🔗 配套官方文档链接

| 领域 | 官方文档入口 |
|------|-------------|
| CUDA | https://docs.nvidia.com/cuda/ |
| TensorRT | https://docs.nvidia.com/deeplearning/tensorrt/latest/ |
| cuDNN | https://docs.nvidia.com/deeplearning/cudnn/latest/ |
| NCCL | https://docs.nvidia.com/deeplearning/nccl/user-guide/ |
| Triton | https://docs.nvidia.com/deeplearning/triton-inference-server/ |
| NIM | https://docs.nvidia.com/nim/ |
| Isaac Sim | https://docs.isaacsim.omniverse.nvidia.com/ |
| Omniverse Kit | https://docs.omniverse.nvidia.com/kit/ |

---

*本指南基于 2026-07-21 整理的词条库，共 ~1150 个术语词条覆盖 NVIDIA 全栈技术。*
