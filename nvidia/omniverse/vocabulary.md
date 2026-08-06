---
tags: [nvidia, omniverse, vocabulary]
domain: omniverse
last_updated: 2026-07-21
part: Overview ✅ / Kit SDK Architecture ✅
source: https://docs.omniverse.nvidia.com/
---

# Omniverse 词汇表

> 来源：https://docs.omniverse.nvidia.com/
> 定位：3D 仿真、数字孪生、工业元宇宙

---

## 核心概念

### Omniverse / Omniverse
基于 Pixar USD 和 NVIDIA RTX 的实时 3D 仿真与协作平台，支持多应用、多用户实时共同编辑。

### USD (Universal Scene Description) / 通用场景描述
Pixar 开发的开放 3D 场景表示标准，Omniverse 的数据基础。支持层（Layer）、引用（Reference）和变体（Variant）。

### digital twin / 数字孪生
物理世界对象或系统的精确虚拟副本，用于仿真、监控和预测。

### RTX / RTX
NVIDIA 实时光线追踪技术，Omniverse 的渲染核心，支持全局光照、阴影、反射的物理级仿真。

### Isaac Sim / Isaac Sim
基于 Omniverse 的机器人仿真平台，用于训练和验证 AI 机器人模型。

### PhysX / PhysX
NVIDIA 物理仿真引擎，提供刚体、布料、流体等高精度物理模拟。

### Nucleus / Nucleus
Omniverse 的实时协作服务，管理 USD 资产的版本控制和多用户同步。

### Connector / Connector
将第三方 DCC 工具（Maya、Blender、Revit 等）接入 Omniverse 生态的插件。

### Extension / Extension
Omniverse 的功能扩展单元，开发者可用 Python 编写自定义功能模块。

---

## Omniverse Kit SDK 架构

> 来源：https://docs.omniverse.nvidia.com/kit/docs/kit-manual/latest/

### Omniverse Kit / Omniverse Kit SDK
用于构建 Omniverse 应用（如 USD Composer、USD Explorer）的核心 SDK。整合 USD/Hydra、Carbonite、Omniverse RTX Renderer、Scripting、omni.ui 五大组件，开发者可自由组合或扩展它们来构建自定义应用。
> *Omniverse Kit is the SDK for building your own Omniverse applications like USD Composer and USD Explorer.*

### Hydra / Hydra
Pixar 开发的渲染抽象层，负责将 USD 场景内容流式传递给任意实现了 Hydra Scene Delegate 的渲染器（如 HDStorm、Omniverse RTX Renderer、IRay）。Kit 通过 Hydra 解耦场景描述与具体渲染实现。

### omni.usd / omni.usd 模块
构建在 USD、Kit 核心和 OmniClient 库之上的 C++ API（含 Python 绑定），提供事件/监听器、选择处理、USD Layer 管理、`USDContext`（访问主 Stage 及其 Layer、Hydra/渲染器/视口相关服务）、MDL 支持等应用级服务。

### Omniverse Client Library / Omniverse 客户端库
Kit 类客户端与 Omniverse 服务器及本地文件系统通信的库。包含：解析 `omniverse://` URI 的 USD AssetResolver、支持 Live Edit 模式的 SDF FileFormat 插件、Nucleus 服务器读写/复制/查询 API、服务器连接管理。

### Carbonite / Carbonite SDK
所有 Omniverse 应用的核心 C++ SDK，提供跨平台统一 API：插件管理、输入处理、文件访问、持久化设置管理、音频、资产加载、线程/任务管理、图像加载、本地化、同步、基础窗口系统等。

### Carbonite Plugin / Carbonite 插件
具有 C 风格接口的共享库，可在运行时动态加载/卸载。接口采用语义化版本控制并保持向后兼容；多数插件接口通过 pybind11 提供 Python 绑定。

### Omniverse RTX Renderer / Omniverse RTX 渲染器
Kit 内置的高复杂度渲染系统，通过 Hydra 连接 USD 与 RTX，同时支持多渲染器、多自定义 Scene Delegate、多 Hydra Engine（GL/Vulkan/DX12），并为 Viewport 提供 Gizmo 等控件的高帧率异步渲染。

### Kit Extension / Kit 扩展
构建在 Scripting 和 Carbonite Plugin 之上的最高层级构建块：具备版本号和运行时启用/禁用状态的软件包，可依赖其他扩展。是 Kit 应用（如 USD Composer）的基本组成单元。

### omni.ui / omni.ui
Kit 的 UI 框架，底层基于 Dear ImGui、用 C++ 实现，但只对外暴露 Python API。

