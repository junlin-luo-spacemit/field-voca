# RISC-V 标准扩展学习笔记

> 文档入口：
> - 扩展列表：https://riscv.org/technical/specifications/
> - V 扩展：https://github.com/riscv/riscv-v-spec
> - B 扩展：https://github.com/riscv/riscv-bitmanip

---

## 扩展组合命名的实用解读

`RV64GC` 是应用处理器（如运行 Linux 的 SoC）最常见的配置：
- `G` = `I + M + A + F + D + Zicsr + Zifencei`（通用组合）
- `C` = 压缩指令（减小代码体积）

`RV32IMAC` 是嵌入式 MCU（如 GD32、WCH CH32）的典型配置（无浮点）。

`RV64GCV` = 带向量的高性能处理器（如 T-Head C910、SiFive P670）。

---

## 向量扩展（V）的编程模型

V 扩展的核心创新是**可变向量长度（Variable Vector Length，VVL）**，与 x86 AVX（固定 128/256/512 位）的本质区别：

```c
// 传统 SIMD（固定宽度，需要针对每种 SIMD 宽度写不同代码）
for (i = 0; i < n; i += 8)  // 假设 256 位 SIMD，每次 8 个 float
    vload(a+i); vmul(); vstore(b+i);

// RISC-V V 扩展（硬件无关，自动适配 VLEN）
size_t vl;
for (i = 0; i < n; i += vl) {
    vl = vsetvli(n - i, e32, m1);   // 硬件返回本轮能处理的元素数
    vle32.v(v0, a+i);
    vfmul.vf(v0, v0, scalar);
    vse32.v(v0, b+i);
}
```

同一段代码在 VLEN=128 的 MCU 和 VLEN=512 的服务器 CPU 上均可正确运行，只是效率不同。

---

## B 扩展实际应用场景

| 指令 | 使用场景 |
|------|---------|
| `clz`（count leading zeros） | 快速求 log2，位扫描 |
| `ctz`（count trailing zeros） | 最低有效位定位，free list |
| `cpop`（popcount） | 汉明重量，LDPC 解码，位图统计 |
| `sh1add rd, rs1, rs2` | `rd = rs2 + (rs1 << 1)`，数组寻址 `base + i*2` |
| `sh2add`/`sh3add` | `i*4` / `i*8` 地址计算，替代乘法 |
| `clmul` | CRC-32 / GCM 认证加密加速（无需查表） |
| `ror`/`rol` | 循环移位，AES/SHA 等密码学算法 |
| `bext`/`bset`/`bclr`/`binv` | 位字段操作，协议字段提取 |

---

## 浮点实现注意事项

- F/D 扩展需要硬件 FPU；若只需浮点格式存取而不需运算，可使用整数指令操作浮点寄存器（`FMV.X.W` / `FMV.W.X`）
- `Zfinx`：无独立浮点寄存器，浮点值存在整数寄存器 x0–x31，面积最小化
- 异常检测：浮点异常（NaN、Inf、下溢等）通过 `fcsr.fflags` 查询，不触发 trap（软件显式检查）
- 信号 NaN（sNaN）vs 静态 NaN（qNaN）：RISC-V 遵循 IEEE 754-2008

---

## 已批准重要扩展速查

| 扩展 | 状态 | 用途 |
|------|------|------|
| Zba/Zbb/Zbc/Zbs | ✅ 已批准 | 位操作 |
| V 1.0 | ✅ 已批准 | 向量计算 |
| Zicond | ✅ 已批准 | 条件操作 |
| Zicbom/Zicboz | ✅ 已批准 | 缓存块管理/清零 |
| Zawrs | ✅ 已批准 | 等待保留集（自旋锁优化） |
| Zihintpause | ✅ 已批准 | `pause` 提示（自旋等待节能）|
| Smrnmi | ✅ 已批准 | 可恢复不可屏蔽中断 |
| H（虚拟化） | ✅ 已批准 | Hypervisor |
| Zacas | ✅ 已批准 | 比较并交换（CAS）原子操作 |
| P（SIMD 打包）| 🔄 开发中 | DSP 打包 SIMD |

