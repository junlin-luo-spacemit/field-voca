---
tags: [progress, spacemit]
domain: all
date: 2026-08-08
---

# SpacemiT 文档中心进度

## 📐 架构演进历史

### 2026-08-08：架构重构，对齐 NVIDIA 模板 A

**背景**：审查发现 atoms/ 混杂了核心概念、产品、SDK、操作系统、工具等 27 个文件，领域文件夹（hardware/software/ai/tools/）全部为空，与 NVIDIA 知识库的成熟架构不一致。

**变更**：
- 参照 `../ARCHITECTURE.md` 定义的模板 A（技术文档知识库），完成重构
- atoms/ 精简为 6 个跨领域核心概念：SoC, Matrix-Extension, X60-Core, HBM, IOMMU, PMIC
- 新建 5 个领域文件夹（hardware/software/ai/tools/cloud/），每个含 vocabulary.md（术语速查）+ notes.md（学习笔记）
- 移动 20+ 个原 atoms/ 文件到对应领域；products/ 移入 hardware/products/
- 更新 atoms/README.md（说明 atoms 选择标准）和 HOME.md（新增快速导航）

**待办**（未完成，优先级从高到低）：
1. ⬜ 更新 `官方文档结构地图.md` 中约 50 处 `atoms/xxx` 路径引用为新路径（如 `hardware/products/K1`）
2. ⬜ 更新本文件下方"统计概览"和"词条迁移状态"表格，反映新结构
3. ⬜ 检查 `跨厂商术语对照表.md`、`术语规范检查清单.md`、`概念地图.md` 中的旧路径引用
4. ⬜ 校验重构后所有 `[[wikilink]]` 是否因文件移动产生断链

**同步变更**：RISC-V 知识库同期做了扁平化优化（`05-调试与工具链/{debug,platform,software}/` 提升为顶层 `05-debug/`、`05-platform/`、`05-software/`），使其三个知识库都符合 ARCHITECTURE.md 规范。

---

## 统计概览（2026-08-08 重构后）

**架构变更**：已完成模板 A 重构，对齐 NVIDIA 标准（详见上方"架构演进历史"）

- **原子词条（atoms/）**：6 个核心概念（SoC, Matrix-Extension, X60-Core, HBM, IOMMU, PMIC）
- **产品页面（hardware/products/）**：18 个
- **领域文件夹**：5 个（hardware, software, ai, tools, cloud），每个含 vocabulary.md + notes.md
- **总计文档数**：~45 个（6 atoms + 18 products + 21 领域详细页）

## 词条迁移状态（已完成 → 重构后状态）

| 原位置 | 重构后位置 | 状态 | 备注 |
|--------|-----------|------|------|
| atoms/27 个混杂文件 | 分类到 5 个领域 | ✅ 完成 | 仅保留 6 个核心概念在 atoms/ |
| atoms/products/ | hardware/products/ | ✅ 完成 | 18 个产品页移入 hardware 领域 |
| atoms/硬件相关 | hardware/*.md | ✅ 完成 | Compute-Architecture, Design-Philosophy |
| atoms/软件相关 | software/*.md | ✅ 完成 | SDK, Bianbu, OpenWrt, Buildroot, OpenHarmony, ROS2 (6 个) |
| atoms/AI相关 | ai/*.md | ✅ 完成 | AI-Platform, AI-SDK, LLM-SDK, LangChain, Ollama, localAI, LlamaIndex (7 个) |
| atoms/工具相关 | tools/*.md | ✅ 完成 | Cross-Compilation-Toolchain, Flashing-Tool, JTAG-Debugging, Trace-Tool, SpacemiT-Studio (5 个) |
| atoms/云相关 | cloud/*.md | ✅ 完成 | SpacemiT-Cloud (1 个) |

## 重点产品状态

| 产品 | 状态 | 数据来源 | 备注 |
|------|------|----------|------|
| K1 | ✅ 已补全 | Datasheet + User Manual (16章) + 硬件设计指南 + SDK文档 + FAQ | 包含完整架构、寄存器映射、内存地址表、启动配置、IO参数、开发资源索引 |
| K3 | ✅ 已补全 | Datasheet + User Manual (17章) + 硬件设计指南 + SDK文档 + FAQ | 高性能AI芯片，8×X100 CPU + 8×A100 AI核心，60 TOPS算力，完整寄存器参考 |
| 其他产品 | ⏳ 待补全 | - | 占位页面已创建，等待官方数据 |

## 领域覆盖

| 领域 | 来源页面 | 目标 | 状态 | 完成日期 |
|------|----------|------|------|----------|
| 硬件 | 文档中心硬件栏目 | 提取术语与核心概念 | ✅ 100% 完成 | 2026-08-07 |
| 软件 | 文档中心软件栏目 | 提取术语与核心概念 | 🔄 83%，缺 Eco-System | - |
| AI | 文档中心 AI 栏目 | 提取术语与核心概念 | 🔄 47%，缺口最大 | - |
| 工具 | 文档中心 Tools 栏目 | 提取术语与核心概念 | 🔄 50%，Studio 待拆分 | - |
| 云 | 文档中心云栏目 | 提取术语与核心概念 | 进行中 | - |

**详细缺口清单见 [[官方文档结构地图]]**，优先补全顺序：
1. AI › Solution（AI Robot / AI Computer 方案概览，2 项）
2. AI › Compute Software Stack（计算软件栈概览/清单/视觉库/数学库，4 项）
3. Tools › SpacemiT Studio（拆分现有 atom 为 Overview/Quick Start/User Guide/FAQ，4 项）
4. AI › Application Software Stack 的 2 个缺口（Speech SDK、OpenWebUI）
5. Software › Eco-System（1 项，待确认内容性质）
| 工具 | 文档中心工具栏目 | 提取术语与核心概念 | 进行中 | - |

## 下一步

- 继续补全剩余产品页面（需官方数据支持）
- 扩展硬件相关原子词条（寄存器、电源域、启动流程等）
- 建立词条间的交叉引用网络