### Kit Kernel (kit.exe / IApp) / Kit 内核
运行任意扩展所需的最小核心，是所有 Kit 应用的入口点，内含扩展管理器和基础接口，是将一切组件串联起来的核心。

### omni.kit.app (omni::kit::IApp) / omni.kit.app 接口
任意扩展都可使用的基础接口，是 Kit Kernel 的主要前端。包含 Carbonite 框架启动、扩展管理器、事件系统、更新循环、设置、Python 上下文/运行时。可从 Kit 可执行文件或 Python 直接运行。

### everything is an extension / 一切皆扩展
Kit 的核心设计哲学：为支持从无窗口离线处理服务到富交互 RTX 应用的极端多样性场景，Kit 采用高度模块化架构，所有功能（包括核心功能）都以扩展形式提供。

### extension dependency resolution / 扩展依赖解析
`omni.kit.app` 根据用户扩展声明的依赖关系，自动解析并按正确顺序加载/启动所需的全部扩展树，用户只需声明自己的扩展即可，无需手动枚举全部间接依赖。

---

## 扩展开发（Extension Development）

> 来源：https://docs.omniverse.nvidia.com/kit/docs/kit-manual/latest/guide/extensions_advanced.html

### extension.toml / 扩展配置文件
放在扩展根目录（或 `config` 子目录）的 TOML 配置文件，声明依赖、Python 模块、Carbonite 插件、设置、环境变量等。没有必填字段，空配置也是合法扩展（只是无效果）。

### Extension Id / 扩展标识符
格式为 `[ext_name]-[ext_tag]-[ext_version]`，如 `omni.kit.commands-1.2.3-beta.1`。`ext_tag` 是可选的实现变体标记（同一接口的不同实现，如 `bar-light` vs `bar-prototype`）。

### Extension Package Id / 扩展包标识符
扩展 ID 加构建元数据：`[ext_id]+[build_meta]`，用于区分同一扩展在不同平台/配置下的多个包（如 `+wx64.r` 表示 Windows x86_64 Release 版）。

### single file extension (.kit file) / 单文件扩展
仅含配置文件、无代码的扩展，配置文件名即扩展 ID。常用作应用入口点（`package.app = true`），展开整棵依赖树后启动完整应用。约定用 `.kit` 后缀以便双击关联 `kit.exe` 启动。

### dev paths / 开发路径
`--/app/exts/devPaths` 和 `--/app/exts/devFolders` 设置，专用于本地开发时链接扩展源码：忽略版本检查、优先于其他搜索路径，便于测试未发布的新版本。

### extension discovery / 扩展发现
扩展系统持续监控指定搜索路径，任何根目录或 `config` 子目录含 `extension.toml` 的子文件夹都被视为一个扩展；文件夹名唯一标识扩展名和 tag。

### dependency resolver / 依赖解析器
启用扩展时递归求解整棵依赖图，成功则按依赖顺序（被依赖者先启动）启用全树；禁用时顺序相反（依赖者先禁用）。解析优先选择本地已有版本而非拉取新版本，避免意外升级一大批扩展。

### core.order / 扩展启动顺序权重
当多个扩展相互独立时启用顺序未定义，可用 `core.order`（负值更早、正值更晚）软性控制启动顺序；依赖关系仍是硬约束，顺序只在依赖允许的范围内生效。

### optional dependency / 可选依赖
`optional = true` 标记的依赖：默认不会被自动启用，扩展需自行处理该依赖缺失的情况。仅当依赖树中有其他扩展将其列为非可选、或用户显式启用时才会加载。

### SemVer requirement operators / 语义化版本约束操作符
`^1.2.3`（caret，兼容更新）、`~1.2`（tilde，限制更新范围）、`=1.2.3`（精确匹配）、`>1.1`（比较）。多个扩展对同一依赖的版本约束会被解析器统一求解，找不到满足所有约束的版本则解析失败。

### hot reload / 热重载
扩展系统监控已启用扩展的文件变化，自动禁用再重新启用（可能牵动整个依赖树），支持 Python 代码热编辑和 C++ 插件热编译。`core.reloadable=false` 可关闭此行为，并会连带阻止其依赖的重载。

### extension registry / 扩展注册中心
用于发布和拉取扩展的远程仓库，默认使用 `omni.kit.registry.nucleus`（基于 Nucleus）。仅当本地缓存无法满足全部依赖时才会联系注册中心；下载即解压 zip 到缓存目录。

