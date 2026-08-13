---
tags: [meta, architecture, guidelines]
date: 2026-08-08
---

# 知识库架构规范

> 本文档定义 field_voca 项目中所有知识库的架构标准，确保结构一致性与可维护性。

---

## 📐 两种架构模板

### 模板 A：技术文档知识库（Vendor Documentation）

**用途**：整理厂商产品文档、API 参考、技术手册

**适用场景**：
- 芯片厂商产品文档（NVIDIA, SpacemiT, AMD, Intel, Qualcomm, ARM）
- 软件框架文档（CUDA, TensorRT, PyTorch）
- 明确的技术栈分类（硬件/软件/AI/工具）

**标准结构**：
```
{vendor}/
├── HOME.md                   # 知识库主页，链接到各领域
├── README.md                 # 项目说明
├── PROGRESS.md               # 进度追踪
├── _templates/               # 词条模板
│   └── term.md
│
├── atoms/                    # 5-10 个核心原子词条
│   ├── README.md             # atoms 选择标准说明
│   └── {concept}.md          # 跨领域、基础性、厂商特色的核心概念
│
├── {domain1}/                # 技术领域（如 hardware, software, ai）
│   ├── vocabulary.md         # 📄 术语速查表（2-5行/术语，广度）
│   ├── notes.md              # 📄 学习笔记、规格表（深度）
│   └── {detail}.md           # 📄 详细技术页面（可选）
│
├── {domain2}/
│   ├── vocabulary.md
│   ├── notes.md
│   └── ...
│
└── glossary/
    └── master_glossary.md    # 全局术语表（所有领域合集）
```

**关键原则**：
1. **atoms/ 定位**：仅存放跨领域引用的核心概念（5-10 个）
2. **vocabulary.md**：速查表，每个术语 2-5 行，侧重广度
3. **notes.md**：学习笔记、规格表、最佳实践，侧重深度
4. **领域划分**：按技术栈（hardware/software/ai）或产品线分类
5. **两层结构**：避免三级嵌套，保持 `domain/file.md` 的扁平化

**atoms 选择标准**：
- ✅ 在多个领域（≥2 个）中被频繁引用
- ✅ 是理解其他概念的前置知识
- ✅ 厂商独有或重点强调的技术
- ✅ 需要独立页面进行完整阐述（>50 行）
- ❌ 只在单一领域使用的术语 → 放入该领域的 vocabulary.md
- ❌ 第三方工具/框架（非核心技术）→ 放入对应领域文件夹

**示例**：
- `nvidia/`：atoms 包含 SM, warp, Tensor Core, HBM（GPU 架构核心）
- `spacemit/`：atoms 包含 SoC, Matrix Extension, X60 Core（芯片核心）

**学习路径配套**（推荐但可选）：
- 可添加 `LEARNING_GUIDE.md` 提供从零开始的学习顺序
- 或添加 `learning_path/` 文件夹，包含分步教程
- 学习路径通过 `[[wikilink]]` 链接到 atoms 和 vocabulary，不重复内容

---

### 模板 B：学习路径知识库（Learning Path）

**用途**：系统学习技术规范、指令集架构、协议标准

**适用场景**：
- ISA 规范学习（RISC-V, ARM Architecture, x86-64）
- 协议标准学习（PCIe, USB, Ethernet）
- 按章节/难度递进的教学内容

**学习路径配套**（推荐但可选）：
- 可添加 `LEARNING_GUIDE.md` 或 `README.md` 提供整体学习路线图
- 或在 `HOME.md` 中包含"学习路径"章节，指引阅读顺序
- 通过 `[[wikilink]]` 链接组织学习顺序，无需重复内容

**标准结构**：
```
{topic}/
├── HOME.md                   # 学习路线主页
├── PROGRESS.md               # 学习进度追踪
├── 概念地图.md                # 知识体系概览
│
├── 01-{chapter1}/            # 按学习顺序编号
│   ├── vocabulary.md         # 本章术语
│   ├── notes.md              # 本章学习笔记
│   └── atoms/                # 本章核心概念（独立页面）
│       └── {concept}.md
│
├── 02-{chapter2}/
│   ├── vocabulary.md
│   ├── notes.md
│   └── atoms/
│
├── learning_path/            # 学习导航（可选）
│   ├── 00-学习导航.md
│   ├── 01-入门.md
│   └── ...
│
└── glossary/
    └── master_glossary.md    # 全章节术语合集
```

**关键原则**：
1. **章节编号**：用 `01-`, `02-` 等前缀标示学习顺序
2. **atoms 分散**：每章有独立 atoms/ 子文件夹（符合教学逻辑）
3. **避免三级嵌套**：如 `05-调试与工具链/` 下有多个主题，应拆分为 `05-debug/`, `05-platform/`, `05-software/`
4. **概念地图**：提供整体知识框架的可视化
5. **学习路径**：可选的循序渐进指南

**示例**：
- `riscv/`：01-isa基础 → 02-特权架构 → 03-标准扩展 → 04-微架构

---

## 🎯 架构选择决策树

```
是厂商产品文档 or 技术框架文档？
├─ 是 → 模板 A（技术文档知识库）
│   ├─ 示例：nvidia/, spacemit/, amd/, pytorch/
│   └─ 可选：添加 LEARNING_GUIDE.md 提供学习路径
│
└─ 否，是学习/规范文档？
    ├─ 是 → 模板 B（学习路径知识库）
    │   ├─ 示例：riscv/, arm-architecture/, pcie-spec/
    │   └─ 可选：添加学习导航文件指引阅读顺序
    │
    └─ 通用原则：两种模板都应配套学习路径
        - 模板 A：通过 LEARNING_GUIDE.md 引导从零学习
        - 模板 B：通过章节编号和导航文件引导学习
        - 都使用 [[wikilink]] 链接，不重复内容
```

