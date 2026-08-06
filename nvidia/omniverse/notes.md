---
tags: [nvidia, omniverse, notes]
domain: omniverse
last_updated: 2026-07-21
source: https://docs.omniverse.nvidia.com/
---

# Omniverse 学习笔记

> 文档入口：
> - Platform Overview: https://docs.omniverse.nvidia.com/
> - Kit SDK Manual: https://docs.omniverse.nvidia.com/kit/docs/kit-manual/latest/

---

## 一句话总结

Omniverse = **USD 作为通用数据层** + **Kit SDK 作为应用构建框架** + **RTX 作为渲染引擎** + **Nucleus 作为协作后端**。开发者用 Kit Extension 组合出具体应用（USD Composer、Isaac Sim 等），所有组件都以可插拔扩展的形式存在。

---

## Kit SDK 组件全景

```
Omniverse Kit SDK
├── USD/Hydra          — 场景描述标准 + 渲染抽象层
│   └── omni.usd       — C++/Python API，封装 USD + Client Library
├── Omniverse Client Library — 与 Nucleus 服务器/本地文件系统通信
├── Carbonite SDK      — 跨平台核心 C++ 基础设施（插件/线程/设置/资产等）
│   └── Carbonite Plugin — 动态加载的 C 风格接口共享库
├── Omniverse RTX Renderer — 通过 Hydra 连接 USD 与 RTX 光线追踪
├── Scripting          — 内嵌 Python，可访问全部插件/USD API
└── omni.ui            — 基于 Dear ImGui 的 Python UI 框架
```

## Kit 架构分层

```
kit.exe / IApp (Kit Kernel)
    │  最小核心：扩展管理器 + 基础接口
    ▼
omni.kit.app (omni::kit::IApp)
    │  Carbonite启动 + 扩展管理器 + 事件系统 + 更新循环 + 设置 + Python运行时
    ▼
Kit Extensions (可依赖彼此，版本化，可运行时启用/禁用)
    │  例：omni.kit.usd / omni.kit.window / omni.kit.rendering / user.tool
    ▼
具体应用 (USD Composer / USD Explorer / Isaac Sim / 自定义应用)
```

**核心设计哲学**："一切皆扩展"（everything is an extension）。这是为了同时支持两种极端场景：
- 无窗口的离线批处理服务（如批量转换 USD 文件）
- 富交互式 RTX 应用（完整 GUI + 实时渲染）

依赖解析是自动化的：用户扩展只需声明直接依赖（如 `omni.kit.usd`），`omni.kit.app` 会自动解析并按正确顺序加载整棵依赖树。

### 两种典型运行模式对比

| 模式 | 依赖树 | 特点 |
|------|--------|------|
| CLI 工具 | `user.tool` → `omni.kit.usd` + `omni.kit.app` | 无 UI，处理完退出 |
| GUI CLI 工具 | 额外引入 `omni.kit.rendering`/`omni.kit.window`/`omni.kit.ui` | 依赖 UI 会展开整棵渲染/窗口依赖树 |

---

## 数据流：USD → Hydra → 渲染器

```
USD Stage (场景数据)
    │ Hydra (渲染抽象层)
    ├──► HDStorm (Pixar 参考渲染器)
    ├──► Omniverse RTX Renderer (光线追踪，生产级)
    └──► IRay (另一渲染后端)
```

USD 本身不绑定任何渲染器；Hydra 的作用是让同一份场景描述可以被多种渲染器消费，这也是 Omniverse 支持 GL/Vulkan/DX12 多图形 API 和多 Scene Delegate 并存的架构基础。

---

## 协作与资产流

```
本地应用 (Kit-based App)
    │ Omniverse Client Library
    │   - AssetResolver 解析 omniverse:// URI
    │   - Live Edit 模式的 SDF FileFormat 插件
    ▼
Nucleus 服务器
    │   - USD 资产版本控制
    │   - 多用户实时同步
    ▼
其他连接的客户端 (多用户协同编辑同一场景)
```

Live Edit 模式依赖 Client Library 中的专用 SDF FileFormat 插件，实现多用户对同一 USD Layer 的实时增量同步，而不是简单的整文件覆盖。

