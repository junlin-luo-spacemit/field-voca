---
tags: [study, day2, spacemit, boot-chain]
domain: software
status: approved
created: 2026-11-16
updated: 2026-11-16
source: C:\Users\luojunlin\docs-buildroot\zh\k1_buildroot\device\boot.md, C:\Users\luojunlin\docs-buildroot\zh\k3_buildroot\device\boot.md
---

# Day 2 · 第一课：启动链（Boot Chain）

## 学习目标

能画出从"通电"到"程序运行"的六个层级，说出每一层的名称和职责。

**配套互动动画**：用浏览器打开 [boot-chain-animation.html](./quiz/boot-chain-animation.html) — 点击「开始」逐层展开，每层都有小白类比。建议**先看动画，再读下面内容**。

---

## 完整启动链（K1 / K3 通用）

> 芯片通电就像叫醒一个完全失忆的人，每一层引导程序把他的记忆和能力一点点还给他，直到他能独立工作为止。

```
通电
  │
  ▼
1. BROM（Boot ROM）—— "保安大叔"
   · 芯片出厂固化，永远不会变，不可修改
   · 唯一任务：去存储介质里找到 FSBL，把它加载进来
  │
  ▼
2. FSBL（First Stage Boot Loader）—— "开灯的人"
   · 为什么叫开灯的人？此时内存（DRAM）还不能用，就像进了一间漆黑的房间
   · SpacemiT 自定义的第一阶段引导
   · 他先初始化内存控制器（把灯打开），再加载 OpenSBI
  │
  ▼
3. OpenSBI（RISC-V Supervisor Binary Interface）—— "翻译官"
   · RISC-V 专有的固件层（x86 / ARM 没有这一层）
   · 硬件最底层的操作（时钟、关机）只有他能做
   · Linux 内核想控制硬件，必须通过他"翻译"
  │
  ▼
4. U-Boot —— "接待员"
   · 为什么叫接待员？每次启动都要准备好所有硬件，"接待"内核入住
   · 通用引导加载程序（开源 + SpacemiT 适配）
   · 初始化硬件外设，加载 Linux 内核 + 设备树（DTB）
   · 特殊技能：进入 Fastboot 模式 → 刷机
  │
  ▼
5. Linux 内核（Kernel）—— "大管家"
   · 为什么叫大管家？他要协调所有硬件（部门），让程序（员工）能正常工作
   · K1 用 Linux 6.1，K3 用 Linux 6.6+
   · 接管所有硬件驱动，挂载根文件系统
  │
  ▼
6. Rootfs（根文件系统）—— "办公室"
   · 选择操作系统：Bianbu / Buildroot / OpenWrt / OpenHarmony
   · 程序从这里开始运行
  │
  ▼
你的程序跑起来了 🎉
```

**每一层"是谁做的"**

| 层级 | 名称 | 谁做的 | 可不可以改 |
|---|---|---|---|
| 1 | BROM | SpacemiT 芯片内固化 | ❌ |
| 2 | FSBL | SpacemiT 开发 | ✅ |
| 3 | OpenSBI | RISC-V 开源社区 | ✅ |
| 4 | U-Boot | 开源社区 + SpacemiT 适配 | ✅ |
| 5 | Linux 内核 | Linux 社区 + SpacemiT BSP | ✅ |
| 6 | Rootfs | SpacemiT / 开源（4 种可选） | ✅ |

### 进阶视角：权限层级的切换

> 从 RISC-V 权限角度看，启动是一个"降权 + 常驻高权限服务"的过程。小白可跳过此节，不影响理解主流程。

RISC-V 把权限分成三层：**M-Mode**（机器模式，最高权限）、**S-Mode**（监管模式）、**U-Mode**（用户模式）。启动时权限的切换过程：

```
FSBL（在 M-Mode 运行）
  ↓ 加载 OpenSBI
OpenSBI（接管 M-Mode，常驻不退出）
  ↓ 加载 U-Boot，降权到 S-Mode
U-Boot（在 S-Mode 运行）
  ↓ 加载 Linux 内核
Linux 内核（在 S-Mode 运行）
  ↓ 需要底层操作时，通过 SBI 调用请求 OpenSBI
OpenSBI（M-Mode，响应请求）
```