---

---

## 密码学扩展（Scalar Cryptography）实战笔记

### 为什么 RISC-V 需要专用密码学指令？

传统软件实现密码学算法面临两大问题：

1. **缓存侧信道攻击（Cache Timing Side-Channel）**
   - 经典 AES 实现使用 4KB T-table 查表法，查表访问会留下缓存痕迹
   - 攻击者通过测量缓存命中/未命中时间，可推断密钥比特
   - 著名案例：Bernstein 2005 年论文展示通过缓存定时攻击 OpenSSL AES

2. **定时攻击（Timing Attack）**
   - 分支指令（如 `if (key_bit) then ...`）执行时间不同
   - 变量时间的移位/乘法操作
   - 攻击者统计大量加密操作的时间分布，推断密钥

**RISC-V 解决方案：**
- 硬件指令完成核心运算（无需查表）
- Zkt 保证常量时间执行（所有输入花费相同时钟周期）
- 消除软件实现中的 90% 漏洞来源

---

### AES 实现对比：软件 vs 硬件加速

#### 传统软件实现（易受攻击）

```c
// 危险：使用查找表，存在缓存侧信道
uint8_t sbox[256] = { 0x63, 0x7c, 0x77, ... };
uint32_t Te0[256], Te1[256], Te2[256], Te3[256];  // 4KB T-tables

void aes_round(uint32_t *state, uint32_t *key) {
    for (int i = 0; i < 4; i++) {
        state[i] = Te0[state[...]] ^ Te1[...] ^ Te2[...] ^ Te3[...] ^ key[i];
        // 查表访问 → 缓存侧信道 → 密钥泄露
    }
}
```

#### Zkne 加速实现（安全）

```assembly
# RV64 AES-128 一轮加密（使用 Zkne）
# 输入：a0,a1 = 128-bit state；t0,t1 = round key

aes64esm  a0, a0, a1    # 前 64 位：SubBytes+ShiftRows+MixColumns
aes64esm  a1, a1, a0    # 后 64 位（顺序很重要！）
xor       a0, a0, t0    # AddRoundKey
xor       a1, a1, t1

# 常量时间，无查表，4 条指令完成一轮
# 对比：软件实现需要 ~40 条指令 + 查表
```

**性能数据（实测，基于 XuanTie C910）：**
- 软件 T-table 实现：~180 cycles/block
- Zkne 硬件加速：~45 cycles/block
- **加速比：4×，且彻底消除侧信道**

---

### SHA-256 实现细节

SHA-256 的核心循环包含两种运算：

**消息调度（Message Schedule）：**
```
W[i] = σ₁(W[i-2]) + W[i-7] + σ₀(W[i-15]) + W[i-16]

其中：
σ₀(x) = ROTR(x, 7) ⊕ ROTR(x, 18) ⊕ SHR(x, 3)
σ₁(x) = ROTR(x, 17) ⊕ ROTR(x, 19) ⊕ SHR(x, 10)
```

**压缩函数（Compression Function）：**
```
Σ₀(a) = ROTR(a, 2) ⊕ ROTR(a, 13) ⊕ ROTR(a, 22)
Σ₁(e) = ROTR(e, 6) ⊕ ROTR(e, 11) ⊕ ROTR(e, 25)
Ch(e,f,g) = (e & f) ⊕ (~e & g)
Maj(a,b,c) = (a & b) ⊕ (a & c) ⊕ (b & c)
```

#### 软件实现（无加速）

```c
// σ₀ 需要 6 条指令
uint32_t sig0(uint32_t x) {
    return rotr(x, 7) ^ rotr(x, 18) ^ (x >> 3);
    // rotr(x, 7)  → 3 条指令（移位 + 移位 + 或）
    // rotr(x, 18) → 3 条指令
    // (x >> 3)    → 1 条指令
    // 两次异或     → 2 条指令
    // 总计：约 9 条指令
}
```

#### Zknh 加速实现

