# beginner-study-lessons

**Display Name**: SpacemiT 技术学习课程与测验交付指南

**Invocation Names**:
- beginner-study-lessons
- study-lessons
- spacemit-study

**Author**: SpacemiT Documentation Team

**When to Use**:
- Creating or reviewing SpacemiT beginner-friendly study lessons (M1–M15 series)
- Creating or reviewing companion quizzes with 15 questions + 14 self-eval items
- Ensuring technical accuracy, bilingual terminology, source traceability, and entry-level accessibility
- Maintaining continuity across sessions with existing M1–M4 lessons and quiz template

**What It Does**:
Guides the creation and review of SpacemiT study materials following established conventions in `spacemit/study/`:
- **Lesson structure**: objective → sections with examples/analogies/diagrams → self-check → source table
- **Quiz structure**: 3 groups (10 foundation + 5 trap-recognition + 14 self-eval), embedded `rv` excerpts, copy-to-clipboard diagnostic
- **Style enforcement**: English pairs for every term, source citations to chapter level, conditions for every number, GLOSSARY links for complex terms, visual aids (flow diagrams, tables, ASCII/mermaid/SVG) for junior audience
- **Validation**: anchor resolution, table integrity, term coverage, quiz DATA schema (15 questions / 14 SA items / `sai` 0–13)

**What It Does NOT Do**:
- Write lessons for products/features outside K1/K3/V100/P-series or docs repos not listed in source tables
- Generate quizzes without validating against existing course content
- Translate raw chip manuals (use `SpacemiT Brief Translator` skill for that)
- Cross-vendor terminology research (use `文档专员` skill for field_voca cross-reference)

---

## Context

This skill operates on the **SpacemiT beginner study series** in `spacemit/study/`, specifically:
- **M1–M4** are complete (product overview, chip architecture, memory/storage, boot process)
- **M5–M15** are planned in ROADMAP.md
- Each module = 3 lessons + 1 quiz (HTML, 15Q + 14SA items)
- Quiz template at `quiz/_template.html` supports 8 parameterized slots
- GLOSSARY.md contains 28 entries (14 original + 14 added 2026-11-24)
- Target audience: technical writers learning SpacemiT chips to document them

**Critical principle**: This is a **cumulative, coherent curriculum**. Every new module must maintain consistency with all prior modules. After writing M4, you must review M1–M3 for terminology alignment, cross-references, and depth calibration. After writing M5, review M1–M4. The entire series (M1–M15 + GLOSSARY + all quizzes) must read as one unified learning path, not a collection of independent documents.

## Key Files

| File | Purpose |
|---|---|
| `STYLE.md` | Lesson writing rules: source tables, English pairs, number conditions, GLOSSARY links |
| `HANDOFF.md` | Progress log, key decisions, M1 test results, unverified items |
| `ROADMAP.md` | M1–M15 module sequence with library assignments and rationale |
| `GLOSSARY.md` | 28 technical terms with what/why/K3-specifics/related-terms structure |
| `quiz/_template.html` | Quiz skeleton: scoring, lock-on-answer, embedded excerpts, 3-tier priority revisit list, copy diagnostic |
| `quiz/README.md` | Placeholder map, field semantics, `sai` alignment, pre-delivery checklist |

## Lesson Writing Workflow

### 1. Plan the Module (3 Lessons)
- Read ROADMAP.md for this module's scope and library list
- Identify 3 self-contained topics that together cover the module theme
- Sketch the progression: concept → application → integration
- Each lesson should stand alone but build toward the module quiz

### 2. Write One Lesson

**Front Matter**:
```yaml
---
tags: [study, m<N>, <topic>, spacemit]
domain: <chip|edge_robotics|...>
aliases: [中文标题, English Title]
status: needs_review
created: YYYY-MM-DD
updated: YYYY-MM-DD
---
```

