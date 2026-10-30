---
tags: [software, robotics, ros2, spacemit]
domain: software
aliases: [ROS2, ROS 2, Robot Operating System 2]
status: approved
created: 2026-08-06
updated: 2026-10-29
source: C:\Users\luojunlin\docs-ros\zh\
superseded_by:
---

# ROS 2

**中文译名**：机器人操作系统 2

## 定义

ROS 2（Robot Operating System 2）是机器人操作系统第 2 版，提供中间件框架、通信机制与工具链，用于机器人与嵌入式系统的软件架构与模块协作。

## 为什么重要

- **标准化框架**：机器人软件开发的事实标准
- **模块化设计**：感知、规划、控制模块可复用
- **分布式通信**：DDS 中间件，支持多设备协同
- **丰富生态**：大量开源算法包与驱动

## 主要功能

- **通信机制**：节点间消息发布/订阅（Topic）、服务调用（Service）、动作（Action）
- **硬件抽象**：统一传感器与执行器接口
- **可视化工具**：RViz2（3D 可视化）、rqt（调试工具）
- **仿真支持**：Gazebo 集成

## 使用场景

- 移动机器人（导航、建图、避障）
- 机械臂控制
- 无人机/无人车
- 服务机器人

## 在 SpacemiT 中

SpacemiT 提供 ROS 2 RISC-V 适配和完整的 **AI Robot SDK** 机器人解决方案。

### 平台支持

| 芯片 | 当前支持的 ROS 2 版本 | 推荐操作系统 |
|------|---------------------|-------------|
| **K3** | Humble、Jazzy | Bianbu 4.x（4.0.1 起）|
| **K1** | Humble、Jazzy | Bianbu 2.3.x |

### AI Robot SDK

**AI Robot** 是面向下一代智能机器人的软硬件协同解决方案，基于 K1 芯片与自研 Bianbu OS，结合 ROS2 运行框架，集成 MediaEngine、RVV-Opt 优化库与 RDK 套件。

#### 核心组件

**1. RDK（ROS2 Dev Kits）**
- **集成 AI 开发工具链**：内置量化工具链、Model Zoo 及示例代码
- **适配主流开源大模型**：支持 DeepSeek、Qwen、Hugging Face Smol 系列
- **完整的语音处理解决方案**：ASR、TTS、多麦克风阵列前处理、声源定位
- **开源 ROS2 服务组件**：视觉感知、语音交互、传感器接入、定位导航、LLM 任务编排

**2. MediaEngine**
- **IO HUB 数据枢纽**：兼容多种协议的相机设备，数据零拷贝传输
- **软硬结合的 Pipeline 加速**：充分调度芯片内置硬件加速单元
- **功能模块化编程模型**：数据采集、编码压缩、模型推理、视频显示
- **低延迟直显机制**：满足高实时性应用需求
- **OpenCL 加速库支持**：畸变矫正、电子防抖（EIS）

**3. RVV-Opt Libs（RISC-V Vector Optimization Libraries）**
- **基础计算库**：OpenCV、OpenVML、Eigen/OpenBLAS，基于 256-bit RVV 优化
- **AI 算子库**：基于 IME 标准的自定义 AI 指令
- **AI 加速推理后端**：无缝接入 ONNXRuntime、Llama.cpp
- **高性能计算语言**：基于 OpenAI Triton 深度适配 RISC-V 后端

#### 方案特点

- **支持丰富的 Model Zoo**：大模型应用、语音处理及传统视觉 AI 模型
- **大模型应用**：兼容 Qwen2 系列、DeepSeek-1.5b、SomlLM2 系列，端到端语音对话延时低于 1.5 秒
- **语音处理**：涵盖 VAD、ASR、TTS、DOA、NS 及多麦克风声源定位
- **传统 AI 视觉**：目标检测、分类、追踪等深度优化基础模型
- **定位与建图**：支持激光导航、单目视觉 VSLAM、双目里程计方案
- **支持 Linux PreemptRT**：兼容工业控制协议 EtherCAT、CAN、RS485/232
- **低延时视频通路**：支持最高 1080P@120FPS 编码

#### 方案优势

- **开发灵活性**：Linux SDK 系统源码全面开放，配备丰富调试工具
- **成本优势**：K1 端侧承担基础 AI 推理任务，降低对云端算力服务器的依赖
- **性能平衡**：结合端侧轻量推理与云端高算力计算
- **隐私增强**：通过本地算力就地处理隐私敏感任务（如语音转文字 ASR）

## 相关词条

- [[../ai/AI-SDK]] — AI 推理与 ROS 2 集成
- [[../hardware/products/K1]] — K1 支持 ROS 2
- [[../hardware/products/K3]] — K3 支持 ROS 2
- [[Bianbu]] — 预装 ROS 2 的官方操作系统

## 来源文档

- SpacemiT 文档中心：Software / SDK / ROS 2
- [AI Robot SDK 文档](https://www.spacemit.com/community/document?lang=zh&nodepath=software/SDK/ros/root_overview.md)
- ROS 2 官网：https://docs.ros.org/