**关键点：**
- FSBL 完成初始化后，把 M-Mode 的控制权交给 OpenSBI
- **OpenSBI 常驻 M-Mode**，不会退出，像"安保主管"永远守在最高权限层
- U-Boot 和 Linux 内核都运行在 S-Mode（降权）
- 内核想做底层操作（关机、调时钟），必须通过 **SBI 调用**请求 OpenSBI 代劳

**类比时间线：**
1. **FSBL** = 第一个到公司的保安，开门后把钥匙（M-Mode 控制权）交给安保主管，自己退场
2. **OpenSBI** = 总机房的安保主管，接过钥匙后永远守在最高权限层（M-Mode）
3. **Linux 内核** = 楼层经理（S-Mode），想动总电闸得找安保主管批准

---

## K3 比 K1 多了什么

K3 可以在 OpenSBI 之前插入一个**可选的安全固件层**：

```
K1：BROM → FSBL → OpenSBI → U-Boot → Kernel → Rootfs
K3：BROM → FSBL → [OP-TEE] → OpenSBI → U-Boot → Kernel → Rootfs
```

**OP-TEE**（Trusted Execution Environment）提供硬件级安全隔离（安全启动、密钥保护）。K3 Buildroot v1.0.9 起支持，存放在独立 TEE 分区。

---

## 固件存在哪里：存储分区布局

> 你不需要记住地址，只需要知道固件是**分开存放**的，写文档时不要把它们说成"一个镜像文件"。

把存储介质想象成一栋楼：

```
NOR Flash（固件仓库）         SSD / eMMC（系统仓库）
┌─────────────────────┐      ┌─────────────────────┐
│  1楼：FSBL.bin      │      │  A区：bootfs        │  ← 内核 + DTB
│  2楼：opensbi.itb   │      │  B区：rootfs        │  ← 根文件系统
│  3楼：u-boot.itb    │      └─────────────────────┘
└─────────────────────┘
```

`FSBL.bin`、`opensbi.itb`、`u-boot.itb` 是**三个独立文件**，刷机时需要分别烧写到各自分区。

---

## 刷机 vs 启动

| | 启动（Boot） | 刷机（Flash） |
|---|---|---|
| 是什么 | 读取已有固件，逐层跑起系统 | 把新固件写入存储，替换旧的 |
| 什么时候 | 每次上电或重启 | 升级系统 / 初始化设备 |

> 官方文档：刷机与启动相互关联——如果修改了启动逻辑，刷机配置通常也要跟着改。

### 刷机用什么工具？

**TitanFlasher（GUI，给普通用户）**：选一个 `.zip` 整包镜像 → 点「开始烧录」，工具自动把每个文件烧到正确分区。类比：iPhone 点"立即更新"，手机自己处理所有文件。

**fastboot 命令行（给开发工程师）**：手动逐个烧写：

```bash
fastboot flash fsbl    factory/FSBL.bin
fastboot flash opensbi fw_dynamic.itb
fastboot flash uboot   u-boot.itb
fastboot flash bootfs  bootfs.img
fastboot flash rootfs  rootfs.ext4
```

**写文档时的判断**：同一个动作，给不同读者写法完全不同——给用户写"一键烧录"，给工程师写每条命令。这是技术写作最核心的判断之一。

---

## 三个常见术语解释

### OpenSBI 是什么？

x86 有 BIOS / UEFI，RISC-V 有 **OpenSBI**——功能类似，都是管理硬件最底层操作的固件层。

OpenSBI 常驻 **M-Mode**（机器模式，RISC-V 最高权限），Linux 内核跑在 **S-Mode**（监管模式）。内核想做底层操作（关机、调时钟），必须通过 **SBI 调用**请求 OpenSBI 代劳。

**类比：** OpenSBI = 董事长（M-Mode），Linux 内核 = 部门经理（S-Mode），你的程序 = 普通员工（U-Mode）。部门经理想调全公司空调温度，得找董事长批准，不能自己动总开关。

写文档时看到"需要 OpenSBI 支持"、"SBI 版本"，知道它是 RISC-V 专有的"底层固件 / 最高权限层"就够了。详细的权限切换过程见上面"进阶视角"章节。

### 设备树（DTB）是什么？

U-Boot 加载"内核 + 设备树（DTB）"——DTB 是一份**硬件说明书**，告诉 Linux 内核这块板子上有哪些硬件（USB 口几个、内存多大、摄像头用哪个接口）。内核是通用的，不可能内置所有板子的信息，所以用 DTB 单独描述。

