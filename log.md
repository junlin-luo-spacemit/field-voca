---
tags: [meta, log, changelog]
---

# 知识库变更日志

> 记录跨知识库的重大结构性变更。日期倒序排列，仅追加，不覆盖历史记录。

---

## 2026-08-13

- Backfill frontmatter on all 26 existing atoms: added `status: approved`, `created`, `updated`, `superseded_by` to nvidia (13), riscv (7), spacemit (6) atoms
- Added `aliases` to spacemit atoms (previously missing)

## 2026-08-12

- 新建 [[AGENT.md]]：定义 AI 协作规范（frontmatter 标准、status 生命周期、四大工作流、协作红线）
- 更新三份 `_templates/term.md`（根目录、nvidia、spacemit）：新增 `status`、`created`、`updated`、`superseded_by` 字段
- 新建本文件 `log.md` 作为全局变更日志
- 更新 [[ARCHITECTURE.md]]：新增 frontmatter 规范、内容保留规则（never-delete 原则）、AI 协作规范章节