**Structure**:
1. **Objective block** (box quote): what you'll understand after this lesson, with 📌 note if it's K3-specific but concepts apply to all SoCs
2. **Numbered sections** (##): 3–5 sections, each with concrete examples or analogies
3. **Quick self-check** (unnumbered list of 3 questions)
4. **Source table** (## 参考来源): library · file path · chapter for every fact

**Style Enforcement** (from STYLE.md):
- **English pairs**: Every technical term's first appearance → `中文（English, Full Expansion）`
  - Example: `直接存储器访问（Direct Memory Access，DMA）`
  - Chip models, registers, file paths stay in original language
- **Number conditions**: Every performance/capacity/rate number includes conditions
  - ❌ "AI 算力 60 TOPS"
  - ✅ "AI 算力 60 TOPS（@INT4 sparse）"
- **GLOSSARY links**: Complex terms link to `[术语](GLOSSARY.md#术语)`
  - Use for terms needing depth (带宽, 簇, AFBC, Boot ROM, OpenSBI, etc.)
  - Keep lesson text concise; GLOSSARY provides deep dive
- **Source traceability**: Only cite files you've read; include version/date for numbers
- **Visual aids for clarity**: Target audience is junior technical staff learning unfamiliar domain
  - Use **flow diagrams** for sequences (e.g., `Boot ROM → FSBL → OpenSBI → U-Boot → Linux`)
  - Use **comparison tables** for similar concepts (e.g., eMMC vs UFS, 根密钥 vs 发版密钥)
  - Use **ASCII art or mermaid diagrams** for structures (memory hierarchy,启动链路, 验证流程)
  - Use **concrete examples with numbers** (e.g., "0.1s 检测介质 → 0.5s 初始化内存 → 1–2s 加载内核")
  - Use **analogies** for abstract concepts (e.g., "eFuse 像给门上焊死一把锁", "OpenSBI 是翻译官")
  - **Prefer SVG diagrams** for architecture/flow when ASCII/mermaid insufficient; store in `spacemit/study/assets/` and reference as `![alt](./assets/m4-boot-flow.svg)`

**Length Target**: 70–105 lines (M1-M4 average). If a lesson exceeds 125 lines, consider splitting.

**Verification**:
- Run `check_glossary.py` to verify all GLOSSARY anchors resolve
- Ensure every number has a condition in parentheses
- Confirm source table rows exist in the specified repos

### 3. Cross-Check with Existing Modules

**After completing a new module**, systematically review **all prior modules** to ensure the entire series remains consistent:

1. **Terminology alignment**:
   - Search for the same concept across all lessons (e.g., "簇", "带宽", "启动")
   - Ensure every occurrence uses the same Chinese term and links to GLOSSARY consistently
   - If M4 introduced a new way to explain "安全启动", check if M2 or M3 mentioned it and update for consistency

2. **Cross-reference updates**:
   - If M4 adds detail that M3 previewed, add forward links from M3 → M4
   - Example: M3-03 originally covered boot flow; after M4 was written, M3-03 was rewritten to defer to M4 with explicit links

3. **Depth calibration**:
   - If M5 targets entry-level but uses a concept explained in M2, verify M2's explanation is accessible
   - If M4 raised the bar (e.g., added端到端耗时示例), check if M1-M3 need similar concreteness

4. **GLOSSARY completeness**:
   - Every term M5 links to must already exist in GLOSSARY
   - If M5 reveals that "FSBL" needs deeper explanation than M4 provided, enhance the GLOSSARY entry and add a link from M4

5. **Quiz alignment**:
   - If M5 quiz references a concept from M2, verify M2's `rv` excerpt still matches the lesson text
   - If M4 rewrites changed wording, update M4 quiz `rv` fields to stay verbatim

**Example**: After writing M4 (boot process), the team discovered M3-03 duplicated boot content. M3-03 was rewritten to focus only on on-chip memory, with all boot details deferred to M4. This maintained M3's utility while eliminating overlap.

**When to do this**:
- **Mandatory**: After completing any module M(N), review M(1) through M(N-1)
- **Incremental**: Fix only what the new module affects; don't rewrite unrelated sections
- **Document**: Update HANDOFF.md with what was changed and why

**Verification**:
- Run `grep -r "<term>" *.md` to find all uses of key terms
- Check GLOSSARY anchor usage: `python check_glossary.py`
- Read through module sequence (M1 → M2 → M3 → new module) as if you're the learner

## Quiz Creation Workflow

### 1. Use the Template

Copy `quiz/_template.html` → `quiz/m<N>-recap.html` and replace 8 placeholders (see `quiz/README.md`):

| Placeholder | Replace With | Example |
|---|---|---|
| `{{MODULE_NUMBER}}` | `1` | for M1 |
| `{{MODULE_TITLE}}` | `产品线全景` | module theme |
| `{{MODULE_TITLE_EN}}` | `Product Overview` | English title |
| `{{NUM_FOUNDATION}}` | `10` | foundation questions |
| `{{NUM_TRAP}}` | `5` | trap-recognition questions |
| `{{NUM_SA}}` | `14` | self-eval items |
| `localStorage_key` | `spacemit.m1.history` | unique key |
| `DATA` array | quiz content | see below |

### 2. Write the DATA Array

**Structure**: 3 groups (foundation, traps, self-eval)

```javascript
const DATA = [
  { type: "基础与应用", qs: [ /* 10 questions */ ] },
  { type: "认陷阱", qs: [ /* 5 questions */ ] },
  { type: "自评", items: [ /* 14 self-eval */ ] }
];
```

**Question Schema**:
```javascript
{
  q: "问题文本",
  sai: 3,  // links to self-eval item index (0-13)
  opts: ["选项 A", "选项 B", "选项 C", "选项 D"],
  a: 1,    // correct option index (0-3)
  e: "解析：为什么 B 对，A/C/D 各是什么",
  wrong: {
    0: { t: "A 错在哪里", why: "为什么会选 A" },
    2: "C 错在哪里",
    3: "D 错在哪里"
  },
  src: "docs-chip · key_stone/k3/k3_docs/k3_ds.md · 2.1.2 A100",
  rv: "从课文逐字摘录的 2-3 句原文，正好解决此题"
}
```

**Self-Eval Schema**:
```javascript
{
  k: "知识点名称（不超过 20 字）",
  v: "可以这样验证：具体的自测方法",
  sai: 0  // unique 0-13, one per item
}
```

**Rules** (from STYLE.md § 第二部分):
- **Options must be plausible**: length within 20%, consistent structure (all noun phrases or all sentences), no absurd choices ("管理照片", "替代内核")
- **Every wrong option has rationale**: in `wrong` object, explain what it actually is
- **Embedded excerpts**: `rv` field contains 2–3 sentences from lesson text, verbatim
- **Source to chapter level**: not just file name
- **sai coverage**: 0–13 all used exactly once across 15 questions; self-eval `sai` matches

### 3. Validate Before Delivery

Run `validate_m<N>_quiz.py` (adapt from `validate_m4_quiz.py` in session files):

```python
# Check:
# - 3 groups, 15 questions, 14 SA items
# - sai distinct 0-13, no gaps, no out-of-range
# - every question has 4 options, `a` in range, `wrong` keys valid
# - no missing `rv`, no missing `src`
```

**Browser test** (all 5 tests must pass):
1. Initial render: 15 questions, 14 SA rows, answered 0/15
2. Answer all correctly → 15/15, rate 100%, no revisit items
3. Answer 2 wrong + mark 3+ SA "unsure" → revisit list renders all items (no TypeError)
4. Copy diagnostic → clipboard shows session summary
5. Reset → answered 0/15, all locked options unlock, SA buttons reset, revisit clears

**Common defects caught**:
- `rv` field absent or misquoted from lesson
- `sai` gaps or duplicates
- `wrong` keys not matching the incorrect option indices
- Revisit list crash when 2+ SA marked unsure (fixed 2026-11-24 with `it.i !== undefined &&` guard in `_template.html`)

## GLOSSARY Maintenance

When adding a term:

1. **Check if it exists**: search GLOSSARY.md for the term first
2. **Structure**: follow existing entries (带宽, Boot ROM, OpenSBI as examples)
   ```markdown
   ## 术语

   **中文**（English, Full Name）简要定义。

   ### 核心概念 / 为什么需要它？
   ...

   ### K3 的 <术语>
   K3 specific details

   ### 类比理解（optional for abstract terms）
   ...

   **相关概念**：term1、term2

   ---
   ```
3. **Add to source table** at end: `| 术语 | docs-* · path | chapter |`
4. **Link from lessons**: `[术语](GLOSSARY.md#术语)` where it's first explained in depth
5. **Verify anchor**: run `check_glossary.py` to confirm `#术语` resolves

## Verification Checklist (Before Commit)

### Lessons
- [ ] Every technical term has English pair on first use
- [ ] Every number includes conditions (e.g., @INT8, HS400, 6400 MT/s)
- [ ] Complex terms link to GLOSSARY.md
- [ ] **Visual aids present for multi-step flows and comparisons**: flow diagrams for sequences, tables for similar concepts, ASCII/mermaid/SVG for structures
- [ ] **Analogies and concrete examples** help junior audience understand abstract concepts
- [ ] Source table lists actual file paths and chapters
- [ ] No broken relative links (`grep -r "\.md\)" *.md` then verify each)
- [ ] Front matter dates updated

### Quiz
- [ ] 3 groups: 10 + 5 + 14
- [ ] `sai` coverage: 0–13 all used exactly once in questions, matched in SA items
- [ ] Every question has 4 options, `a` valid, `wrong` keys match incorrect indices
- [ ] `rv` field present and verbatim from lesson
- [ ] `src` cites chapter, not just file
- [ ] Browser test passes all 5 scenarios (render, scoring, multi-SA revisit, copy, reset)
- [ ] `localStorage` key unique (`spacemit.m<N>.history`)

### Cross-Module
- [ ] Run `check_glossary.py`: 0 unresolved anchors, 0 unused entries
- [ ] Run link validator: 0 broken `./` links across all lessons
- [ ] Table integrity: pipe count consistent in every table block
- [ ] **Review M1–M(N-1) for consistency**: terminology alignment, cross-references added where new module builds on prior content, depth calibration (if new module raises clarity bar, consider updating older modules)
- [ ] Update HANDOFF.md progress table and document any cross-module changes
- [ ] Update README.md with new module links

## Common Pitfalls

1. **Reusing old `DATA[2]` hardcoding**: Use `DATA.find(g => g.type === "自评")` (all M1–M4 quizzes already fixed)
2. **Forgetting `it.i !== undefined` guard**: Revisit list crashes with 2+ SA marked unsure (fixed in template + all quizzes 2026-11-24)
3. **`rv` not verbatim from lesson**: Copy-paste exact sentences, don't paraphrase
4. **`sai` gaps**: Python validator catches this; also manually check 0–13 all present
5. **Options too easy to eliminate**: Avoid absurd choices; every wrong option should map to a real misconception
6. **Missing English pairs for new abbreviations**: DFS, ADB, TEE, PCIe, etc. must expand on first use
7. **Text-only explanations for complex flows**: Junior audience needs visual aids; don't skip diagrams for multi-step processes (boot sequence, memory hierarchy, verification chain)
8. **Assuming domain familiarity**: Explain "why this matters" before diving into specs; use concrete examples and analogies

## Example Session Flow

**User asks**: "Create M5 lessons on Buildroot vs Bianbu"

**You do**:
1. Read ROADMAP.md M5 row: theme, library assignments, rationale
2. Read `docs-buildroot`, `docs-bianbu`, `docs-network` relevant chapters
3. Sketch 3 lesson topics (e.g., system philosophy, package management, use-case mapping)
4. Write m5-01 following STYLE.md: objective, 3–5 sections with examples, self-check, source table
5. Add any new terms to GLOSSARY.md if not already present
6. Write m5-02, m5-03 similarly
7. Create `quiz/m5-recap.html` from template: 15Q (10+5) + 14SA, link each Q to lesson via `rv`
8. Run `validate_m5_quiz.py` + browser test
9. **Review M1–M4 for consistency**: search for overlapping concepts (e.g., if M5 mentions "内存" or "eMMC", verify M2/M3 terminology matches), add cross-references where M5 builds on prior modules, check if any M1–M4 GLOSSARY links need updates
10. Update README.md, HANDOFF.md, ROADMAP.md progress
11. Commit with descriptive message citing STYLE adherence and cross-module review

**After M5 is merged**, the entire M1–M5 sequence should read as a unified learning path with consistent terminology, aligned depth, and bidirectional cross-references.

## References

- [STYLE.md](../../spacemit/study/STYLE.md) — writing rules
- [HANDOFF.md](../../spacemit/study/HANDOFF.md) — progress log and key decisions
- [ROADMAP.md](../../spacemit/study/ROADMAP.md) — M1–M15 sequence
- [GLOSSARY.md](../../spacemit/study/GLOSSARY.md) — 28 technical terms
- [quiz/_template.html](../../spacemit/study/quiz/_template.html) — quiz skeleton
- [quiz/README.md](../../spacemit/study/quiz/README.md) — template placeholder map

---

**Version**: 2026-11-25  
**Maintained by**: SpacemiT Documentation Team

---

## Changelog

**2026-11-25**: Added cross-module consistency review mandate (after M(N), review M1–M(N-1) for terminology/cross-refs/depth); added visual-aids-for-junior-audience guidance (flow diagrams, tables, ASCII/mermaid/SVG, concrete examples, analogies); emphasized M1–M15 as unified learning path, not independent documents.