| 格式 | 扩展名 | 用途 |
|---|---|---|
| DTS | `.dts` | 人类可读源码，工程师修改这个 |
| DTB | `.dtb` | 编译后的二进制，U-Boot 加载这个 |

常见说法解读：
- "bootfs 包含 kernel 和 DTB" → 两个文件：内核 + 硬件配置
- "适配新板子需要修改 DTS" → 修改硬件说明书的源码

### Linux 和 Ubuntu 是什么关系？

**Linux = 引擎（内核），Ubuntu = 整辆车（完整操作系统）。**

Linux 内核是开源的，任何人都可以拿来加工成自己的操作系统（发行版）：

| 发行版 | 定位 | 类比 |
|---|---|---|
| Ubuntu | 通用桌面 / 服务器 | 大众汽车 |
| Buildroot | 极简嵌入式 | 卡丁车（只有发动机和轮子） |
| OpenWrt | 路由器专用 | 摩托车 |
| **Bianbu** | 基于 Ubuntu，针对 SpacemiT K1/K3 优化 | 大众改装版（专为 RISC-V 路面） |

Bianbu 就是"Ubuntu 的 RISC-V 版"：
```
Ubuntu 24.04.1  →  Bianbu 2.x（K1 专用）
Ubuntu 26.04    →  Bianbu 4.x（K3 专用，仅 K3）
```

---

## 易错点

| 错误 | 正确 |
|---|---|
| "启动和刷机是同一件事" | 完全不同：启动是**读取**运行，刷机是**写入**替换 |
| "OpenSBI 和 BIOS 没有关系" | 功能类似，OpenSBI 是 RISC-V 专有的 BIOS/UEFI 等价物 |
| "K3 和 K1 启动链完全一样" | K3 多了可选的 **OP-TEE** 安全固件层 |
| "固件是一个大文件" | FSBL / OpenSBI / U-Boot 是**三个独立文件**，分别烧写 |
| "Bianbu 2.x 也支持 K3" | Bianbu 2.x 仅支持 K1；K3 用 **Bianbu 4.x** |

---

## 关键记忆点

1. BROM 是芯片内固化的，**不可修改**
2. FSBL 是 SpacemiT **自定义**的第一阶段引导
3. **OpenSBI 是 RISC-V 专有**的，x86 / ARM 没有
4. 刷机 ≠ 启动：刷机是**写入**，启动是**读取**
5. K3 多 **OP-TEE** 安全固件层（Buildroot v1.0.9 起支持）
6. DTB = 硬件说明书，由 U-Boot 传给 Linux 内核
7. Bianbu = Ubuntu 的 RISC-V 版（2.x → K1，4.x → K3）

---

## 作业

画一张从"通电"到"程序运行"的流程图，标出：
1. 六个层级的名称和各自职责
2. 刷机发生在哪一层（提示：U-Boot Fastboot 模式）
3. K3 比 K1 多的可选层
4. DTB 在哪一步被传给内核

## 学习目标

能画出从"通电"到"程序运行"的六个层级，说出每一层的名称和职责。

**配套互动动画**：用浏览器打开 [boot-chain-animation.html](./quiz/boot-chain-animation.html) — 点击「开始」逐层展开，每层都有小白类比说明。建议**先看动画，再读下面的详细内容**。

---

## 完整启动链（K1 / K3 通用）

> **小白类比**：芯片通电就像早晨叫醒一个完全失忆的人。他不知道自己是谁，不认识任何人，也不知道怎么说话。每一层"引导程序"就像一个叫醒他的人，把他的记忆和能力一点点还给他，直到他完全清醒，能独立工作为止。

