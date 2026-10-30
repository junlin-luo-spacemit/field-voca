---
tags: [tools, ide, studio, web, spacemit]
domain: tools
aliases: [SpacemiT Studio, Studio, 进迭时空工作台]
status: approved
created: 2026-08-06
updated: 2026-10-29
source: C:\Users\luojunlin\docs-tool\zh\studio\
superseded_by:
---

# SpacemiT Studio

**官方名称**：SpacemiT Studio  
**访问地址**：https://studio.spacemit.com/

## 定义

**SpacemiT Studio** 是由进迭时空打造的 RISC-V AI CPU 开发生态一站式 Web 工作台，集成镜像管理、系统烧录、设备管理、终端调试、云端编译、AI 开发和应用部署等能力，支持从系统安装到应用上线的完整开发流程。

它为 RISC-V 开发提供统一入口，帮助开发者在同一界面中完成设备准备、软件开发和应用部署。

## 为什么重要

- **一站式开发**：将设备管理、终端工具、文件管理、Web IDE、远程桌面和应用中心整合在同一界面
- **智能烧录**：向导式系统安装，自动识别板型并推荐镜像版本
- **云端编译**：提供云端编译服务，支持 Makefile、CMake 和 Meson，保证跨平台一致性
- **AI 原生**：集成 AI 开发助手、RAG 知识库和预置 Skill 能力库
- **零安装**：Web 应用，无需本地安装，支持 Windows、Linux、macOS

## 系统要求

- **Windows**: Windows 10+
- **Linux**: Ubuntu 20.04+
- **macOS**: macOS 11+
- **浏览器**: 现代浏览器（Chrome、Firefox、Edge、Safari）

## 核心功能

### 1. 智能烧录系统

**向导式系统安装**：
- 自动识别板型（MUSE Pi、MUSE Book、MUSE Card 等）
- 推荐镜像版本（Bianbu、OpenWrt、Buildroot、OpenHarmony）
- 在线镜像管理（下载、缓存、版本选择）
- 一键烧录

**预置配置能力**：
- SSH 密钥预配置
- Wi-Fi 凭证写入
- 主机名设置
- 用户密码配置

**量产支持**：
- 量产卡制作
- EEPROM 写号

### 2. 一体化开发工作台

**设备管理**：
- 设备扫描与连接
- 多设备同时管理
- 设备状态监控

**终端工具**：
- Web 终端（SSH）
- 命令行交互
- 日志查看

**文件管理**：
- 远程文件浏览
- 文件上传/下载
- 文件编辑

**Web IDE**：
- 在线代码编辑
- 语法高亮
- 多文件管理

**远程桌面**：
- 图形界面访问
- VNC/RDP 支持

**应用中心**：
- 应用商城
- 一键安装应用
- 应用管理

### 3. 云端开发环境

**云端编译服务**：
- 支持 **Makefile**
- 支持 **CMake**
- 支持 **Meson**
- 跨平台一致性保证

**优势**：
- 无需本地配置交叉编译工具链
- 统一编译环境
- 适合 Windows 开发者
- 快速构建

### 4. AI 原生开发

**AI 开发助手**：
- 智能代码补全
- 代码解释与建议
- 错误诊断

**RAG 知识库**：
- SpacemiT 文档检索
- 技术问答
- 示例代码查找

**预置 Skill 能力库**：
- AI 模型部署
- 端侧应用开发
- 常见任务模板

### 5. 在线镜像管理

**镜像库**：
- Bianbu GNOME/LXQt
- OpenWrt
- Buildroot
- OpenHarmony

**版本管理**：
- 最新版本推荐
- 历史版本选择
- 版本更新通知

## 使用流程

### 快速上手

```plaintext
1. 访问 Studio
   └─ 打开 https://studio.spacemit.com/

2. 注册/登录
   ├─ 手机号注册（中国国内）
   ├─ 邮箱注册
   └─ 第三方登录（微信、QQ、GitHub、Gitee）

3. 安装驱动
   ├─ 首次启动自动提示
   ├─ 下载对应平台驱动
   └─ 安装并重启服务

4. 连接设备
   ├─ 设备进入烧录模式
   ├─ USB 连接电脑
   └─ Studio 扫描设备

5. 烧录系统
   ├─ 选择板型
   ├─ 选择镜像
   ├─ 配置预置参数（SSH、Wi-Fi 等）
   └─ 开始烧录

6. 开发调试
   ├─ 终端连接
   ├─ 文件管理
   ├─ 代码编辑
   └─ 云端编译
```

