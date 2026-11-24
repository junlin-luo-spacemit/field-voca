---
tags: [study, m3, storage, emmc, ufs, sd, qspi, spacemit]
domain: chip
aliases: [存储接口, Storage Interfaces]
status: needs_review
created: 2026-11-19
updated: 2026-11-19
---

# 存储接口：eMMC、UFS、SD 和 QSPI

> **本讲目标**：理解 K3 支持的四种存储接口（eMMC、UFS、SD/MMC、Quad-SPI）的特点和适用场景，知道选哪种取决于什么。
>
> **📌 说明**：本模块以 **K3** 为例讲解存储接口，但 eMMC/UFS/SD/QSPI 的用途划分和选型原则适用于所有 SoC 芯片。

## 一、四种存储接口一览

K3 支持四种外部存储接口，各自针对不同的用途：

| 接口 | 最高速度 | 用途 | 典型容量 |
|---|---|---|---|
| eMMC | 400 MB/s（HS400） | 主存储，放操作系统和应用 | 8 GB – 256 GB |
| UFS | 高速（HS-GEAR3） | 高性能主存储，速度比 eMMC 快 | 32 GB – 512 GB |
| SD/MMC | 104 MB/s（SDR104） | 可热插拔、可扩展存储 | 最高 2 TB |
| Quad-SPI | 低速，时钟 13.25–102 MHz | 启动固件、配置文件 | 1 MB – 128 MB |

> 存储接口与[内存](GLOSSARY.md#内存与存储)不同：内存（LPDDR5）是运行时临时数据，断电即失；存储接口连接的是持久化存储设备，断电不丢失。

## 二、eMMC：最常见的主存储

**eMMC**（embedded Multi-Media Card，嵌入式多媒体卡）是大多数嵌入式产品的主存储。

- 符合 eMMC 5.1 规范，8 位数据宽度
- 最高速度：HS400 模式 400 MB/s
- 直接焊在板上，不可热插拔，可靠性高

**用途**：中低端产品的主存储，放操作系统、应用程序和用户数据。

## 三、UFS：高性能主存储

**UFS**（Universal Flash Storage，通用闪存存储）是比 eMMC 更快的存储接口，常见于高端手机和嵌入式产品。

- 符合 JEDEC UFS 2.2 规范，最多 2 TX + 2 RX 通道（全双工）
- 支持 HS-GEAR3 高速模式和 PWM-GEAR1 低速模式
- 支持标准 SCSI 命令，可直接从 UFS 启动

**eMMC 与 UFS 的核心区别** —— 不在于"谁更新"，而在于**能不能同时读写**：

| 对比项 | eMMC | UFS |
|---|---|---|
| 接口类型 | 并行总线 | 串行差分 |
| 全双工 | 否（读写不能同时进行） | 是（读写同时进行） |
| 速度 | 最高 400 MB/s | 高于 eMMC |
| 成本 | 较低 | 较高 |
| 适合场景 | 通用嵌入式 | 高性能、高频读写 |

## 四、SD/MMC：可扩展存储

**SD/MMC** 支持标准 SD 存储卡，最大的特点是**可热插拔**。

- 符合 SD 3.0 UHS-I 规范，4 位数据宽度
- 最高速率：SDR104 模式约 104 MB/s
- 支持卡插拔检测（通过 GPIO）

**用途**：用户自行扩展容量、用换卡的方式现场更换系统、采集并保存大量数据（如摄像头录像和日志）。

## 五、Quad-SPI：启动固件专用

**Quad-SPI**（四线 SPI）是一种低速串行 Flash 接口，专门用来存**启动代码**。

- 时钟频率 13.25 MHz 至 102 MHz，数据线宽 1/2/4 线可配置
- 支持 SPI NOR Flash 和 SPI NAND Flash
- 容量小（几 MB 到几十 MB），价格便宜
- 支持 **XIP**（Execute-In-Place，就地执行）：CPU 直接在这块 Flash 上执行代码，不必先复制到内存

**为什么需要它**：Boot ROM 之后要加载的引导程序需要有个地方存。此时 LPDDR5 还没初始化，引导程序只能先加载到 SRAM 里运行，而存放它的介质要求便宜、简单、上电即可读——这就是 Quad-SPI 的位置。

## 六、怎么选存储接口

```
需要热插拔或用户扩展？
  是 → SD/MMC
  否 → 需要存放启动固件（几 MB）？
         是 → Quad-SPI NOR Flash
         否 → 对性能要求很高（高频读写）？
                是 → UFS
                否 → eMMC（性价比最高）
```

**实际产品通常同时用两三种**：Quad-SPI 存引导代码，eMMC 或 UFS 存系统和应用，SD 卡槽按需提供用户扩展。

## 快速自测

1. eMMC 和 UFS 的核心区别是什么？为什么它决定了吞吐量？
2. 为什么需要 Quad-SPI？直接用 eMMC 存引导代码可以吗？
3. 什么场景下会同时用 eMMC 和 SD 卡？

## 参考来源

| 来源 | 章节 | 内容 |
|---|---|---|
| docs-chip · key_stone/k3/k3_docs/k3_ds.md | 2.2.3 Quad-SPI | 时钟频率、数据线宽、XIP 模式 |
| docs-chip · key_stone/k3/k3_docs/k3_ds.md | 2.2.4 eMMC 接口 | eMMC 5.1 规范、速度模式 |
| docs-chip · key_stone/k3/k3_docs/k3_ds.md | 2.2.5 SD/MMC 接口 | SD 3.0 UHS-I、速度模式 |
| docs-chip · key_stone/k3/k3_docs/k3_ds.md | 2.2.6 UFS 接口 | UFS 2.2 规范、HS-GEAR3 |