```
通电
  │
  ▼
1. BROM（Boot ROM）—— "保安大叔"
   · 芯片出厂时已经固化在里面，永远不会变
   · 他只会做一件事：去存储介质（eMMC/SD 卡/NOR）里
     找到 FSBL 这个文件，把它加载进来
   · 类比：公司大门的保安，他不会帮你工作，
     但他会开门让真正的工作人员进来
  │
  ▼
2. FSBL（First Stage Boot Loader）—— "开灯的人"
   · SpacemiT 自定义的第一阶段引导
   · 此时内存（DRAM）还不能用——就像进了一间漆黑的房间
   · 他的任务：先把灯打开（初始化内存控制器），
     然后加载 OpenSBI
   · 类比：进办公室第一件事，把灯和空调打开，
     让后面的人能正常工作
  │
  ▼
3. OpenSBI（RISC-V Supervisor Binary Interface）—— "翻译官"
   · RISC-V 专有的一层（x86/ARM 没有这个角色）
   · 硬件最底层的操作（比如关机、时钟）只有他能做
   · 操作系统（Linux）想要操控硬件，必须通过他"翻译"
   · 类比：一个只懂硬件"机器语言"的翻译官，
     把操作系统的指令翻译成硬件能听懂的语言
  │
  ▼
4. U-Boot —— "装修工头"
   · 通用引导加载程序（开源 + SpacemiT 适配）
   · 把剩余的硬件外设都初始化好（网卡、存储、显示...）
   · 然后把 Linux 内核请进来
   · 还有一个特殊技能：进入 Fastboot 模式 → 刷机
   · 类比：装修工头，先把房间布置好，再请房主入住；
     如果需要重新装修（刷机），也找他
  │
  ▼
5. Linux 内核（Kernel）—— "大管家"
   · SpacemiT 适配版（K1 用 Linux 6.1，K3 用 Linux 6.6+）
   · 接管所有硬件驱动，管理内存、CPU、设备
   · 挂载根文件系统，准备好用户空间
   · 类比：公司大管家，所有部门（硬件）都归他协调，
     他把一切安排好，员工（程序）才能开始工作
  │
  ▼
6. Rootfs（根文件系统）—— "办公室"
   · 在这里选择操作系统版本：
     Bianbu（Ubuntu 基础）/ Buildroot（最小化）/
     OpenWrt（路由器）/ OpenHarmony（鸿蒙）
   · 类比：员工终于进了办公室，桌椅电脑都有了，
     可以开始干活了
  │
  ▼
你的程序跑起来了 🎉
```

---

## 每一层"是谁做的"

| 层级 | 名称 | 谁做的 | 可不可以改 |
|---|---|---|---|
| 1 | BROM | SpacemiT 芯片内固化 | ❌ 不可改 |
| 2 | FSBL | SpacemiT 开发 | ✅ 可编译替换 |
| 3 | OpenSBI | RISC-V 开源社区 | ✅ 可配置 |
| 4 | U-Boot | 开源社区 + SpacemiT 适配 | ✅ 可配置 |
| 5 | Linux 内核 | Linux 社区 + SpacemiT BSP | ✅ 可编译 |
| 6 | Rootfs | SpacemiT / 开源（4 种可选） | ✅ 可选 |

---

## 存储介质上的分区布局

> **技术写作需要知道的程度**：你不需要记住地址，但要知道"固件是分开存放的"——这样写文档时不会把它们说成一个东西。

### 小白版理解

把存储介质（eMMC / NOR Flash / SD 卡）想象成一栋楼，每个固件住在不同的楼层：

```
NOR Flash 芯片（小仓库，存固件）
┌─────────────────────────────┐
│  1楼：FSBL.bin              │  ← 第一个被找到的固件
│  2楼：opensbi.itb           │  ← OpenSBI 固件
│  3楼：u-boot.itb            │  ← U-Boot 固件
└─────────────────────────────┘

SSD / eMMC（大仓库，存系统）
┌─────────────────────────────┐
│  A区：bootfs                │  ← Linux 内核 + 设备树
│  B区：rootfs                │  ← 根文件系统（Bianbu / Buildroot...）
└─────────────────────────────┘
```

### 为什么重要

**写文档时的常见错误**：把固件说成"一个镜像文件"。

实际上刷机时需要**分别烧写**到对应位置——`FSBL.bin`、`opensbi.itb`、`u-boot.itb` 是三个独立文件。如果文档里写"烧录固件包"但没说清楚哪些文件要烧到哪里，工程师会看不懂。

> 如果你将来写刷机手册，记住这句话就够了：**每个固件文件对应一个存储分区，分别烧写，不能合并。**

---

## K3 比 K1 多了什么

K3 的启动链可以插入一个可选的安全固件层：

```
K1：BROM → FSBL → OpenSBI → U-Boot → Kernel → Rootfs
K3：BROM → FSBL → [OP-TEE] → OpenSBI → U-Boot → Kernel → Rootfs
                   ↑
         可选安全固件（Buildroot v1.0.9 起支持）
         存放在独立 TEE 分区（GPT 介质为 4MB 分区）
```

**OP-TEE**（Trusted Execution Environment）提供硬件级安全隔离，用于安全启动、密钥保护等。

---