## 适用场景

### 系统准备与设备初始化
- 开发板烧录
- 网络配置
- 基础环境设置

### 日常开发与调试
- 代码编写
- 编译构建
- 终端调试
- 文件同步

### AI 应用构建与部署
- AI 模型加载
- 模型调试
- 端侧部署
- 性能优化

## 登录方式

### 注册账号
- **手机号注册**：仅支持中国国内手机号
- **邮箱注册**：国际通用

### 登录方式
- **短信登录**：手机验证码
- **密码登录**：账号密码
- **第三方登录**：
  - 微信账号
  - QQ 账号
  - GitHub 账号
  - Gitee 账号

## 驱动安装

**首次启动**：
- 首页显示"服务未启动"提示
- 驱动安装引导窗口自动弹出
- 选择对应平台驱动下载
- 安装完成后重启服务

**支持平台**：
- Windows（自动安装）
- Linux（AppImage 或系统包）
- macOS（DMG 安装包）

## 界面概览

**主界面包含**：
- **导航栏** — 功能模块入口
- **工具栏** — 常用操作快捷方式
- **设备区域** — 已连接设备列表
- **工作区** — 主要操作区域（终端、编辑器、文件管理器等）
- **AI 工作区** — AI 助手交互区域

## 与 TitanTools 对比

| 特性 | SpacemiT Studio | TitanTools |
|-----|----------------|------------|
| **平台** | Web 应用 | 桌面应用（Windows/Linux） |
| **安装** | 无需安装（仅驱动） | 需要安装客户端 |
| **烧录** | ✅ 智能烧录 | ✅ 完整烧录工具 |
| **设备管理** | ✅ 在线管理 | ✅ 本地管理 |
| **开发工具** | ✅ IDE、终端、文件管理 | ❌ 无 |
| **云端编译** | ✅ 支持 | ❌ 无 |
| **AI 助手** | ✅ 集成 | ❌ 无 |
| **量产** | ✅ 支持 | ✅ 完整量产工具 |
| **适用场景** | 开发、调试、部署 | 烧录、量产 |

**选择建议**：
- **开发阶段** → 使用 **SpacemiT Studio**（一站式开发）
- **量产阶段** → 使用 **TitanTools**（批量烧录、多机写号）

## 支持的设备

**MUSE 系列**：
- MUSE Pi
- MUSE Pi Pro
- MUSE Card
- MUSE Book
- MUSE Paper / Paper2

**开发板**：
- BPI-F3
- deb1 / deb2

## 在 SpacemiT 生态中

- **与 TitanTools 互补**：Studio 专注开发，TitanTools 专注量产
- **与 AI SDK 集成**：直接部署 AI 模型
- **与官方镜像仓库集成**：一键下载最新镜像
- **与文档中心集成**：RAG 知识库实时检索

## 相关词条

- [[Flashing-Tool]] — TitanTools 桌面刷机工具
- [[Cross-Compilation-Toolchain]] — 交叉编译工具链（Studio 云端编译可替代）
- [[../software/Bianbu]] — Bianbu 系统（Studio 支持烧录）
- [[../software/OpenWrt]] — OpenWrt 固件（Studio 支持烧录）
- [[../software/Buildroot]] — Buildroot 固件（Studio 支持烧录）
- [[../software/OpenHarmony]] — OpenHarmony 固件（Studio 支持烧录）
- [[../ai/AI-SDK]] — AI SDK（Studio 集成 AI 开发）

## 来源文档

- SpacemiT Studio 官方文档：C:\Users\luojunlin\docs-tool\zh\studio\
- SpacemiT Studio 官网：https://studio.spacemit.com/
- 产品概述：docs-tool/zh/studio/overview.md
- 快速入门：docs-tool/zh/studio/quick_start.md
- 用户指南：docs-tool/zh/studio/user_guide/
