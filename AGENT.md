---
tags: [meta, agent, collaboration, guidelines]
status: approved
created: 2026-08-12
updated: 2026-08-12
---

# AI Agent Collaboration Contract

> Defines what AI agents may and must do when working in this knowledge base. All AI assistants operating in `field_voca/` must follow these rules.

---

## 1. Knowledge Base Identity

**Position**: `field_voca` is a cross-vendor technical vocabulary knowledge base for chip documentation specialists.

**Core Goals**:
1. **Dual Accessibility**: Human engineers browse via Obsidian graph + index pages; AI agents retrieve via semantic search + wikilinks.
2. **Knowledge Accumulation**: Merge new information into existing nodes rather than creating scattered duplicates. The knowledge base should grow denser, not wider.
3. **Human-AI Collaboration**: AI handles tedious work (formatting, linking, organizing); humans make architectural decisions and final approvals.

---

## 2. Knowledge Tiers & Directory Rules

The knowledge base uses **two architectural templates** (Template A: vendor docs, Template B: learning path). See [[ARCHITECTURE]] for full specification.

### Common Rules Across All Vendors

**atoms/** — Core atomic concepts only:
- Cross-domain concepts referenced by ≥2 domains
- Foundational knowledge required to understand other concepts
- Vendor-specific flagship technologies
- Requires independent deep-dive page (>50 lines)
- ❌ Single-domain terms → put in `{domain}/vocabulary.md`
- ❌ Third-party tools → put in corresponding domain folder

**vocabulary.md** — Quick reference tables:
- 2-5 lines per term, breadth over depth
- Used for rapid lookup, not learning

**notes.md** — Learning notes & specifications:
- Deep technical content, best practices, spec tables
- Where you synthesize understanding from multiple sources

---

## 3. Frontmatter Standards (Mandatory)

Every markdown file must have YAML frontmatter with these fields:

```yaml
---
tags: [atom/term, vendor, domain]
domain: <see domain taxonomy below>
aliases: [中文别名, EnglishAlias, short-form]
status: draft | needs_review | approved | deprecated
created: YYYY-MM-DD
updated: YYYY-MM-DD
source: <official doc URL>
superseded_by: "[[new-file]]"  # only when status=deprecated
---
```

### Status Lifecycle

**Status flow rules**:
- **`draft`**: Incomplete content, work in progress.
- **`needs_review`**: **Default state when AI creates or updates content**. Awaiting human verification.
- **`approved`**: Human-verified content, ready for production use. **AI cannot set status to `approved`**.
- **`deprecated`**: Content superseded or obsoleted. **Never delete deprecated files**—mark deprecated and add `superseded_by` field pointing to replacement.

**Critical Rules**:
1. AI must set `status: needs_review` on all created/updated content.
2. Only humans can change status to `approved`.
3. When deprecating, `superseded_by` field is mandatory.
4. Physical file deletion is prohibited—deprecate instead.

### Domain Taxonomy

Universal domains across all vendors (choose the best fit):

**Hardware & Architecture**:
- `chip_architecture` — Core design, ISA, microarchitecture
- `memory_system` — Cache, memory hierarchy, bandwidth
- `interconnect` — NVLink, PCIe, on-chip networks
- `power_thermal` — Power delivery, thermal design, TDP

**Software & Tools**:
- `compiler_toolchain` — GCC, LLVM, compilation
- `driver_kernel` — Device drivers, kernel modules, BSP
- `sdk_framework` — CUDA, TensorRT, ROCm, AI frameworks
- `debug_profiling` — Debugging tools, profilers, analyzers

**System & Platform**:
- `os_distribution` — Linux distros, OS support, system software
- `virtualization` — VM, containers, resource partitioning
- `security` — Secure boot, encryption, TEE

**Application & Workload**:
- `ai_ml` — AI/ML inference, training, model optimization
- `hpc` — Scientific computing, HPC workloads
- `edge_robotics` — Edge AI, robotics, embedded systems

**Cross-cutting**:
- `isa` — Instruction set architecture (RISC-V specific)
- `standards_spec` — Industry standards, protocols, specifications

---

## 4. Wikilink & Reference Rules

### Link Style

1. **No URL encoding**: Use raw Chinese characters in wikilinks.
   - ❌ `[[Knowledge_Atoms/K3%E8%8A%AF%E7%89%87...]]`
   - ✅ `[[atoms/K3芯片热设计]]`

2. **Display text for inline links**: Use `[[path/file|display text]]` for natural reading.
   - Example: "See [[atoms/HBM|high-bandwidth memory specs]] for details."

3. **Cross-vendor links**: Use relative paths.
   - From nvidia domain: `[[../riscv/01-isa基础/atoms/hart]]`
   - From root: `[[nvidia/atoms/SM]]`

4. **Bilingual alias strategy**: Use YAML `aliases` field to enable cross-language linking.
   - Chinese primary content + English aliases in frontmatter
   - Allows both `[[SM]]` and `[[Streaming Multiprocessor]]` to work

### Reference Topology (Template A only)

If using Template A (vendor docs):
- `atoms/` can reference other `atoms/` or domain `vocabulary.md`
- Domain files can reference `atoms/` or other domain files
- Keep references bidirectional where it aids understanding

---

## 5. AI Agent Core Workflows

### Workflow 1: Ingest (Knowledge Intake)

**Trigger**: New official documentation added or discovered.

**Agent Actions**:
1. Extract core parameters, specs, and facts.
2. Check if existing atom/vocabulary entry covers this concept:
   - If yes → update existing entry, increment `updated` date
   - If no → create new entry with `status: needs_review`
3. Establish wikilinks to related existing concepts.
4. Add entry to `HOME.md` index if it's a new atom.
5. Log the change in `log.md`.

**Bilingual handling**: If both Chinese and English sources exist, cross-check for consistency. If conflict detected, flag for human review—do not choose unilaterally.

---

### Workflow 2: Consolidate (Conversation Distillation)

**Trigger**: After a deep technical discussion or troubleshooting session with a human.

**Agent Actions**:
1. Extract reusable technical insights from conversation.
2. Create or update relevant atom/vocabulary entry.
3. Link to related concepts using wikilinks.
4. Set `status: needs_review`.
5. Log in `log.md`: "Distilled knowledge from session: [[topic]]"

---

### Workflow 3: Refactor (Structure Maintenance)

**Trigger**: Periodic or on-demand when quality issues accumulate.

**Agent Actions**:
1. **Broken link scan**: Find all broken wikilinks and report.
2. **Orphan detection**: Find atoms/terms with no incoming links and suggest integration.
3. **Duplicate detection**: Identify similar content that should be merged.
4. **Consistency check**: Verify all atoms have required frontmatter fields.
5. **Report to human**: Present findings, do not auto-fix structural issues.

**Do not**:
- Automatically delete files
- Automatically merge content without human approval
- Change `status: approved` entries without explicit instruction

---

### Workflow 4: Update Templates

**Trigger**: Human requests template or structure updates.

**Agent Actions**:
1. Update `_templates/term.md` in affected vendor folder(s).
2. **Do not** automatically backfill existing files with new fields—wait for human decision.
3. Document the template change in `log.md`.
4. Update `ARCHITECTURE.md` if the change affects structural standards.

---

## 6. Collaboration Red Lines

**Prohibited Actions** (absolute rules):

1. **No Fabrication**: Never invent technical facts, specs, or parameters. All content must be sourced from official docs or verified conversation.

2. **Preserve Conflicts**: When sources contradict (including Chinese/English doc mismatches), preserve both versions and flag with `status: needs_review`. Do not pick one arbitrarily.

3. **Never Delete Deprecated Content**: Mark as `deprecated` + add `superseded_by`, but physical deletion is forbidden. Historical references must remain intact.

4. **No Auto-Approval**: AI cannot set `status: approved`. All AI-created/modified content must be `needs_review` or `draft`.

5. **No Parallel Bilingual Files**: Never create separate Chinese and English files for the same concept (e.g., `HBM.md` + `HBM-zh.md`). Use one file with Chinese primary + English aliases.

6. **Major Changes Require Logging**: Any structural change (new atoms, path refactoring, template updates) must be logged in `log.md` and indexed in `HOME.md`.

---

## 7. Index & Log Hygiene

### HOME.md — Knowledge Graph Entry Point

Every vendor's `HOME.md` must maintain:
- ✅ Link to all core atoms (5-10 entries)
- ✅ Link to all domain areas with brief descriptions
- ✅ Learning resources and progress tracking
- ✅ Quick reference section (key numbers, hierarchies)

**Agent responsibility**: When creating new atoms, add them to `HOME.md` under appropriate section.

---

### log.md — Running Changelog

**What to log** (use append, never overwrite):
- New atom creation: "2026-08-12: Created [[atoms/Matrix-Extension]] covering SpacemiT vector ISA extensions"
- Major refactoring: "2026-08-12: Consolidated 3 thermal docs into [[atoms/thermal-design]], deprecated [[old-thermal-notes]]"
- Template updates: "2026-08-12: Added `status` and `superseded_by` fields to all vendor templates"
- Content updates: "2026-08-12: Updated [[atoms/HBM]] with HBM3e specs from new NVIDIA whitepaper"

**Format** (chronological, newest on top):
```markdown
## 2026-08-12
- Created [[atoms/X]] (status: needs_review)
- Updated [[atoms/Y]] with new specs (status: needs_review)
- Deprecated [[old-doc]] → superseded by [[new-doc]]

## 2026-08-11
...
```

---

## 8. Quality Standards

Every atom must contain:
1. ✅ **Definition**: Clear, concise concept explanation
2. ✅ **Official Source**: Quote from official docs (if available)
3. ✅ **Key Specs/Examples**: Concrete data or code samples
4. ✅ **Related Terms**: Wikilinks to connected concepts
5. ✅ **Source Link**: URL to official documentation

Every vocabulary entry must contain:
1. ✅ **English term** + **中文译名**
2. ✅ **1-sentence definition**
3. ✅ **Domain context** (where it's used)
4. ✅ **Optional**: Link to atom for deep dive

---

## 9. Human Escalation Triggers

**Immediately escalate to human** when:
- ❌ Two official sources conflict on specs/definitions
- ❌ Unsure whether to create new atom vs. update existing
- ❌ Major structural change needed (directory reorg, template overhaul)
- ❌ Content involves deprecated/removed features (human decides preservation strategy)
- ❌ Dealing with vendor-sensitive information (NDAs, pre-release specs)

**Present options, do not decide unilaterally** for:
- Merging multiple related concepts into one atom
- Splitting one concept into multiple atoms
- Changing domain categorization
- Approving AI-generated content for production use

---

## 10. Continuous Improvement

This `AGENT.md` is living documentation. When you discover a gap in the rules or an ambiguous situation:

1. Handle the immediate situation conservatively (default to `needs_review` + human escalation)
2. Suggest an amendment to this document in `log.md`
3. Wait for human to approve and update `AGENT.md`

**Version History**:
- 2026-08-12 (v1.0): Initial version, established core collaboration contract

---

**Maintainer**: field_voca knowledge base team  
**Last Updated**: 2026-08-12