## 刷机 vs 启动：两件不同的事

技术写作里这对概念最容易混淆：

| 概念 | 是什么 | 什么时候发生 |
|---|---|---|
| **启动（Boot）** | 按照存储里已有的固件，一步步把系统跑起来 | 每次上电或重启 |
| **刷机（Flash）** | 把新的固件/镜像写入存储介质，替换旧的 | 升级系统 / 初始化设备时 |

**刷机的原理**：U-Boot 进入 Fastboot 模式 → PC 通过 USB 传输镜像 → 写入 eMMC/NOR 对应分区

> 官方文档原文：刷机与启动是相互关联的，如果需要对刷机做自定义配置，启动可能也需要做相关的调整，反之亦然。

### 刷机时需要烧三个文件吗？

**要看你用什么工具**，这是技术写作里需要判断读者身份的典型场景：

#### 方式一：TitanFlasher 工具（GUI，给普通用户）

不需要手动处理三个文件。TitanFlasher 读取一个**整包 `.zip` 镜像**，工具自动把每个文件烧到正确分区。

```
你做的事：选 .zip 镜像 → 点「开始烧录」
工具做的事：自动把 FSBL.bin / opensbi.itb / u-boot.itb / rootfs 等
           分别写入各自对应的分区
```

**类比**：就像 iPhone 更新系统，你只管点"立即更新"，手机自己处理所有文件。

#### 方式二：fastboot 命令行（给开发工程师）

需要手动指定每个文件：

```bash
fastboot flash fsbl    factory/FSBL.bin
fastboot flash opensbi fw_dynamic.itb
fastboot flash uboot   u-boot.itb
fastboot flash bootfs  bootfs.img
fastboot flash rootfs  rootfs.ext4
```

**类比**：自己装系统，要分别格式化 C 盘、装驱动、装软件，每一步手动来。

#### 写文档时的判断

| 写给谁 | 用什么工具 | 文档怎么说 |
|---|---|---|
| 普通用户 / 客户快速上手 | TitanFlasher | "下载镜像包，用 TitanFlasher 一键烧录" |
| 开发工程师参考手册 | fastboot 命令行 | 列出每条 `fastboot flash` 命令和对应文件 |

**同一个动作（刷机），给不同读者写的文档是完全不同的** — 这是技术写作最核心的判断之一。

---

## OpenSBI 是什么？为什么 RISC-V 需要它？

**类比**：
- x86 有 **BIOS / UEFI** 管理硬件初始化和特权级转换
- RISC-V 有 **OpenSBI** 做同样的事

**RISC-V 三个特权级**（看得懂就行，不需要记）：

```
M-Mode（机器模式） ← OpenSBI 在这里运行
S-Mode（监管模式） ← Linux 内核在这里运行
U-Mode（用户模式） ← 你的程序在这里运行
```

写文档时看到"需要 OpenSBI 支持"、"SBI 版本"，知道它是 RISC-V 特有的"固件层"就够了。

---

## 背景知识：设备树（Device Tree / DTB）是什么？

U-Boot 章节提到"加载内核 + 设备树（DTB）"——这里解释 DTB 是什么。

### 一句话版本

> **设备树是一份"硬件说明书"，告诉 Linux 内核这块板子上有哪些硬件、怎么连的。**

### 为什么需要它？

Linux 内核是通用的，可以跑在无数种硬件上。但每块板子的配置都不一样：有几个 USB 口、内存多大、摄像头用哪个接口、显示屏连在哪里……内核不可能内置所有硬件信息，于是有了设备树——一个**单独的配置文件**，启动时由 U-Boot 传给内核。

**类比**：设备树 = 新员工入职时拿到的"办公室座位图 + 设备清单"

```
没有设备树的内核：
"我是新来的，不知道有几台电脑、打印机在哪、会议室怎么用..."

有了设备树的内核：
"好，我看了说明书：左边三台电脑，右边一台打印机，
 2楼有摄像头，USB 接在 B 号插座上..."
```

### 文件格式

| 格式 | 扩展名 | 说明 |
|---|---|---|
| DTS | `.dts` | 人类可读的源码（给工程师修改） |
| DTB | `.dtb` | 编译后的二进制文件（U-Boot 加载这个传给内核） |

### 对技术写作的意义

文档里常见的说法，现在你都能理解了：

