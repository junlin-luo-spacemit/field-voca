---
tags: [meta, log, changelog]
---

# 知识库变更日志

> 记录跨知识库的重大结构性变更。日期倒序排列，仅追加，不覆盖历史记录。

---

## 2026-10-29

### SpacemiT 全面更新

**背景**：SpacemiT 知识库自 2026-08 重构后未更新，内容停留在占位符阶段，与实际文档不符。

**本次更新内容**：

1. **新增原子词条**
   - 创建 `spacemit/atoms/X100-Core.md` — K3 的高性能 CPU 核心
   - 创建 `spacemit/atoms/A100-AI-Core.md` — K3 的专用 AI 加速核心
   - 修复跨厂商术语对照表中 8 处断链

2. **扩展空白文件（11 个从占位符升级为完整文档）**
   - `software/Bianbu.md` — 从"疑为..."占位升级为完整定义（SpacemiT 官方 Ubuntu 发行版）
   - `software/Buildroot.md`, `OpenWrt.md`, `OpenHarmony.md`, `ROS2.md`, `SDK.md`
   - `ai/Ollama.md`, `localAI.md`, `LangChain.md`, `LlamaIndex.md`
   - `tools/Flashing-Tool.md`, `JTAG-Debugging.md`, `Trace-Tool.md`

3. **修正事实错误**
   - `hardware/vocabulary.md`：K3 "万兆以太网" → "双千兆以太网"（基于 K3.md 实际规格）
   - `atoms/HBM.md`：说明 SpacemiT 不使用 HBM，仅作对比参考
   - `atoms/Matrix-Extension.md`：区分 K1 指令扩展（约 2 TOPS）与 K3 A100 硬件核心（约 60 TOPS）

4. **Frontmatter 规范化**
   - 所有 atoms 更新 `date:` → `created:` + `updated:`
   - 移除不符合内容的 `status: approved`，统一为 `status: needs_review`
   - 添加中文译名到所有扩展后的文件

5. **新增学习资源**
   - 创建 `spacemit/LEARNING_GUIDE.md` — 专为技术写作新手设计的分阶段学习指南
   - 创建 `spacemit/学习记录.md` — 配套学习日志模板
   - 在 `spacemit/HOME.md` 首位推荐学习指南

6. **更新索引**
   - `spacemit/atoms/README.md` — 反映新增的 X100 与 A100
   - `spacemit/HOME.md` — 新增"更新记录"章节，完整导航
   - `HOME.md`（根目录）— 在知识域表格首位添加 SpacemiT

**文件统计**：
- 新增：2 个原子词条 + 2 个学习文件
- 扩展：11 个从占位符到完整文档
- 修正：8 个事实/格式错误
- 规范化：全部 8 个 atoms frontmatter

**未完成（待后续补充）**：
- `cloud/notes.md`, `SpacemiT-Cloud.md` 仍为占位符（官方云平台资料有限）
- `hardware/notes.md`, `software/notes.md`, `ai/notes.md`, `tools/notes.md` 中"待补充"内容

---

## 2026-08-13

- Backfill frontmatter on all 26 existing atoms: added `status: approved`, `created`, `updated`, `superseded_by` to nvidia (13), riscv (7), spacemit (6) atoms
- Added `aliases` to spacemit atoms (previously missing)

## 2026-08-12

- 新建 [[AGENT.md]]：定义 AI 协作规范（frontmatter 标准、status 生命周期、四大工作流、协作红线）
- 更新三份 `_templates/term.md`（根目录、nvidia、spacemit）：新增 `status`、`created`、`updated`、`superseded_by` 字段
- 新建本文件 `log.md` 作为全局变更日志
- 更新 [[ARCHITECTURE.md]]：新增 frontmatter 规范、内容保留规则（never-delete 原则）、AI 协作规范章节