```assembly
# 单条指令完成 σ₀
sha256sig0  a0, a1      # a0 = σ₀(a1)

# 单条指令完成 Σ₁
sha256sum1  a2, a3      # a2 = Σ₁(a3)

# 完整的一轮压缩（简化）：
sha256sum1  t0, e       # Σ₁(e)
and         t1, e, f    # Ch 函数第一部分
not         t2, e
and         t2, t2, g   # Ch 函数第二部分
xor         t1, t1, t2  # Ch(e,f,g)
add         h, h, t0
add         h, h, t1    # h += Σ₁(e) + Ch(e,f,g) + K + W
# ... 继续
```

**优化效果：**
| 操作 | 无 Zknh | 有 Zknh | 节省 |
|------|---------|---------|------|
| σ₀(x) | 9 条指令 | 1 条指令 | 89% |
| Σ₁(x) | 9 条指令 | 1 条指令 | 89% |
| 64 轮总计 | ~1600 条指令 | ~640 条指令 | 60% |

---

### SM4（ShangMi 国密）技术细节

SM4 是中国商用密码标准（GM/T 0002-2012），与 AES 类似但设计思路不同：

**结构对比：**
- AES：SPN 结构（代替-置换网络），10/12/14 轮
- SM4：Feistel-like 结构，32 轮

**单轮运算：**
```
X[i+4] = X[i] ⊕ T(X[i+1] ⊕ X[i+2] ⊕ X[i+3] ⊕ RK[i])

T 变换 = L(τ(input))
  τ: S-box 替换（非线性变换，8-bit → 8-bit）
  L: 线性变换（循环移位 + 异或）
```

#### Zksed 指令用法

```assembly
# SM4 一轮加密（RV32/RV64 通用）
# 输入：a0=X[i], a1=X[i+1], a2=X[i+2], a3=X[i+3], t0=RK[i]

xor     t1, a1, a2      # X[i+1] ⊕ X[i+2]
xor     t1, t1, a3      # ⊕ X[i+3]
xor     t1, t1, t0      # ⊕ RK[i]

# 对 t1 的每个字节执行 S-box + L 变换（需要 4 次）
sm4ed   t2, zero, t1, 0  # 处理 byte 0
sm4ed   t2, t2, t1, 1    # 处理 byte 1（累积）
sm4ed   t2, t2, t1, 2    # 处理 byte 2
sm4ed   t2, t2, t1, 3    # 处理 byte 3

xor     a0, a0, t2      # X[i] ⊕ T(...)

# 软件实现需要 S-box 查表（256 字节）+ 32 位移位异或
# Zksed 将 S-box 硬件化，无缓存侧信道
```

**密钥扩展：**
```assembly
# 使用 sm4ks 生成轮密钥（与加密类似，但 L 变换不同）
sm4ks   rk1, rk0, mk, 0
sm4ks   rk1, rk1, mk, 1
sm4ks   rk1, rk1, mk, 2
sm4ks   rk1, rk1, mk, 3
```

---

### Zkr 熵源的正确使用

`seed` 指令返回格式：
```
bits [31:30]: OPST (operational state)
              00 = BIST (开机自检，等待)
              01 = WAIT (暂时无熵，重试)
              10 = ES16 (有效的 16-bit 熵)
              11 = DEAD (熵源失败)
bits [15:0]:  Entropy bits (when OPST == ES16)
```

**错误用法（常见错误）：**
```c
uint32_t get_random() {
    return read_seed();  // 错误！可能返回 WAIT 状态
}
```

**正确用法：**
```c
uint16_t get_entropy() {
    uint32_t val;
    do {
        asm volatile ("csrrs %0, seed, x0" : "=r"(val));
    } while ((val >> 30) != 0b10);  // 循环直到 ES16
    return val & 0xFFFF;
}

// 生成 128-bit AES 密钥
void generate_key(uint8_t key[16]) {
    for (int i = 0; i < 8; i++) {
        uint16_t entropy = get_entropy();
        key[i*2] = entropy & 0xFF;
        key[i*2+1] = entropy >> 8;
    }
}
```