---

## 待深入阅读

- [ ] Nucleus Server 部署与权限管理
- [ ] PhysX / Newton 物理引擎集成 API 细节
- [ ] OmniGraph Node Library 全量节点参考
- [ ] Isaac Lab 训练框架深入

---

## 扩展开发（Extension Development）精读

> 来源：https://docs.omniverse.nvidia.com/kit/docs/kit-manual/latest/guide/extensions_advanced.html

### 一句话总结

扩展 = **一个带 `extension.toml` 配置的文件夹**。扩展系统扫描配置、解析依赖图、按拓扑顺序加载 Python 模块/Carbonite 插件/设置，支持热重载、版本化发布、依赖可选化——本质上是一个专为运行时插件生态设计的轻量级包管理器。

### 扩展 = 文件夹 + 配置

```
[extensions-folder]
└── omni.appwindow
    ├── bin/windows-x86_64/debug/omni.appwindow.plugin.dll   ← Carbonite 插件
    ├── config/extension.toml                                 ← 配置文件（唯一必需）
    └── omni/appwindow/__init__.py                             ← Python 模块
```

**最小合法扩展**：一个空的 `extension.toml` 即可，没有任何强制字段。

### 依赖解析：拓扑排序 + 语义化版本

```
用户声明: user.tool 依赖 omni.kit.usd + omni.kit.app
    │
    ▼
omni.kit.app 自动展开整棵依赖树
    │
    ▼
按拓扑顺序启动：被依赖者先启动，依赖者后启动
    （禁用时反过来：依赖者先禁用）
```

**两种典型运行模式**（复习）：
- CLI 工具：`user.tool → omni.kit.usd + omni.kit.app`，无 UI，跑完退出
- GUI 工具：额外拉入 `omni.kit.rendering`/`omni.kit.window`/`omni.kit.ui`，依赖树因 UI 而急剧展开

**版本约束求解**：多个扩展可能对同一依赖给出不同版本要求（如 `^1.2.0` 和 `~1.3`），解析器需要找到同时满足所有约束的版本；找不到就解析失败，扩展无法启用。优先复用本地已下载版本，避免"仅仅因为注册中心同步了一次"就意外升级一大批扩展。

### 热重载机制

```
文件系统变化（fswatcher 监控）
    │
    ▼
扩展被禁用 → 重新启用（可能牵动整个依赖子树）
    │
    ▼
Python 代码热编辑生效 / C++ 插件热编译生效
```

`core.reloadable = false` 可以关闭某个扩展的热重载能力，但代价是它的所有依赖者也会被强制标记为不可重载——这是为了保证依赖链的一致性。

### 依赖可选化：optional dependency

```
foo 声明: bar 是可选依赖（optional = true）
    │
    ├─ 只启动 foo → bar 不会被自动启用
    │
    └─ 显式启动 foo + bar → 两者都启用，且保证 bar 先于 foo 启动
```

这个机制让"扩展 A 支持但不强制要求扩展 B"成为一等公民设计，而不需要在代码里写一堆 `try/except ImportError`。

### 与前文 Kit 架构笔记的关系

结合本文件前半部分的"Kit 架构分层图"，`extension.toml` 的 `[dependencies]` 段正是驱动那张依赖树自动展开图的数据来源；`omni.kit.app` 读取这些声明，完成拓扑排序和加载。理解扩展系统本质上就是理解 Kit SDK 如何把"一切皆扩展"的设计哲学落地为具体的文件格式和运行时行为。

---

## OmniGraph 精读

> 来源：https://docs.omniverse.nvidia.com/extensions/latest/ext_omnigraph.html

### 定位：让静态 USD 世界"活"起来

OmniGraph 是 Omniverse 的可视化脚本系统，专门解决"USD 场景本身是静态数据，如何添加行为和交互"的问题。它不是单一图类型，而是多种图系统（形变器、粒子系统、骨骼动画、事件驱动逻辑）共用的统一框架。

```
静态 USD 场景（几何、材质、层级）
    │  + OmniGraph
    ▼
带行为和交互的动态世界
    （事件响应、持续计算、粒子/骨骼驱动）
```

