---
tags: [meta, verification, spacemit]
status: in_progress
created: 2026-10-29
updated: 2026-10-29
---

# SpacemiT GitHub Documentation Verification Checklist

## GitHub Organization: spacemit-com

**Target repositories**: All repos named `docs-*` under https://github.com/spacemit-com/

## Verification Tasks

### 1. Repository Discovery
- [ ] List all `docs-*` repositories
- [ ] Note last update dates for each repo
- [ ] Identify primary documentation repos (likely: docs-k1, docs-k3, docs-bianbu, docs-ai, docs-sdk)

### 2. Key Technical Specifications to Verify

#### Hardware (K1 Chip)
- [ ] X60 Core count and configuration (current: 8×X60, dual cluster)
- [ ] Cluster0 AI extension capabilities (current: ~2 TOPS)
- [ ] Memory type and specifications (current: LPDDR4x)
- [ ] Cache sizes (L1: 32KB I+D per core, L2: 512KB per cluster, TCM: 512KB)

#### Hardware (K3 Chip)
- [ ] X100 Core specifications (current: 8×X100 @2.4GHz)
- [ ] A100 AI Core specifications (current: 8×A100, ~60 TOPS)
- [ ] Memory specifications (current: LPDDR5-6400, ~51.2 GB/s bandwidth)
- [ ] Ethernet specifications (current knowledge base says: Dual Gigabit, RGMII/SGMII)
- [ ] Cache hierarchy

#### Software
- [ ] Bianbu OS official description and features
- [ ] SDK composition (bootloader, kernel, rootfs, toolchain)
- [ ] Supported OS list (Bianbu, OpenWrt, Buildroot, OpenHarmony, ROS 2)

#### AI Capabilities
- [ ] Matrix Extension implementation details (K1 instruction-level vs K3 hardware)
- [ ] Supported model sizes (current: 7B-13B parameters for K3)
- [ ] AI SDK components and features
- [ ] LLM SDK specifics

#### Tools
- [ ] Flashing tool official name and features
- [ ] JTAG debugging interface specifications
- [ ] Trace tool capabilities
- [ ] SpacemiT Studio IDE features

### 3. Terminology Verification

Check official English/Chinese term pairs:
- [ ] "Matrix Extension" → 矩阵扩展
- [ ] "Flashing Tool" → 烧录工具 or 刷机工具?
- [ ] "Bianbu" → 边部系统 (verify official translation)
- [ ] "AI Core" vs "AI Engine" terminology

### 4. Product Line Verification

- [ ] K1 product variants and specifications
- [ ] K3 product variants and specifications  
- [ ] P1/P1S PMIC specifications
- [ ] MUSE series board specifications
- [ ] RV2768 cluster specifications

### 5. Documentation Structure

Map official doc structure to current knowledge base:
- [ ] Hardware section coverage
- [ ] Software section coverage
- [ ] AI section coverage
- [ ] Tools section coverage
- [ ] Cloud services section coverage

### 6. Corrections Needed

Based on GitHub verification, document:
- Factual errors to fix
- Missing content to add
- Outdated specifications to update
- Broken links to repair

## Execution Plan

1. **Manual GitHub Review** (requires user assistance)
   - Browse https://github.com/orgs/spacemit-com/repositories
   - Clone key `docs-*` repos locally if available
   - Extract key specifications

2. **Automated Verification** (when connectivity restored)
   - Use GitHub API to fetch repo list
   - Clone or browse repos programmatically
   - Extract specs from markdown/rst files

3. **Knowledge Base Update**
   - Apply corrections to affected files
   - Add missing content
   - Update PROGRESS.md with findings

## Status

**Current**: Network connectivity issues prevent direct GitHub access
**Next Step**: User assistance needed to access https://github.com/spacemit-com/ repos
