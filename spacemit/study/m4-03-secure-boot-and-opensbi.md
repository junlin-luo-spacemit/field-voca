---
tags: [study, m4, secure-boot, efuse, opensbi, tee, spacemit]
domain: chip
aliases: [安全启动与 OpenSBI, Secure Boot and OpenSBI]
status: needs_review
created: 2026-11-23
updated: 2026-11-23
---

# 安全启动与 OpenSBI

> **本讲目标**：理解 K3 安全启动的信任链结构，知道 eFuse 和数字签名如何保证固件完整性，以及 OpenSBI 在 RISC-V 启动链路中的角色。
>
> **📌 说明**：本模块以 **K3** 为例讲解安全启动，但信任链、签名验证、eFuse 等概念适用于所有现代 SoC 芯片。

## 一、为什么需要安全启动

普通启动模式下，设备会运行任何放在存储介质上的引导程序和内核。这带来安全风险：

- **固件替换攻击**：攻击者用恶意固件替换 Bootloader 或内核
- **供应链攻击**：设备出厂时固件被篡改

**安全启动**（Secure Boot）通过数字签名验证每个引导阶段，确保设备只运行经过授权的固件。

## 二、信任链（Chain of Trust）

K3 安全启动以 Boot ROM 为信任根，逐级验证：

```
Boot ROM（信任根，片上固化，不可修改）
  │  用 eFuse 中的公钥哈希验证 FSBL 签名
  ▼
FSBL
  │  用编译期注入的公钥验证 OpenSBI + U-Boot 签名
  ▼
OpenSBI + U-Boot
  │  用编译期注入的公钥验证内核 + 设备树签名
  ▼
Linux 内核 + 设备树
```

**关键原则**：每一级只信任上一级验证过的公钥。如果任意一级验证失败，启动停止，防止后续恶意代码运行。

## 三、eFuse：不可逆的信任根

**eFuse**（电子熔断丝）是芯片内部的一次性可编程存储单元。

| 特性 | 说明 |
|---|---|
| 存储内容 | 根公钥（ROTPK）的 SHA256 哈希值 |
| 可修改性 | **不可逆**，烧录后永久固化 |
| 作用 | Boot ROM 用 eFuse 中的哈希值验证 FSBL 的签名证书 |

**重要警告**：eFuse 一旦烧录，设备只能运行签名固件，无法回退到非签名启动。量产前必须充分测试签名流程。

**类比**：eFuse 就像门锁上的指纹信息——出厂时确认，之后只认识这把"指纹"开门的固件。

## 四、密钥体系

K3 使用分级密钥，每级职责独立：

| 密钥 | 签什么 | 公钥存在哪里 |
|---|---|---|
| 根密钥（ROTPK） | 签 FSBL 证书 | 公钥哈希烧入 eFuse |
| SPL 密钥 | 签 FSBL 二进制 | 由根密钥证书携带 |
| U-Boot 密钥 | 签 OpenSBI、U-Boot | 编译期注入 FSBL 的 DTB |
| 内核密钥 | 签内核 + 设备树 | 编译期注入 U-Boot 的 DTB |

**为什么分级？**  
根密钥极其重要，只在签发 FSBL 时使用，平时离线保存。内核密钥用于日常发版，泄露影响范围有限；换一把内核密钥，重新编译 U-Boot 注入新公钥，不需要重烧 eFuse。

## 五、FIT 格式：固件的打包容器

**FIT**（Flattened Image Tree）是 U-Boot 定义的固件打包格式，可以把多个镜像（内核、设备树、RAMDISK）打包为一个文件，并附带数字签名。

```
fw_dynamic.itb（FIT 格式）
  ├── OpenSBI 二进制
  ├── U-Boot 二进制
  └── 签名数据（SHA256 + RSA2048）

Image.itb（FIT 格式）
  ├── Linux 内核二进制
  ├── 设备树（DTB）
  └── 签名数据
```

**优点**：一个文件包含完整的镜像和签名，验证和传输都更方便。

## 六、OpenSBI：RISC-V 的固件层

**OpenSBI**（Open Supervisor Binary Interface）是 RISC-V 生态的开源固件层，定义了 M 模式（Machine Mode）对上层软件的标准接口。

### RISC-V 特权级

RISC-V 定义了三个特权级，权限从高到低：

| 特权级 | 名称 | 运行什么 |
|---|---|---|
| M 模式 | Machine Mode | OpenSBI，最高权限，直接访问硬件 |
| S 模式 | Supervisor Mode | Linux 内核，通过 SBI 调用访问底层 |
| U 模式 | User Mode | 普通应用程序，权限最低 |

### OpenSBI 的作用

**OpenSBI 解决了什么问题？**

没有 OpenSBI 时，每个 SoC 厂商需要为 Linux 内核提供定制的底层代码（定时器、IPI、电源管理）。有了 OpenSBI，Linux 只需调用标准 SBI 接口，不需要知道具体芯片的实现细节。

```
Linux 内核（S 模式）
  │  调用 SBI（标准接口）
  ▼
OpenSBI（M 模式）
  │  把 SBI 调用转为实际的硬件操作
  ▼
K3 硬件
```

**类比**：OpenSBI 像是操作系统和硬件之间的"翻译官"——内核说"我要设置定时器"，OpenSBI 把这个标准请求翻译成 K3 具体寄存器的操作。

## 快速自测

1. 信任链从哪里开始，谁是最终的"信任根"？
2. eFuse 烧录后不可逆，这意味着量产前需要做好什么？
3. OpenSBI 在 RISC-V 启动链路中解决了什么问题？

## 参考来源

| 来源 | 章节 | 内容 |
|---|---|---|
| docs-buildroot · k3_buildroot/device/secureboot.md | 安全启动机制、密钥体系 | 信任链、eFuse、FIT 签名、密钥分级 |
| docs-buildroot · k3_buildroot/device/boot.md | 固件布局 | FIT 容器格式、分区结构 |
| docs-chip · key_stone/k3/k3_docs/k3_ds.md | 2.2.1 片上存储 | eFuse、Boot ROM |