### 两种图类型的选择

| 图类型 | 求值方式 | 适用场景 |
|--------|----------|----------|
| Action Graph | 事件驱动，触发时才执行 | 交互逻辑、按钮响应、状态机 |
| Push Graph | 每帧持续求值 | 物理驱动可视化、连续动画 |

选择依据很直接：如果逻辑是"发生了什么就响应什么"，用 Action Graph；如果逻辑是"每一帧都要重新算一遍"，用 Push Graph。

### 核心抽象：Node → Attribute → Connection

```
Node（节点，图的顶点）
    │ 包含
    ▼
Attribute（属性：输入/输出/状态三类）
    │ 通过
    ▼
Connection（连接，图的边，有向依赖关系）
    │ 构成
    ▼
求值网络（Authoring Graph 编写 → Execution Graph 实际运行）
```

**Authoring Graph vs Execution Graph**：开发者在编辑器里摆节点、连线操作的是 Authoring Graph；底层编译后用于实际按依赖顺序求值执行的是 Execution Graph。日常开发时说"图"通常默认指前者。

**架构设计目标**：官方文档强调 OmniGraph 的图表示"在单机上表现良好，也能在多节点数据中心场景下不改变图的表示方式就发挥全部算力"——即同一份图定义可以从单机小规模场景扩展到分布式大规模计算，这是它区别于普通视觉脚本工具的关键设计考量。

---

## Isaac Sim 精读

> 来源：https://docs.isaacsim.omniverse.nvidia.com/latest/

### 定位：机器人仿真在 NVIDIA 全栈机器人生态中的位置

```
01 构建场景 & 装配机器人   →  Isaac Sim（仿真器本身）
02 训练 RL/IL 策略（可选） →  Isaac Lab（训练框架，并行环境）
03 大规模评估策略（可选）  →  Lab-Arena（策略基准测试）
04 运行集成 SIL 测试       →  Isaac Sim（对接 ROS 2/Isaac ROS 软件栈）
```

Isaac Sim 是这条链路的起点和终点：既是场景/机器人装配的地方，也是最终跑软件在环（SIL）测试、验证真实机器人软件栈的地方。中间的训练和评估环节（Isaac Lab、Lab-Arena）是可选的，取决于任务是否需要 RL/IL 策略。

### 技术栈组成

```
Isaac Sim
├── RTX          — 渲染（物理级光线追踪）
├── PhysX / Newton — 物理仿真（两种后端可切换）
├── OmniGraph    — 行为/通信图（继承自 Omniverse Kit）
├── Replicator   — 合成数据生成（Synthetic Data Generation）
└── OpenUSD      — 场景表示（与 Omniverse 生态统一）
```

Isaac Sim 本质上是 Omniverse Kit SDK 之上的一个垂直应用：复用了 RTX 渲染、OmniGraph 行为系统、USD 场景表示，额外叠加了机器人专属的物理仿真后端和传感器仿真能力。

### 四阶段循环工作流

```
Import（导入）
  资产来源：URDF / MJCF / Onshape CAD / 原生 USD / NuRec（现实重建）
    │
    ▼
Configure（配置）
  材质、传感器、场景、语义标注、机器人物理参数、通信图
    │
    ▼
Simulate（仿真）
  物理步进、传感器输出、Replicator 随机化捕获、控制回路
    │
    ▼
Connect/Deploy（连接/部署）
  → 数据集导出给训练管线
  → 或对接外部机器人软件栈做预硬件验证（SIL）
```

四个阶段都围绕同一个"共享 Isaac Sim 场景"（一份 USD 场景 + 物理状态 + 传感器 + 语义 + 图，一个运行时），这是官方文档反复强调的设计原则：资产准备、机器人/场景配置、仿真执行、软件栈连接全部复用同一份场景表示，不需要在不同阶段之间转换数据格式。

### 开源生态

Isaac Sim 采用 Apache 2.0 开源许可，代码托管在 GitHub（`github.com/isaac-sim`），也可通过 `pip install isaacsim` 直接安装，体现出与传统"重型仿真软件"不同的、更贴近 Python/ROS 生态开发者习惯的分发方式。
