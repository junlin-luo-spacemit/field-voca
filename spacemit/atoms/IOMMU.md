---
tags: [atom, iommu, spacemit]
domain: hardware
aliases: [IOMMU, I/O Memory Management Unit, 输入输出内存管理单元]
status: needs_review
created: 2026-08-06
updated: 2026-10-29
source:
superseded_by:
---

# IOMMU（输入输出内存管理单元）

**中文译名**：输入输出内存管理单元

## 定义

IOMMU（Input-Output Memory Management Unit，输入输出内存管理单元）为外设访问内存提供虚拟化与隔离，支持设备的 DMA 虚拟地址映射和安全控制。

## 为什么重要

在多租户或需要外设虚拟化的系统中，IOMMU 可防止外设越界访问内存并支持设备直通与性能隔离。

## 在 SpacemiT 中

- 产品列表提及 T100 IOMMU 作为计算 IP，用于外设虚拟化与加速。

## 相关词条

- [[SoC]] — 系统级芯片
