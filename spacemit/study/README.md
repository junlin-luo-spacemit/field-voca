---
tags: [study, index, spacemit]
domain: all
status: needs_review
created: 2026-11-16
updated: 2026-11-18
---

# SpacemiT 技术基础知识库

## 这是什么

给**技术写作团队**用的 SpacemiT 产品与生态基础课。

目标不是练写作，是**把技术底子打起来**：团队要先知道我们卖的是什么、这颗芯片里有什么、软件是怎么跑起来的、AI 能力从哪来。写文档是以后的事，等底子够了自然会写。

判断学会的标准很朴素：**不看资料，能用大白话把一件事讲明白。** 讲不明白就是还没懂。

## 素材来源：六份官方文档库

| 库 | 篇数 | 讲什么 |
|---|---:|---|
| `docs-product` | 117 | 产品线：K1 / K3 / V100 三个系列各有哪些整机形态 |
| `docs-chip` | 149 | 芯片本身：CPU 核、AI 核、内存、存储、显示、图像、互联、安全 |
| `docs-bianbu` | 104 | 面向通用计算的操作系统：桌面版、命令行版、怎么开发 |
| `docs-buildroot` | 349 | 面向嵌入式的系统：驱动、外设、多媒体、图形、内核调试 |
| `docs-ai` | 92 | AI 能力：指令集扩展、推理引擎、量化、应用工具、行业方案 |
| `docs-tool` | 45 | 开发工具：Studio、交叉编译、烧录、JTAG、Trace |

本地路径都在 `C:\Users\luojunlin\` 下，同事机器上可能不存在。

## 组织方式：按技术主线，不按库

这一点最容易搞错。

**不是"一天看一份库"。** 同一件事在六份库里都有，各讲一个侧面。比如"启动"：

- 芯片库讲 Boot ROM 和启动模式
- Buildroot 库讲 U-Boot 和设备树
- Bianbu 库讲系统镜像怎么做出来
- 产品库讲用户怎么刷机
- 工具库讲烧录工具怎么用

所以每一讲都是**一条技术主线，穿过多份库**。这样学完的东西是立体的，不是一个库里零散的知识点。

完整的课表见 [ROADMAP.md](./ROADMAP.md)。

## 一条主线的样本

以"D1 显示与图形"为例，一讲里会同时用到：

| 库 | 贡献什么 |
|---|---|
| `docs-chip` | DPU0/DPU1 的接口不对称、MIPI-DSI 与 DP/eDP 的区别 |
| `docs-buildroot` | 显示驱动框架、panel porting、GPU 驱动 |
| `docs-product` | 具体板子上引出的是 DP 还是 DSI、几个口 |
| `docs-ai` | GPU 的 OpenCL 在 AI 预/后处理里的角色 |

一讲读完，你能同时回答"芯片支持什么""驱动怎么做""这块板子里有什么"。

## 怎么用

1. 按 [ROADMAP.md](./ROADMAP.md) 的顺序走，模块一讲一讲来。
2. 每讲读完，先自己复述一遍，再做题。题目用来找概念混淆，不是门槛。
3. 讲不清就回看那一讲，不用重做整套题。
4. 遇到术语，查 [GLOSSARY.md](./GLOSSARY.md)（本课程配套术语表）或 `spacemit/glossary/`、`spacemit/atoms/`。
5. 跨会话或换机器继续，先读 [HANDOFF.md](./HANDOFF.md)。

课程怎么写、测验怎么出，见 [STYLE.md](./STYLE.md)。那份文件是硬性规范，编写时逐条对照。

## 现状

**M1、M2、M3、M4 已完成**：

### M1：产品概览

| 文件 | 内容 |
|---|---|
| [m1-01-series-naming.md](./m1-01-series-naming.md) | 产品系列与命名规律：K/P/V 三个系列、编号规则、K1 与 K3 的内核差异 |
| [m1-02-form-factors.md](./m1-02-form-factors.md) | 同一颗芯片的多种形态 |
| [m1-03-reading-specs.md](./m1-03-reading-specs.md) | 读懂规格表里的数字 |
| [quiz/m1-recap.html](./quiz/m1-recap.html) | 15 道选择题（基础与应用 10 + 认陷阱 5）+ 14 个知识点的分点自评，含错题计数与回看清单 |

### M2：芯片架构基础

| 文件 | 内容 |
|---|---|
| [m2-01-chip-subsystems.md](./m2-01-chip-subsystems.md) | 芯片由哪些子系统组成：CPU、内存、图像、视频、显示、外设 |
| [m2-02-cpu-and-memory.md](./m2-02-cpu-and-memory.md) | X100 与 A100 的区别、内存层次、带宽计算 |
| [m2-03-peripherals-overview.md](./m2-03-peripherals-overview.md) | 图像子系统、视频子系统、显示子系统的分工 |
| [quiz/m2-recap.html](./quiz/m2-recap.html) | 15 道选择题（基础与应用 10 + 认陷阱 5）+ 14 个知识点的分点自评，含错题计数与回看清单 |

### M3：内存与存储

| 文件 | 内容 |
|---|---|
| [m3-01-lpddr-and-bandwidth.md](./m3-01-lpddr-and-bandwidth.md) | LPDDR5 与带宽：内存速度怎么算、为什么是 AI 的瓶颈 |
| [m3-02-storage-interfaces.md](./m3-02-storage-interfaces.md) | 四种存储接口：QSPI、eMMC、SD、UFS 各自给谁用 |
| [m3-03-onchip-memory-and-boot.md](./m3-03-onchip-memory-and-boot.md) | 片上存储：Boot ROM 与 SRAM 各是什么、为什么启动离不开它们（启动链路见 M4） |
| [quiz/m3-recap.html](./quiz/m3-recap.html) | 15 道选择题（基础与应用 10 + 认陷阱 5）+ 14 个知识点的分点自评 |

### M4：启动流程

| 文件 | 内容 |
|---|---|
| [m4-01-boot-flow.md](./m4-01-boot-flow.md) | 启动流程概览：Boot ROM → FSBL → OpenSBI → U-Boot → Linux |
| [m4-02-boot-modes-and-flashing.md](./m4-02-boot-modes-and-flashing.md) | 启动模式与固件刷写：下载模式、正常启动、SD 卡优先、Fastboot |
| [m4-03-secure-boot-and-opensbi.md](./m4-03-secure-boot-and-opensbi.md) | 安全启动与固件保护：逐级验证、eFuse 不可逆、两类密钥 |
| [quiz/m4-recap.html](./quiz/m4-recap.html) | 15 道选择题（基础与应用 10 + 认陷阱 5）+ 14 个知识点的分点自评 |

### 配套资源

| 文件 | 内容 |
|---|---|
| [GLOSSARY.md](./GLOSSARY.md) | 术语表：带宽、内存与存储、虚拟机、RVA23、RVV 1.0、RVH 1.0、IME、SoC、LPDDR5、ISP、VPU、簇、编码压缩比、AFBC |
| [quiz/_template.html](./quiz/_template.html) | 测验模板，用于创建新模块的测验 |

M5 之后的模块按 [ROADMAP.md](./ROADMAP.md) 逐个编写。

早期按「一天一份库」写的 `day1-*` ~ `day4-*` 已全部删除，不再保留。