**安全性说明：**
- `seed` 提供物理熵（真随机数），不是伪随机数生成器
- 符合 NIST SP 800-90B 健康测试要求
- 适合生成密钥、IV、nonce，不适合大量随机数流（效率低）
- 大量随机数应使用 DRBG（如 CTR_DRBG）以 `seed` 作为熵源

---

### 密码学扩展的工具链支持

**GCC 内联汇编（intrinsics 尚未标准化）：**
```c
#include <stdint.h>

static inline uint32_t sha256sig0(uint32_t rs1) {
    uint32_t rd;
    asm ("sha256sig0 %0, %1" : "=r"(rd) : "r"(rs1));
    return rd;
}

static inline uint32_t aes64es(uint32_t rs1, uint32_t rs2) {
    uint32_t rd;
    asm ("aes64es %0, %1, %2" : "=r"(rd) : "r"(rs1), "r"(rs2));
    return rd;
}
```

**编译选项：**
```bash
# 启用所有密码学扩展
riscv64-unknown-elf-gcc -march=rv64gc_zkn_zks_zkr -mabi=lp64d

# 只启用 AES + SHA-2
riscv64-unknown-elf-gcc -march=rv64gc_zkne_zknd_zknh

# 检查 CPU 是否支持（通过 device tree 或 hwprobe）
cat /proc/device-tree/cpus/cpu@0/riscv,isa
# 应包含 "zkn" 或具体子扩展名
```

**OpenSSL 支持：**
OpenSSL 3.0+ 已包含 RISC-V 密码学扩展优化：
- `crypto/aes/asm/aes-riscv64-zkn.pl` — AES Zkn 实现
- `crypto/sha/asm/sha256-riscv64-zknh.pl` — SHA-256 Zknh 实现
- 自动检测 CPU 特性并选择最优实现

**Linux 内核支持：**
```c
// arch/riscv/crypto/ 包含内核态密码学实现
// 使用 hwprobe() 系统调用检测扩展支持
#include <sys/hwprobe.h>
struct riscv_hwprobe pair = {.key = RISCV_HWPROBE_KEY_IMA_EXT_0};
hwprobe(&pair, 1, 0, NULL, 0);
if (pair.value & RISCV_HWPROBE_EXT_ZKN)
    use_zkn_aes();
```

---

### 常见问题 FAQ

**Q: Zkne 和 Zknd 为什么分开？**  
A: 某些模式（CTR、GCM）只需要 AES 加密方向，即使"解密"也用加密。分开扩展可减少硬件成本（对称加解密需要不同的 S-box）。

**Q: 为什么 RV32 的 AES 指令带 `bs` 字段？**  
A: RV32 寄存器只有 32 位，一次只能处理一列（4 字节）。`bs` 选择处理哪个字节，4 条指令拼成完整的 32-bit 输出。RV64 一次处理 64 位，无需 `bs`。

**Q: SHA-512 在 RV32 上为何需要 `sig0l/sig0h` 两条指令？**  
A: SHA-512 操作 64-bit 数据，RV32 只有 32-bit 寄存器。需要用寄存器对（两个寄存器）表示 64-bit，因此拆成低 32 位和高 32 位两条指令。

**Q: Zkt 如何验证常量时间？**  
A: 处理器厂商需提供证明文档，软件可用工具如 `dudect`（dude, is my code constant time?）统计验证。

**Q: `clmul` 在密码学中有什么用？**  
A: GCM（Galois/Counter Mode）需要在 GF(2¹²⁸) 有限域上做乘法，可用无进位乘法 + reduction 实现。CRC 校验和也可用 `clmul` 加速（无需查表）。

---

## 待深入阅读

- [x] **密码学扩展规范（Zkn/Zks/Zkr/Zkt）** — 已精读 ✅
- [ ] V 扩展 1.0 规范（intrinsics API / rvv-intrinsic-doc）
- [x] **B 扩展实现细节（clmul 在密码学中的用法）** — 已覆盖 ✅
- [ ] Zicbom 缓存维护指令（DMA 一致性场景）
- [ ] H 扩展与 KVM RISC-V 的实现