- "加载内核 + 设备树（DTB）" → U-Boot 同时加载了内核和硬件配置文件
- "bootfs 分区包含 kernel 和 DTB" → bootfs 里存了两个文件：内核 + 硬件配置
- "适配新板子需要修改 DTS" → 要修改硬件说明书的源码

---

> 启动链的最后一层是 Rootfs，里面跑的是 Bianbu / Buildroot 等操作系统。理解它们之前，先要搞清 Linux 和 Ubuntu 的关系。

### 一句话版本

> **Linux 是引擎，Ubuntu 是整辆车。**

**Linux** 实际上指的是 **Linux 内核（Kernel）**。内核管理 CPU、内存、硬盘、网卡，让硬件能工作。但内核本身不能直接用——就像只有发动机的车，你没法开它上路。

**Ubuntu** 是在 Linux 内核上加了一堆东西之后做成的**完整操作系统**（发行版）：

```
Ubuntu = Linux 内核
       + 命令行工具（ls, cd, apt...）
       + 桌面环境（GNOME）
       + 软件包管理器（apt install）
       + 浏览器、文件管理器、各种应用
       + Ubuntu 的品牌和技术支持
```

### 类比

| 类比 | Linux 内核 | Ubuntu |
|---|---|---|
| 汽车 | 发动机 | 整辆车（发动机 + 车壳 + 方向盘 + 座椅） |
| 手机 | Android 内核 | 小米 MIUI / 华为 EMUI（都基于同一个内核） |
| 建筑 | 钢筋结构 | 整栋楼（钢筋 + 墙 + 门窗 + 装修） |

### 为什么有这么多"Linux 发行版"？

Linux 内核是**开源免费**的，任何人都可以拿来加工成自己的操作系统：

| 发行版 | 定位 | 类比 |
|---|---|---|
| **Ubuntu** | 通用桌面/服务器，新手友好 | 大众汽车 |
| **Buildroot** | 极简嵌入式，只留必要组件 | 卡丁车（只有发动机和轮子） |
| **OpenWrt** | 专为路由器优化 | 摩托车（特定用途） |
| **Bianbu** | SpacemiT 基于 Ubuntu 定制，专为 K1/K3 优化 | 大众汽车改装版（专为 RISC-V 路面） |

### 回到 SpacemiT 的语境

```
Ubuntu 24.04.1（Canonical 官方）
    ↓ SpacemiT 拿来，针对 RISC-V 重新编译 + 适配驱动
Bianbu 2.x（K1 专用）

Ubuntu 26.04（Canonical 官方）
    ↓ SpacemiT 拿来，针对 K3 重新编译 + 适配驱动
Bianbu 4.x（K3 专用）
```

Bianbu 就是"Ubuntu 的 RISC-V 版本"——底层内核、包管理器（apt）、软件生态都跟 Ubuntu 一样，只是针对 SpacemiT 芯片做了适配和优化。

### 结论

- **Linux** = 内核，不是完整的操作系统
- **Ubuntu** = 基于 Linux 内核的完整操作系统（发行版）
- **Bianbu** = 基于 Ubuntu 的、专为 SpacemiT RISC-V 芯片优化的发行版
- 说"我在用 Linux"和"我在用 Ubuntu"都对，只是粒度不同

---

## 易错点

| 错误 | 正确 |
|---|---|
| "启动和刷机是同一件事" | 完全不同：启动是**读取**运行，刷机是**写入**替换 |
| "OpenSBI 和 BIOS 是不同的东西" | 功能类似，OpenSBI 是 RISC-V 专有的 BIOS/UEFI 等价物 |
| "K3 和 K1 启动链完全一样" | K3 多了可选的 **OP-TEE** 安全固件层 |
| "固件是一个大文件" | FSBL / OpenSBI / U-Boot 是**三个独立文件**，分别存在不同分区 |

---

## 关键记忆点（5 个）

1. BROM 是芯片内固化的，**不可修改**
2. FSBL 是 SpacemiT **自定义**的第一阶段引导
3. **OpenSBI 是 RISC-V 专有**的，x86/ARM 没有
4. 刷机 ≠ 启动：刷机是**写入**固件，启动是**读取**运行
5. K3 多 **OP-TEE** 安全固件层（v1.0.9 起支持）

---

## 作业

画一张从"通电"到"程序运行"的流程图，标出：
1. 六个层级的名称
2. OpenSBI 和 U-Boot 各负责什么
3. 刷机发生在哪一层
4. K3 比 K1 多的可选层