---

## 📂 当前知识库分类

| 知识库 | 架构模板 | 学习路径 | 状态 | 备注 |
|--------|---------|---------|------|------|
| `nvidia/` | 模板 A | ✅ LEARNING_GUIDE.md | ✅ 标准 | 技术文档知识库的参考实现 |
| `riscv/` | 模板 B | ✅ learning_path/ | ✅ 优化完成 | 已扁平化三级嵌套 |
| `spacemit/` | 模板 A | ⬜ 待添加 | ✅ 重构完成 | 已对齐 NVIDIA 结构 |

---

## 📝 命名规范

### 文件命名
- **HOME.md**：知识库主页，大写
- **README.md**：项目说明，大写
- **PROGRESS.md**：进度追踪，大写
- **vocabulary.md**：术语表，小写
- **notes.md**：笔记，小写
- **{concept}.md**：具体词条，使用 kebab-case（如 `Matrix-Extension.md`）

### 领域文件夹命名（模板 A）
- 小写，使用短横线：`hardware/`, `ai-platform/`, `dev-tools/`
- 避免缩写：`software/` 优于 `sw/`
- 优先使用英文：`hardware/` 优于 `硬件/`

### 章节文件夹命名（模板 B）
- 编号 + 中文描述：`01-isa基础/`, `02-特权架构/`
- 编号用两位数：`01-` 而非 `1-`

---

## 🔗 内部链接规范

### 链接到 atoms
- 模板 A：`[[atoms/SoC]]` 或 `[[../atoms/SoC]]`（从领域文件夹）
- 模板 B：`[[01-isa基础/atoms/RV32I]]`

### 链接到 vocabulary
- 模板 A：`[[hardware/vocabulary]]` 或 `[[hardware/vocabulary#HBM]]`（锚点）
- 模板 B：`[[01-isa基础/vocabulary]]`

### 跨知识库链接
- 使用相对路径：`[[../nvidia/atoms/HBM]]`
- 或使用绝对路径：`[[nvidia/atoms/HBM]]`（如果 Obsidian 配置支持）

---

## ✅ 质量标准

每个知识库应包含：
1. ✅ **HOME.md**：清晰的导航和核心词条列表
2. ✅ **PROGRESS.md**：当前进度与待办事项
3. ✅ **vocabulary.md**：每个领域/章节都有术语速查表
4. ✅ **atoms/**：核心概念有独立深度页面
5. ✅ **学习路径**：LEARNING_GUIDE.md（模板 A）或章节导航（模板 B），从零到熟练的阅读顺序
6. ✅ **glossary/master_glossary.md**：全局术语表（可选）

每个 atom 词条应包含：
1. ✅ **定义**：清晰的概念解释
2. ✅ **官方原文**：引用官方文档（如有）
3. ✅ **关键参数/示例**：具体数据或代码
4. ✅ **相关词条**：双向链接到其他概念
5. ✅ **来源链接**：标注官方文档 URL

每个词条 frontmatter 应包含：
1. ✅ **status**：`draft` / `needs_review` / `approved` / `deprecated`
2. ✅ **aliases**：双语别名列表，支持中英文双向检索
3. ✅ **created / updated**：日期追踪
4. ✅ **superseded_by**：当 `status: deprecated` 时必填，指向替代文档

**内容保留规则**：
- ❌ 禁止物理删除文件，无论内容多旧
- ✅ 过期内容标记为 `status: deprecated` + 填写 `superseded_by`
- ✅ 不再在 `HOME.md` 主索引中展示 deprecated 词条，但文件保留

---

## 🔄 架构演进

**版本历史**：
| 日期 | 版本 | 变更 |
|------|------|------|
| 2026-08-12 | 1.2 | 新增 frontmatter 规范（status 生命周期、aliases、superseded_by）；新增内容保留规则；新增 AGENT.md AI 协作规范；新增 log.md 全局变更日志 |
| 2026-08-08 | 1.1 | 明确两种模板都应配套学习路径（模板 A 用 LEARNING_GUIDE.md，模板 B 用章节导航），更新决策树和示例 |
| 2026-08-08 | 1.0 | 初始版本，定义模板 A 和 B；同日完成 SpacemiT 重构对齐模板 A（详见 `spacemit/PROGRESS.md`）与 RISC-V 扁平化（`05-调试与工具链/` 拆分为顶层 `05-debug/05-platform/05-software/`） |

**未来扩展**：
- 模板 C（待定）：用于对比性研究（如"GPU 架构对比"）
- 自动化工具：生成 master_glossary.md 的脚本
- 质量检查：词条完整性自动验证

---

## 📚 参考资料

- **Obsidian 最佳实践**：[Obsidian Help](https://help.obsidian.md/)
- **Zettelkasten 方法**：原子化笔记与双向链接
- **Documentation System**：Divio 四象限（教程/指南/参考/解释）

---

**最后更新**：2026-08-12  
**维护者**：field_voca 项目组

---

## 🤖 AI 协作规范

> AI Agent 在此知识库中工作时，必须遵守 [[AGENT]] 中定义的协作规范。

核心原则摘要：
- AI 创建/修改内容默认设为 `status: needs_review`，人工审核后才能设为 `approved`
- 禁止创建同一概念的平行双语文件，以中文为主体 + `aliases` 实现双语检索
- 禁止物理删除文件，使用 `deprecated` + `superseded_by` 替代
- 重大变更必须记录到 [[log]]
