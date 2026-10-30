---
tags: [software, sdk, spacemit]
domain: software
aliases: [SDK, Software Development Kit, 软件开发套件]
status: approved
created: 2026-08-06
updated: 2026-10-29
source: C:\Users\luojunlin\docs-buildroot\zh\k1_buildroot\
superseded_by:
---

# SDK

**中文译名**：软件开发套件

## 定义

SDK（Software Development Kit，软件开发套件）包含工具链、库、示例与文档，便于在目标平台上开发与部署应用。

## SpacemiT SDK 组成

完整的 BSP（Board Support Package），包括：

1. **Bootloader**
   - OpenSBI（RISC-V Supervisor Binary Interface）
   - U-Boot / UEFI

2. **Linux Kernel**
   - 适配 K1/K3 的内核树
   - 驱动支持（GPU、VPU、ISP、AI 核心）

3. **Rootfs**
   - 根文件系统（Bianbu / Buildroot）
   - 预装驱动与库

4. **工具链**
   - 交叉编译工具链（GCC / LLVM）
   - 调试工具（GDB、OpenOCD）

5. **示例代码**
   - 硬件功能演示
   - API 使用示例

## 开发流程

```
1. 安装 SDK 与工具链
2. 配置内核与设备树
3. 编译内核、bootloader、rootfs
4. 烧录到目标设备
5. 开发应用程序
6. 调试与优化
```

## 与其他 SDK 的关系

- **SDK**：通用软件开发套件（BSP + 工具链）
- **[[../ai/AI-SDK]]**：AI 专用开发套件（模型部署、推理优化）
- **[[../ai/LLM-SDK]]**：大语言模型专用套件

## 相关词条

- [[Cross-Compilation-Toolchain]] — 交叉编译工具链
- [[Bianbu]] — 官方 Linux 发行版
- [[Buildroot]] — 自定义系统构建
- [[../hardware/products/K1]] — K1 SDK
- [[../hardware/products/K3]] — K3 SDK

## 来源文档

- SpacemiT 文档中心：Software / SDK