### IExt interface / IExt 接口
Python 侧扩展入口点接口（`omni.ext.IExt`），需实现 `on_startup(ext_id)` 和 `on_shutdown()`；C++ 侧对应 `omni::ext::IExt`，扩展启用/禁用时分别调用 `onStartup`/`onShutdown`。

### package.category / 扩展分类
UI 展示用分类标签，取值包括：`animation`、`graph`、`rendering`、`audio`、`simulation`、`example`、`internal`、`other`。

---

## OmniGraph（可视化编程节点系统）

> 来源：https://docs.omniverse.nvidia.com/extensions/latest/ext_omnigraph.html

### OmniGraph / OmniGraph
Omniverse 的可视化脚本语言，为静态的 USD 世界赋予行为和交互能力。不是单一图类型，而是多种图系统在统一框架下的组合，涵盖形变器、粒子、事件驱动图等场景。
> *OmniGraph is not a single type of graph, but a composition of many different types of graph systems under a single framework.*

### Action Graph / 动作图
事件驱动型图类型，节点响应特定事件触发执行（而非每帧持续求值），适合交互逻辑、触发器等场景。

### Push Graph / 推送图
持续求值型图类型，节点每帧不断重新计算，适合需要连续更新状态的场景（如物理驱动的可视化）。

### Authoring Graph / 编写图
用户在 OmniGraph 编辑器中创建和编辑的图表示，是开发者直接交互的层。

### Execution Graph / 执行图
Authoring Graph 编译后用于实际运行求值的内部表示，两者常被笼统称为"图"，但日常开发中"图"多指 Authoring Graph。

### Node (OmniGraph) / 节点
图系统的核心构件，由节点类型（node type）定义，可包含输入、输出和状态三类属性（Attribute）。

### Attribute / 属性
节点上具名的数据槛，含特定数据类型。属性之间可相互连接，构成求值网络。

### Connection / 连接
节点属性之间的有向依赖关系，相当于图论中的"边"（节点是"顶点"）。

---

## Isaac Sim（机器人仿真平台）

> 来源：https://docs.isaacsim.omniverse.nvidia.com/latest/

### Isaac Sim / Isaac Sim
基于 Omniverse 和 OpenUSD 的机器人仿真器，开源（Apache 2.0），支持从 URDF/MJCF/Onshape CAD/USD 导入机器人和场景，用 PhysX 或 Newton 仿真物理，加入 RTX 渲染和物理传感器，生成合成数据，为 Isaac Lab 准备训练场景，并通过 ROS 2 验证机器人软件栈。

### PhysX / PhysX 物理引擎
Isaac Sim 支持的物理仿真后端之一，提供成熟的刚体/关节/碰撞仿真。

### Newton / Newton 物理引擎
Isaac Sim 新支持的物理仿真后端，与 PhysX 并列可选，用户可在同一仿真器中切换物理后端。

### Replicator / Replicator
Omniverse 的合成数据生成（Synthetic Data Generation）扩展，用于从 Isaac Sim 场景中生成带标注的训练数据（如随机化场景、传感器捕获）。

### Isaac Lab / Isaac Lab
NVIDIA 的强化学习/模仿学习（RL/IL）训练框架，基于 Isaac Sim 准备好的机器人和场景，用并行环境训练策略。是仿真到训练流程的下一环。

### Isaac Sim Workflow（四阶段工作流）/ Isaac Sim 四阶段工作流
一套围绕共享 USD 场景的循环流程：
1. **Import**（导入）：场景/机器人/传感器/CAD/USD/NuRec 资产接入统一 USD 工作空间
2. **Configure**（配置）：材质、传感器、场景、语义、机器人物理参数、通信图
3. **Simulate**（仿真）：物理步进、传感器输出、Replicator 捕获、控制回路
4. **Connect/Deploy**（连接/部署）：导出数据集给训练管线，或对接外部机器人软件栈做预硬件验证

### SIL test (Software-in-the-Loop) / 软件在环测试
在 Isaac Sim 中用真实 ROS 2 或 Isaac ROS 机器人软件栈运行的集成测试，验证软件逻辑而不需要真实硬件，是硬件部署前的关键验证环节。

### URDF (Unified Robot Description Format) / 统一机器人描述格式
描述机器人运动学、外观、传感器等的标准 XML 格式，是 Isaac Sim 导入机器人的常见输入格式之一（还支持 MJCF、Onshape CAD、原生 USD）。

### NuRec / NuRec
NVIDIA 的资产重建（重建自现实世界数据）技术，可作为场景/资产来源接入 Isaac Sim 的 USD 工作空间。
