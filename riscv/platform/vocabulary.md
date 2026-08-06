---
tags: [riscv, platform, vocabulary]
domain: platform
last_updated: 2026-07-22
source: https://github.com/riscv-non-isa/riscv-sbi-doc
---

# RISC-V Platform Specification Vocabulary / RISC-V 平台规范词汇表

> Covers: SBI, ACLINT, AIA, Platform Profiles (Server/Embedded Platform)
> 涵盖：SBI、ACLINT、AIA、平台配置文件（Server/Embedded Platform）
> Source / 来源：https://riscv.org/technical/specifications/ · Non-ISA Specifications

---

## SBI（Supervisor Binary Interface）

### SBI (Supervisor Binary Interface) / 监管者二进制接口

Defines the calling interface between S-mode software (OS kernel) and M-mode firmware (e.g., OpenSBI), analogous to x86 BIOS interrupts or ARM PSCI. S-mode invokes SBI services via `ecall`, and M-mode firmware responds by operating the underlying hardware on behalf of the caller.

定义 S 模式软件（OS 内核）与 M 模式固件（如 OpenSBI）之间的调用接口，类似 x86 的 BIOS 中断或 ARM 的 PSCI。S 模式通过 `ecall` 调用 SBI 服务，M 模式固件响应并代为操作底层硬件。
**SBI Calling Convention / SBI 调用约定：**
- **Extension ID (EID)**: Passed in `a7`, identifies the SBI extension / 通过 `a7` 传递，标识 SBI 扩展
- **Function ID (FID)**: Passed in `a6`, identifies the function within the extension / 通过 `a6` 传递，标识扩展内的具体函数
- **Arguments**: `a0`-`a5` carry up to 6 input parameters / `a0`-`a5` 最多传递 6 个输入参数
- **Return values**: `a0` holds error code, `a1` holds return value (if successful) / `a0` 返回错误码，`a1` 返回值（成功时）

**SBI Error Codes / SBI 错误码：**

| Code | Name | Description / 说明 |
|------|------|-------------------|
| 0 | `SBI_SUCCESS` | Call completed successfully / 调用成功完成 |
| -1 | `SBI_ERR_FAILED` | Call failed / 调用失败 |
| -2 | `SBI_ERR_NOT_SUPPORTED` | Function not supported / 功能不支持 |
| -3 | `SBI_ERR_INVALID_PARAM` | Invalid parameter / 参数无效 |
| -4 | `SBI_ERR_DENIED` | Permission denied / 权限被拒绝 |
| -5 | `SBI_ERR_INVALID_ADDRESS` | Invalid address / 地址无效 |
| -6 | `SBI_ERR_ALREADY_AVAILABLE` | Resource already available / 资源已可用 |
| -7 | `SBI_ERR_ALREADY_STARTED` | Already started / 已启动 |
| -8 | `SBI_ERR_ALREADY_STOPPED` | Already stopped / 已停止 |
| -9 | `SBI_ERR_NO_SHMEM` | Shared memory not available / 共享内存不可用 |

---

### SBI Extension IDs / SBI 扩展 ID

**Base Extensions (EID 0x00–0x0F) / 基础扩展：**

| EID | Extension Name / 扩展名 | Minimum Version / 最低版本 | Description / 说明 |
|-----|------------------------|---------------------------|-------------------|
| 0x00 | Reserved / 保留 | — | — |
| 0x01 | `sbi_set_timer` (Legacy) | 0.1 | Legacy timer extension / 遗留定时器扩展 |
| 0x02 | `sbi_console_putchar` (Legacy) | 0.1 | Legacy console output / 遗留控制台输出 |
| 0x03 | `sbi_console_getchar` (Legacy) | 0.1 | Legacy console input / 遗留控制台输入 |
| 0x04 | `sbi_clear_ipi` (Legacy) | 0.1 | Legacy IPI clear / 遗留 IPI 清除 |
| 0x05 | `sbi_send_ipi` (Legacy) | 0.1 | Legacy IPI send / 遗留 IPI 发送 |
| 0x06 | `sbi_remote_fence_i` (Legacy) | 0.1 | Legacy remote FENCE.I / 遗留远程 FENCE.I |
| 0x07 | `sbi_remote_sfence_vma` (Legacy) | 0.1 | Legacy remote SFENCE.VMA / 遗留远程 SFENCE.VMA |
| 0x08 | `sbi_remote_sfence_vma_asid` (Legacy) | 0.1 | Legacy remote SFENCE.VMA with ASID / 带 ASID 的遗留远程 SFENCE.VMA |
| 0x09 | `sbi_shutdown` (Legacy) | 0.1 | Legacy system shutdown / 遗留系统关机 |
| 0x10 | **BASE** | 0.2 | Base extension / 基础扩展 |
| 0x54494D45 | **TIME** | 0.2 | Timer programming / 定时器编程 |
| 0x735049 | **sPI** (IPI) | 0.2 | Inter-processor interrupts / 处理器间中断 |
| 0x52464E43 | **RFNC** | 0.2 | Remote FENCE / 远程 FENCE |
| 0x48534D | **HSM** | 0.2 | Hart state management / hart 状态管理 |
| 0x53525354 | **SRST** | 0.3 | System reset / 系统复位 |
| 0x504D55 | **PMU** | 0.3 | Performance monitoring / 性能监控 |
| 0x44424347 | **DBCN** | 0.3 | Debug console / 调试控制台 |
| 0x53555350 | **SUSP** | 0.3 | System suspend / 系统挂起 |
| 0x4350424 | **CPPC** | 0.3 | Collaborative processor performance control / 协作处理器性能控制 |
| 0x4E41434C | **NACL** | 0.3 | Nested acceleration / 嵌套加速 |
| 0x535441 | **STA** | 0.3 | Steal-time accounting / 窃取时间统计 |

**Vendor Extensions (EID 0x09000000–0x09FFFFFF) / 供应商扩展：**
Reserved for vendor-specific extensions / 保留给供应商特定扩展

**Firmware Extensions (EID 0x0A000000–0x0AFFFFFF) / 固件扩展：**
Reserved for firmware-specific extensions / 保留给固件特定扩展

---

### SBI BASE Extension (EID 0x10) / SBI 基础扩展

Provides functions to query SBI implementation details and available extensions.

提供查询 SBI 实现细节和可用扩展的函数。

**Functions / 函数：**

| FID | Function Name / 函数名 | Parameters / 参数 | Returns / 返回值 | Description / 说明 |
|-----|----------------------|------------------|-----------------|-------------------|
| 0 | `sbi_get_spec_version` | — | `a1`: Spec version / 规范版本 | Returns SBI specification version (upper 24 bits: major, lower 7 bits: minor) / 返回 SBI 规范版本（高 24 位：主版本号，低 7 位：次版本号） |
| 1 | `sbi_get_impl_id` | — | `a1`: Implementation ID / 实现 ID | Returns implementation identifier (0=BBL, 1=OpenSBI, 2=Xvisor, etc.) / 返回实现标识符（0=BBL，1=OpenSBI，2=Xvisor 等） |
| 2 | `sbi_get_impl_version` | — | `a1`: Implementation version / 实现版本 | Returns implementation-specific version / 返回实现特定版本 |
| 3 | `sbi_probe_extension` | `a0`: Extension ID / 扩展 ID | `a1`: 0=unavailable, 1=available / 0=不可用，1=可用 | Checks if an extension is available / 检查扩展是否可用 |
| 4 | `sbi_get_mvendorid` | — | `a1`: `mvendorid` CSR value / `mvendorid` CSR 值 | Returns JEDEC manufacturer ID / 返回 JEDEC 制造商 ID |
| 5 | `sbi_get_marchid` | — | `a1`: `marchid` CSR value / `marchid` CSR 值 | Returns microarchitecture ID / 返回微架构 ID |
| 6 | `sbi_get_mimpid` | — | `a1`: `mimpid` CSR value / `mimpid` CSR 值 | Returns implementation ID / 返回实现 ID |

**Usage Example / 使用示例：**
```c
// Check SBI version / 检查 SBI 版本
struct sbiret ret = sbi_ecall(SBI_EXT_BASE, SBI_EXT_BASE_GET_SPEC_VERSION, 0, 0, 0, 0, 0, 0);
unsigned long major = ret.value >> 24;
unsigned long minor = ret.value & 0x7F;

// Probe TIME extension / 探测 TIME 扩展
ret = sbi_ecall(SBI_EXT_BASE, SBI_EXT_BASE_PROBE_EXT, SBI_EXT_TIME, 0, 0, 0, 0, 0);
if (ret.value == 1) {
    // TIME extension available / TIME 扩展可用
}
```

---

### SBI TIME Extension (EID 0x54494D45) / SBI 定时器扩展

Provides timer programming functionality, replacing the legacy `sbi_set_timer`.

提供定时器编程功能，替代遗留的 `sbi_set_timer`。

**Functions / 函数：**

| FID | Function Name / 函数名 | Parameters / 参数 | Returns / 返回值 | Description / 说明 |
|-----|----------------------|------------------|-----------------|-------------------|
| 0 | `sbi_set_timer` | `a0`: 64-bit timer value / 64 位定时器值 | `a0`: Error code / 错误码 | Programs the timer to trigger at absolute time `a0` (in ticks). Pass -1ULL to clear a pending timer interrupt. / 编程定时器在绝对时间 `a0`（以时钟周期为单位）触发。传递 -1ULL 清除挂起的定时器中断。 |

**Implementation Details / 实现细节：**
- On RV32, the 64-bit timer value is passed with low 32 bits in `a0` and high 32 bits in `a1` / 在 RV32 上，64 位定时器值的低 32 位通过 `a0` 传递，高 32 位通过 `a1` 传递
- Internally, the SBI implementation programs `mtimecmp` or uses Sstc's `stimecmp` / 内部实现通过编程 `mtimecmp` 或使用 Sstc 扩展的 `stimecmp`
- Timer interrupt is delivered as a supervisor timer interrupt (STIP) / 定时器中断作为监管者定时器中断（STIP）传递

**Usage Example / 使用示例：**
```c
// Set timer to fire in 1 second (assuming 10 MHz timebase) / 设置定时器在 1 秒后触发（假设 10 MHz 时基）
uint64_t next_tick = rdtime() + 10000000;
sbi_ecall(SBI_EXT_TIME, SBI_EXT_TIME_SET_TIMER, next_tick, 0, 0, 0, 0, 0);
```

---

### SBI IPI Extension (EID 0x735049) / SBI 处理器间中断扩展

Provides inter-processor interrupt (IPI) send functionality, replacing legacy `sbi_send_ipi`.

提供处理器间中断（IPI）发送功能，替代遗留的 `sbi_send_ipi`。

**Functions / 函数：**

| FID | Function Name / 函数名 | Parameters / 参数 | Returns / 返回值 | Description / 说明 |
|-----|----------------------|------------------|-----------------|-------------------|
| 0 | `sbi_send_ipi` | `a0`: Hart mask (unsigned long) / Hart 掩码<br>`a1`: Hart mask base (unsigned long) / Hart 掩码基址 | `a0`: Error code / 错误码 | Sends an IPI to all harts specified by the mask. Hart ID `i` is included if `(hart_mask >> (i - hart_mask_base)) & 1`. / 向掩码指定的所有 hart 发送 IPI。Hart ID `i` 包含在内如果 `(hart_mask >> (i - hart_mask_base)) & 1`。 |

**Hart Mask Encoding / Hart 掩码编码：**
- `hart_mask_base`: Starting hart ID for the mask / 掩码的起始 hart ID
- `hart_mask`: Bitmap of target harts relative to the base / 相对于基址的目标 hart 位图
- Example: To send IPI to harts 3, 5, 6, use `hart_mask_base=3, hart_mask=0b1101` (binary: bit 0=hart 3, bit 2=hart 5, bit 3=hart 6) / 示例：向 hart 3、5、6 发送 IPI，使用 `hart_mask_base=3, hart_mask=0b1101`（二进制：位 0=hart 3，位 2=hart 5，位 3=hart 6）

**Usage Example / 使用示例：**
```c
// Send IPI to all harts (0-63) / 向所有 hart（0-63）发送 IPI
sbi_ecall(SBI_EXT_IPI, SBI_EXT_IPI_SEND_IPI, -1UL, 0, 0, 0, 0, 0);

// Send IPI to harts 4, 5, 7 / 向 hart 4、5、7 发送 IPI
unsigned long mask = (1 << 0) | (1 << 1) | (1 << 3);  // bits 0,1,3
sbi_ecall(SBI_EXT_IPI, SBI_EXT_IPI_SEND_IPI, mask, 4, 0, 0, 0, 0);
```

---

### SBI RFNC Extension (EID 0x52464E43) / SBI 远程 FENCE 扩展

Provides remote FENCE operations to synchronize instruction and data caches across multiple harts, replacing legacy remote FENCE functions.

提供远程 FENCE 操作以在多个 hart 间同步指令和数据缓存，替代遗留的远程 FENCE 函数。

**Functions / 函数：**

| FID | Function Name / 函数名 | Parameters / 参数 | Returns / 返回值 | Description / 说明 |
|-----|----------------------|------------------|-----------------|-------------------|
| 0 | `sbi_remote_fence_i` | `a0`: Hart mask / Hart 掩码<br>`a1`: Hart mask base / Hart 掩码基址 | `a0`: Error code / 错误码 | Executes FENCE.I on remote harts to synchronize instruction caches. / 在远程 hart 上执行 FENCE.I 以同步指令缓存。 |
| 1 | `sbi_remote_sfence_vma` | `a0`: Hart mask / Hart 掩码<br>`a1`: Hart mask base / Hart 掩码基址<br>`a2`: Start address / 起始地址<br>`a3`: Size / 大小 | `a0`: Error code / 错误码 | Executes SFENCE.VMA on remote harts for the specified address range. Pass `start_addr=0, size=0` for global TLB flush. / 在远程 hart 上为指定地址范围执行 SFENCE.VMA。传递 `start_addr=0, size=0` 进行全局 TLB 刷新。 |
| 2 | `sbi_remote_sfence_vma_asid` | `a0`: Hart mask / Hart 掩码<br>`a1`: Hart mask base / Hart 掩码基址<br>`a2`: Start address / 起始地址<br>`a3`: Size / 大小<br>`a4`: ASID | `a0`: Error code / 错误码 | Executes SFENCE.VMA with ASID on remote harts. / 在远程 hart 上执行带 ASID 的 SFENCE.VMA。 |
| 3 | `sbi_remote_hfence_gvma_vmid` | `a0`: Hart mask<br>`a1`: Hart mask base<br>`a2`: Start GPA<br>`a3`: Size<br>`a4`: VMID | `a0`: Error code | Executes HFENCE.GVMA with VMID on remote harts (H extension). / 在远程 hart 上执行带 VMID 的 HFENCE.GVMA（H 扩展）。 |
| 4 | `sbi_remote_hfence_gvma` | `a0`: Hart mask<br>`a1`: Hart mask base<br>`a2`: Start GPA<br>`a3`: Size | `a0`: Error code | Executes HFENCE.GVMA on remote harts. / 在远程 hart 上执行 HFENCE.GVMA。 |
| 5 | `sbi_remote_hfence_vvma_asid` | `a0`: Hart mask<br>`a1`: Hart mask base<br>`a2`: Start GVA<br>`a3`: Size<br>`a4`: ASID | `a0`: Error code | Executes HFENCE.VVMA with ASID on remote harts (H extension). / 在远程 hart 上执行带 ASID 的 HFENCE.VVMA（H 扩展）。 |
| 6 | `sbi_remote_hfence_vvma` | `a0`: Hart mask<br>`a1`: Hart mask base<br>`a2`: Start GVA<br>`a3`: Size | `a0`: Error code | Executes HFENCE.VVMA on remote harts. / 在远程 hart 上执行 HFENCE.VVMA。 |

**When to Use / 何时使用：**
- After modifying page tables on one hart, use `sbi_remote_sfence_vma` to invalidate TLB on other harts / 在一个 hart 上修改页表后，使用 `sbi_remote_sfence_vma` 使其他 hart 的 TLB 失效
- After writing new code to memory, use `sbi_remote_fence_i` to ensure all harts see the updated instructions / 写入新代码到内存后，使用 `sbi_remote_fence_i` 确保所有 hart 看到更新的指令
- For hypervisor operations, use `sbi_remote_hfence_*` variants to manage guest TLBs / 对于虚拟化操作，使用 `sbi_remote_hfence_*` 变体管理客户 TLB

---

### SBI HSM Extension (EID 0x48534D) / SBI Hart 状态管理扩展

Provides hart lifecycle management, allowing S-mode to start, stop, and query hart states.

提供 hart 生命周期管理，允许 S 模式启动、停止和查询 hart 状态。

**Hart States / Hart 状态：**

| State / 状态 | Value / 值 | Description / 说明 |
|-------------|-----------|-------------------|
| STARTED | 0 | Hart is running in S-mode or U-mode / Hart 正在 S 模式或 U 模式下运行 |
| STOPPED | 1 | Hart is not executing, powered down or in WFI / Hart 未执行，已关闭电源或处于 WFI |
| START_PENDING | 2 | Hart start request is pending / Hart 启动请求挂起 |
| STOP_PENDING | 3 | Hart stop request is pending / Hart 停止请求挂起 |
| SUSPENDED | 4 | Hart is suspended (low-power state) / Hart 已挂起（低功耗状态）|
| SUSPEND_PENDING | 5 | Hart suspend request is pending / Hart 挂起请求挂起 |
| RESUME_PENDING | 6 | Hart resume request is pending / Hart 恢复请求挂起 |

**Functions / 函数：**

| FID | Function Name / 函数名 | Parameters / 参数 | Returns / 返回值 | Description / 说明 |
|-----|----------------------|------------------|-----------------|-------------------|
| 0 | `sbi_hart_start` | `a0`: Target hart ID / 目标 hart ID<br>`a1`: Start address / 起始地址<br>`a2`: Private data (opaque) / 私有数据（不透明）| `a0`: Error code / 错误码 | Starts the target hart at the specified address in S-mode with `a1=opaque`. The target hart must be in STOPPED state. / 在指定地址以 S 模式启动目标 hart，`a1` 传递 opaque。目标 hart 必须处于 STOPPED 状态。 |
| 1 | `sbi_hart_stop` | — | Does not return / 不返回 | Stops the calling hart and puts it in STOPPED state. This call does not return. / 停止调用的 hart 并将其置于 STOPPED 状态。此调用不返回。 |
| 2 | `sbi_hart_get_status` | `a0`: Target hart ID / 目标 hart ID | `a0`: Error code / 错误码<br>`a1`: Hart state / Hart 状态 | Returns the current state of the target hart. / 返回目标 hart 的当前状态。 |
| 3 | `sbi_hart_suspend` | `a0`: Suspend type / 挂起类型<br>`a1`: Resume address / 恢复地址<br>`a2`: Private data / 私有数据 | `a0`: Error code / 错误码 | Suspends the calling hart. Type 0=retentive suspend (state preserved), 0x80000000=non-retentive (state lost). / 挂起调用的 hart。类型 0=保持挂起（状态保留），0x80000000=非保持（状态丢失）。 |

**Usage Example / 使用示例：**
```c
// Start hart 1 at kernel_secondary_start_sbi / 在 kernel_secondary_start_sbi 启动 hart 1
extern void kernel_secondary_start_sbi(unsigned long hartid, unsigned long opaque);
sbi_ecall(SBI_EXT_HSM, SBI_EXT_HSM_HART_START, 
          1, (unsigned long)kernel_secondary_start_sbi, 0, 0, 0, 0);

// Suspend current hart (retentive) / 挂起当前 hart（保持状态）
sbi_ecall(SBI_EXT_HSM, SBI_EXT_HSM_HART_SUSPEND, 
          0, (unsigned long)resume_address, 0, 0, 0, 0);
```

---

### SBI SRST Extension (EID 0x53525354) / SBI 系统复位扩展

Provides system reset and shutdown functionality, replacing legacy `sbi_shutdown`.

提供系统复位和关机功能，替代遗留的 `sbi_shutdown`。

**Reset Types / 复位类型：**

| Type / 类型 | Value / 值 | Description / 说明 |
|------------|-----------|-------------------|
| SHUTDOWN | 0x00000000 | Shutdown / 关机 |
| COLD_REBOOT | 0x00000001 | Cold reboot (power cycle) / 冷重启（电源循环）|
| WARM_REBOOT | 0x00000002 | Warm reboot (no power cycle) / 热重启（无电源循环）|

**Reset Reasons / 复位原因：**

| Reason / 原因 | Value / 值 | Description / 说明 |
|--------------|-----------|-------------------|
| NO_REASON | 0x00000000 | No specific reason / 无特定原因 |
| SYSTEM_FAILURE | 0x00000001 | System failure / 系统故障 |

**Functions / 函数：**

| FID | Function Name / 函数名 | Parameters / 参数 | Returns / 返回值 | Description / 说明 |
|-----|----------------------|------------------|-----------------|-------------------|
| 0 | `sbi_system_reset` | `a0`: Reset type / 复位类型<br>`a1`: Reset reason / 复位原因 | Does not return / 不返回 | Resets or powers off the system. This call does not return on success. / 复位或关闭系统。成功时此调用不返回。 |

**Usage Example / 使用示例：**
```c
// Shutdown system / 关机系统
sbi_ecall(SBI_EXT_SRST, SBI_EXT_SRST_RESET, SBI_SRST_TYPE_SHUTDOWN, SBI_SRST_REASON_NONE, 0, 0, 0, 0);

// Cold reboot / 冷重启
sbi_ecall(SBI_EXT_SRST, SBI_EXT_SRST_RESET, SBI_SRST_TYPE_COLD_REBOOT, SBI_SRST_REASON_NONE, 0, 0, 0, 0);
```

---

### SBI PMU Extension (EID 0x504D55) / SBI 性能监控扩展

Provides access to hardware performance counters (HPM), allowing S-mode to configure and read performance monitoring events.

提供访问硬件性能计数器（HPM）的能力，允许 S 模式配置和读取性能监控事件。

**Counter Types / 计数器类型：**

| Type / 类型 | Description / 说明 |
|------------|-------------------|
| FIRMWARE | Firmware counter / 固件计数器 |
| HARDWARE | Hardware counter / 硬件计数器 |
| CACHE | Cache event counter / 缓存事件计数器 |
| CYCLE | Cycle counter / 周期计数器 |
| INSTRUCTION | Instruction counter / 指令计数器 |

**Functions / 函数：**

| FID | Function Name / 函数名 | Parameters / 参数 | Returns / 返回值 | Description / 说明 |
|-----|----------------------|------------------|-----------------|-------------------|
| 0 | `sbi_pmu_num_counters` | — | `a0`: Error code<br>`a1`: Number of counters | Returns total number of performance counters / 返回性能计数器总数 |
| 1 | `sbi_pmu_counter_get_info` | `a0`: Counter index | `a0`: Error code<br>`a1`: Counter info | Returns counter type and width / 返回计数器类型和宽度 |
| 2 | `sbi_pmu_counter_config_matching` | `a0`: Counter index base<br>`a1`: Counter mask<br>`a2`: Config flags<br>`a3`: Event index<br>`a4`: Event data | `a0`: Error code<br>`a1`: Counter index | Configures matching counters for the specified event / 为指定事件配置匹配的计数器 |
| 3 | `sbi_pmu_counter_start` | `a0`: Counter index base<br>`a1`: Counter mask<br>`a2`: Start flags<br>`a3`: Initial value | `a0`: Error code | Starts the specified counters / 启动指定的计数器 |
| 4 | `sbi_pmu_counter_stop` | `a0`: Counter index base<br>`a1`: Counter mask<br>`a2`: Stop flags | `a0`: Error code | Stops the specified counters / 停止指定的计数器 |
| 5 | `sbi_pmu_counter_fw_read` | `a0`: Counter index | `a0`: Error code<br>`a1`: Counter value | Reads firmware counter value / 读取固件计数器值 |
| 6 | `sbi_pmu_counter_fw_read_hi` | `a0`: Counter index | `a0`: Error code<br>`a1`: High 32 bits | Reads high 32 bits (RV32 only) / 读取高 32 位（仅 RV32）|

**Common Events / 常见事件：**
- **CPU cycles**: Event index 1 / CPU 周期：事件索引 1
- **Instructions retired**: Event index 2 / 已执行指令：事件索引 2
- **Cache misses**: Event index 3 / 缓存未命中：事件索引 3
- **Branch mispredictions**: Event index 4 / 分支预测失败：事件索引 4

**Usage Example / 使用示例：**
```c
// Get number of counters / 获取计数器数量
struct sbiret ret = sbi_ecall(SBI_EXT_PMU, SBI_EXT_PMU_NUM_COUNTERS, 0, 0, 0, 0, 0, 0);
unsigned long num_counters = ret.value;

// Configure counter 3 for cache miss events / 为缓存未命中事件配置计数器 3
sbi_ecall(SBI_EXT_PMU, SBI_EXT_PMU_COUNTER_CFG_MATCH, 3, 1UL << 3, 0, 3, 0, 0);

// Start counter / 启动计数器
sbi_ecall(SBI_EXT_PMU, SBI_EXT_PMU_COUNTER_START, 3, 1UL << 3, 0, 0, 0, 0);
```

---

### SBI DBCN Extension (EID 0x44424347) / SBI 调试控制台扩展

Provides debug console I/O, replacing legacy console functions with better performance and buffer support.

提供调试控制台 I/O，用更好的性能和缓冲区支持替代遗留控制台函数。

**Functions / 函数：**

| FID | Function Name / 函数名 | Parameters / 参数 | Returns / 返回值 | Description / 说明 |
|-----|----------------------|------------------|-----------------|-------------------|
| 0 | `sbi_debug_console_write` | `a0`: Number of bytes<br>`a1`: Physical address base (low)<br>`a2`: Physical address base (high) | `a0`: Error code<br>`a1`: Bytes written | Writes bytes from physical memory to debug console / 从物理内存向调试控制台写入字节 |
| 1 | `sbi_debug_console_read` | `a0`: Number of bytes<br>`a1`: Physical address base (low)<br>`a2`: Physical address base (high) | `a0`: Error code<br>`a1`: Bytes read | Reads bytes from debug console to physical memory / 从调试控制台读取字节到物理内存 |
| 2 | `sbi_debug_console_write_byte` | `a0`: Byte to write | `a0`: Error code | Writes a single byte / 写入单个字节 |

**Usage Example / 使用示例：**
```c
// Write string to debug console / 向调试控制台写入字符串
const char *msg = "Hello from S-mode\n";
unsigned long phys_addr = virt_to_phys(msg);
sbi_ecall(SBI_EXT_DBCN, SBI_EXT_DBCN_CONSOLE_WRITE, 
          strlen(msg), phys_addr, 0, 0, 0, 0);
```

---

### SBI SUSP Extension (EID 0x53555350) / SBI 系统挂起扩展

Provides system-wide suspend functionality for power management.

提供系统级挂起功能用于电源管理。

**Suspend Types / 挂起类型：**

| Type / 类型 | Value / 值 | Description / 说明 |
|------------|-----------|-------------------|
| SUSPEND_TO_RAM | 0x00000000 | Suspend to RAM (S3-like) / 挂起到 RAM（类似 S3）|

**Functions / 函数：**

| FID | Function Name / 函数名 | Parameters / 参数 | Returns / 返回值 | Description / 说明 |
|-----|----------------------|------------------|-----------------|-------------------|
| 0 | `sbi_system_suspend` | `a0`: Suspend type<br>`a1`: Resume address (low)<br>`a2`: Resume address (high) | `a0`: Error code | Suspends the system to the specified low-power state / 将系统挂起到指定的低功耗状态 |

---

### SBI CPPC Extension (EID 0x4350424) / SBI 协作处理器性能控制扩展

Implements ACPI CPPC (Collaborative Processor Performance Control) for dynamic frequency and voltage scaling.

实现 ACPI CPPC（协作处理器性能控制）用于动态频率和电压调节。

**Functions / 函数：**

| FID | Function Name / 函数名 | Parameters / 参数 | Returns / 返回值 | Description / 说明 |
|-----|----------------------|------------------|-----------------|-------------------|
| 0 | `sbi_cppc_probe` | `a0`: Register ID | `a0`: Error code<br>`a1`: Register supported | Probes if a CPPC register is supported / 探测 CPPC 寄存器是否支持 |
| 1 | `sbi_cppc_read` | `a0`: Register ID | `a0`: Error code<br>`a1`: Register value (low)<br>`a2`: Register value (high) | Reads a CPPC register / 读取 CPPC 寄存器 |
| 2 | `sbi_cppc_read_hi` | `a0`: Register ID | `a0`: Error code<br>`a1`: High 32 bits | Reads high 32 bits (RV32 only) / 读取高 32 位（仅 RV32）|
| 3 | `sbi_cppc_write` | `a0`: Register ID<br>`a1`: Value (low)<br>`a2`: Value (high) | `a0`: Error code | Writes a CPPC register / 写入 CPPC 寄存器 |

**Common CPPC Registers / 常见 CPPC 寄存器：**
- **Highest Performance**: Maximum supported performance level / 最高性能级别
- **Nominal Performance**: Guaranteed performance level / 标称性能级别
- **Lowest Performance**: Minimum performance level / 最低性能级别
- **Desired Performance**: OS-requested performance level / OS 请求的性能级别

---

### SBI NACL Extension (EID 0x4E41434C) / SBI 嵌套加速扩展

Enables nested virtualization acceleration by allowing a guest hypervisor to use hardware virtualization features.

通过允许客户虚拟化监控程序使用硬件虚拟化功能来启用嵌套虚拟化加速。

**Functions / 函数：**

| FID | Function Name / 函数名 | Parameters / 参数 | Returns / 返回值 | Description / 说明 |
|-----|----------------------|------------------|-----------------|-------------------|
| 0 | `sbi_nacl_probe_feature` | `a0`: Feature ID | `a0`: Error code<br>`a1`: Feature value | Probes nested acceleration features / 探测嵌套加速功能 |
| 1 | `sbi_nacl_set_shmem` | `a0`: Shared memory address (low)<br>`a1`: Shared memory address (high)<br>`a2`: Flags | `a0`: Error code | Sets shared memory region for nested acceleration / 为嵌套加速设置共享内存区域 |
| 2 | `sbi_nacl_sync_csr` | `a0`: CSR number | `a0`: Error code | Synchronizes a VS-mode CSR with the nested guest / 与嵌套客户同步 VS 模式 CSR |
| 3 | `sbi_nacl_sync_hfence` | `a0`: HFENCE type | `a0`: Error code | Synchronizes HFENCE operations for nested guests / 为嵌套客户同步 HFENCE 操作 |

---

### SBI STA Extension (EID 0x535441) / SBI 窃取时间统计扩展

Tracks steal time for guest VMs (time when a VCPU is runnable but not running due to hypervisor scheduling).

跟踪客户虚拟机的窃取时间（VCPU 可运行但由于虚拟化监控程序调度而未运行的时间）。

**Functions / 函数：**

| FID | Function Name / 函数名 | Parameters / 参数 | Returns / 返回值 | Description / 说明 |
|-----|----------------------|------------------|-----------------|-------------------|
| 0 | `sbi_sta_set_shmem` | `a0`: Shared memory address (low)<br>`a1`: Shared memory address (high)<br>`a2`: Flags | `a0`: Error code | Sets shared memory region for steal-time accounting / 为窃取时间统计设置共享内存区域 |

**Shared Memory Layout / 共享内存布局：**
```c
struct sbi_sta_struct {
    uint32_t sequence;      // Sequence number for consistency / 一致性序列号
    uint32_t flags;         // Flags / 标志
    uint64_t steal;         // Accumulated steal time in nanoseconds / 累计窃取时间（纳秒）
};
```

**Usage Example / 使用示例：**
```c
// Set up steal-time tracking / 设置窃取时间跟踪
struct sbi_sta_struct *sta_region = alloc_shared_mem();
unsigned long phys_addr = virt_to_phys(sta_region);
sbi_ecall(SBI_EXT_STA, SBI_EXT_STA_SET_SHMEM, phys_addr, 0, 0, 0, 0, 0);

// Read steal time / 读取窃取时间
uint64_t steal_time_ns = sta_region->steal;
```
**SBI Calling Convention / SBI 调用约定：**
- `a7` = SBI Extension ID (EID) / SBI 扩展 ID（Extension ID，EID）
- `a6` = SBI Function ID (FID) / SBI 函数 ID（Function ID，FID）
- `a0–a5` = Arguments / 参数
- `a0` = Error code return / 错误码返回，`a1` = Value return / 值返回

### SBI Extension / SBI 扩展

A functionally-grouped set of interfaces in the SBI specification, each extension identified by a unique EID.

SBI 规范中按功能分组的接口集合，每个扩展有唯一 EID：

| EID | Extension Name / 扩展名 | Function / 功能 |
|-----|--------|------|
| 0x10 | Base | Query SBI version/implementation/extension support / 查询 SBI 版本/实现/扩展是否支持 |
| 0x54494D45 | TIME | Set timer (replaces direct `mtimecmp` access) / 设置定时器（替代 `mtimecmp` 直接访问） |
| 0x735049 | sPI | Send software interrupt to other harts / 发送软件中断给其他 hart |
| 0x48534D | HSM | Hart State Management (hart start/stop/suspend) / Hart 状态管理（hart 启动/停止/挂起） |
| 0x53525354 | SRST | System reset/shutdown / 系统复位/关机 |
| 0x504D55 | PMU | Performance monitoring unit access / 性能监控单元访问 |
| 0x52464E43 | RFNC | Remote Fence (cross-hart TLB flush) / 远程屏障（Remote Fence，跨 hart 刷新 TLB） |
| 0x4442434E | DBCN | Debug console (simple character output) / 调试控制台（简单字符输出） |
| 0x53555350 | SUSP | System suspend / 系统挂起（suspend） |

### SBI HSM (Hart State Management) / Hart 状态管理

Manages the lifecycle of harts in multi-core systems: `sbi_hart_start` (start secondary cores), `sbi_hart_stop` (stop current core), `sbi_hart_suspend` (suspend, low power). Linux SMP boot and CPU hotplug rely on this interface.

管理多核系统中各 hart 的生命周期：`sbi_hart_start`（启动辅助核）、`sbi_hart_stop`（停止当前核）、`sbi_hart_suspend`（挂起，低功耗）。Linux 的 SMP 启动和 CPU 热插拔依赖此接口。

### SBI RFNC (Remote Fence) / 远程屏障

Sends TLB flush requests across harts: `sbi_remote_sfence_vma` (flush TLB of specified hart set), `sbi_remote_fence_i` (flush I-cache). Must be called after OS kernel modifies page tables to ensure TLB consistency across all harts.

跨 hart 发送 TLB 刷新请求：`sbi_remote_sfence_vma`（刷新指定 hart 集合的 TLB）、`sbi_remote_fence_i`（刷新 I-cache）。OS 内核修改页表后需调用，确保所有 hart 的 TLB 一致。

### OpenSBI / OpenSBI

The official reference SBI firmware implementation for RISC-V (open source, written in C), runs in M-mode, and supports all mainstream SBI extensions. Adopted by most RISC-V platforms (QEMU, SiFive, StarFive, T-Head, etc.). Project URL: https://github.com/riscv-software-src/opensbi

RISC-V 官方参考 SBI 固件实现（开源，C 语言），运行在 M 模式，支持所有主流 SBI 扩展。被大多数 RISC-V 平台（QEMU、SiFive、StarFive、T-Head 等）采用。项目地址：https://github.com/riscv-software-src/opensbi

---

## Interrupt Controller / 中断控制器（Interrupt Controller）

### PLIC (Platform Level Interrupt Controller) / 平台级中断控制器

The traditional external interrupt router for RISC-V platforms, routes interrupts from peripherals (up to 1023 interrupt sources) to each hart's M/S mode external interrupt. Supports priority and threshold, uses claim/complete mechanism for interrupt acknowledgment.

RISC-V 平台的传统外部中断路由器，将来自外设的中断（最多 1023 个中断源）路由到各 hart 的 M/S 模式外部中断。支持优先级和阈值，通过 claim/complete 机制确认中断。

**PLIC Workflow / PLIC 工作流程：**
```
外设触发中断 → PLIC 判断优先级 > 目标hart阈值？
→ 是 → 设置 mip.MEIP/mip.SEIP
→ hart 进入中断处理 → 读 PLIC claim 寄存器（获取中断号）
→ 处理完毕 → 写 PLIC complete 寄存器（释放中断）
```

---

## Interrupt Controllers / 中断控制器

### PLIC (Platform-Level Interrupt Controller) / 平台级中断控制器

The standard interrupt controller for RISC-V systems, manages external interrupt sources and routes them to harts. PLIC is widely deployed in current RISC-V SoCs.

RISC-V 系统的标准中断控制器，管理外部中断源并将其路由到 hart。PLIC 在当前 RISC-V SoC 中广泛部署。

**PLIC Architecture / PLIC 架构：**

```
External Interrupt Sources (1-1023)
          ↓
    [PLIC Gateway]
          ↓
    [Priority & Pending]
          ↓
    [Context/Target]
          ↓
Hart 0-N (S-mode/M-mode)
```

**Key Concepts / 关键概念：**

| Concept / 概念 | Description / 说明 |
|---------------|-------------------|
| **Interrupt Source** | External device interrupt, numbered 1-1023 (0 reserved) / 外部设备中断，编号 1-1023（0 保留）|
| **Gateway** | Converts level/edge interrupts to pending state / 将电平/边沿中断转换为挂起状态 |
| **Priority** | Each source has a priority (0-7 typical, 0=never interrupt) / 每个源有优先级（典型 0-7，0=永不中断）|
| **Threshold** | Per-context minimum priority to deliver interrupt / 每个上下文的最小交付优先级 |
| **Context** | A hart's interrupt delivery target (M-mode or S-mode) / Hart 的中断交付目标（M 模式或 S 模式）|
| **Claim** | Read to acknowledge and get interrupt ID / 读取以确认并获取中断 ID |
| **Complete** | Write to signal interrupt handling finished / 写入以表示中断处理完成 |

**PLIC MMIO Register Layout / PLIC MMIO 寄存器布局：**

| Offset / 偏移 | Register / 寄存器 | Description / 说明 |
|--------------|------------------|-------------------|
| 0x000000 | Source Priority | Priority for interrupt source 1-1023 / 中断源 1-1023 的优先级 |
| 0x001000 | Pending Bits | Bitmap of pending interrupts / 挂起中断的位图 |
| 0x002000 | Enable Bits (Context 0) | Enable bits for first context / 第一个上下文的使能位 |
| 0x002080 | Enable Bits (Context 1) | Enable bits for second context / 第二个上下文的使能位 |
| ... | ... | Enable bits for additional contexts / 额外上下文的使能位 |
| 0x200000 | Priority Threshold (Context 0) | Threshold for first context / 第一个上下文的阈值 |
| 0x200004 | Claim/Complete (Context 0) | Claim/Complete for first context / 第一个上下文的 Claim/Complete |
| 0x201000 | Priority Threshold (Context 1) | Threshold for second context / 第二个上下文的阈值 |
| 0x201004 | Claim/Complete (Context 1) | Claim/Complete for second context / 第二个上下文的 Claim/Complete |

**Context Numbering / 上下文编号：**
- Context = `hart_id * 2 + mode` where mode: 0=M-mode, 1=S-mode
- Example: Hart 0 M-mode = context 0, Hart 0 S-mode = context 1, Hart 1 M-mode = context 2
- 上下文 = `hart_id * 2 + 模式`，其中模式：0=M 模式，1=S 模式
- 示例：Hart 0 M 模式 = 上下文 0，Hart 0 S 模式 = 上下文 1，Hart 1 M 模式 = 上下文 2

**PLIC Interrupt Flow / PLIC 中断流程：**

1. **Device asserts interrupt** / 设备断言中断
2. **Gateway latches interrupt in pending register** / 网关在挂起寄存器中锁存中断
3. **PLIC checks priority and threshold** / PLIC 检查优先级和阈值
4. **If priority > threshold, assert interrupt to hart** / 如果优先级 > 阈值，向 hart 断言中断
5. **Hart enters trap handler, reads Claim register** / Hart 进入陷阱处理程序，读取 Claim 寄存器
6. **PLIC returns interrupt ID and clears pending** / PLIC 返回中断 ID 并清除挂起
7. **Hart handles interrupt** / Hart 处理中断
8. **Hart writes interrupt ID to Complete register** / Hart 将中断 ID 写入 Complete 寄存器
9. **PLIC marks interrupt as completed** / PLIC 标记中断为已完成

**Usage Example / 使用示例：**
```c
// Initialize PLIC / 初始化 PLIC
void plic_init(void) {
    // Set priority for interrupt source 10 / 设置中断源 10 的优先级
    PLIC_PRIORITY(10) = 7;
    
    // Enable interrupt 10 for hart 0 S-mode (context 1) / 为 hart 0 S 模式（上下文 1）启用中断 10
    PLIC_ENABLE(1, 10) = 1;
    
    // Set threshold to 0 (accept all priorities) / 设置阈值为 0（接受所有优先级）
    PLIC_THRESHOLD(1) = 0;
}

// Interrupt handler / 中断处理程序
void plic_handler(void) {
    // Claim interrupt / 声明中断
    uint32_t irq = PLIC_CLAIM(1);
    
    // Handle interrupt / 处理中断
    handle_device_irq(irq);
    
    // Complete interrupt / 完成中断
    PLIC_COMPLETE(1) = irq;
}
```

**PLIC Limitations / PLIC 局限性：**
- No native MSI support / 无原生 MSI 支持
- Limited scalability (max ~1023 sources) / 可扩展性有限（最多约 1023 个源）
- No hardware virtualization support / 无硬件虚拟化支持
- Requires software arbitration for simultaneous interrupts / 同时中断需要软件仲裁

---

### ACLINT (Advanced Core Local Interruptor) / 高级核本地中断器

A standardized specification replacing the SiFive CLINT, provides hart-local interrupt sources (timer and software interrupts) without going through PLIC. ACLINT is part of the RISC-V platform specification.

替代 SiFive CLINT 的标准化规范，提供 hart 本地中断源（定时器和软件中断），无需经过 PLIC。ACLINT 是 RISC-V 平台规范的一部分。

**ACLINT Components / ACLINT 组件：**

#### 1. **MTIMER (Machine Timer)** / M 模式定时器

Provides per-hart timers for M-mode timer interrupts (MTIP).

为每个 hart 提供定时器用于 M 模式定时器中断（MTIP）。

**MMIO Register Layout / MMIO 寄存器布局：**

| Offset / 偏移 | Register / 寄存器 | Size / 大小 | Description / 说明 |
|--------------|------------------|------------|-------------------|
| 0x0000 | `mtime` | 64-bit | Current time value (read-only) / 当前时间值（只读）|
| 0x0000 | `mtimecmp[0]` | 64-bit | Timer compare for hart 0 / Hart 0 的定时器比较值 |
| 0x0008 | `mtimecmp[1]` | 64-bit | Timer compare for hart 1 / Hart 1 的定时器比较值 |
| ... | `mtimecmp[N]` | 64-bit | Timer compare for hart N / Hart N 的定时器比较值 |

**Behavior / 行为：**
- When `mtime >= mtimecmp[i]`, assert MTIP for hart `i` / 当 `mtime >= mtimecmp[i]` 时，为 hart `i` 断言 MTIP
- Writing `mtimecmp[i]` clears the pending MTIP / 写入 `mtimecmp[i]` 清除挂起的 MTIP
- `mtime` increments at a fixed frequency (typically 1-10 MHz) / `mtime` 以固定频率递增（通常 1-10 MHz）

**Timebase Frequency / 时基频率：**
- Specified in device tree `timebase-frequency` property / 在设备树 `timebase-frequency` 属性中指定
- Common values: 1 MHz, 10 MHz, 32.768 kHz / 常见值：1 MHz、10 MHz、32.768 kHz

#### 2. **MSWI (Machine Software Interrupt)** / M 模式软件中断

Provides per-hart software-triggered interrupts for inter-processor interrupts (IPI).

为每个 hart 提供软件触发的中断用于处理器间中断（IPI）。

**MMIO Register Layout / MMIO 寄存器布局：**

| Offset / 偏移 | Register / 寄存器 | Size / 大小 | Description / 说明 |
|--------------|------------------|------------|-------------------|
| 0x0000 | `msip[0]` | 32-bit | Software interrupt for hart 0 / Hart 0 的软件中断 |
| 0x0004 | `msip[1]` | 32-bit | Software interrupt for hart 1 / Hart 1 的软件中断 |
| ... | `msip[N]` | 32-bit | Software interrupt for hart N / Hart N 的软件中断 |

**Behavior / 行为：**
- Write 1 to `msip[i]` to assert MSIP for hart `i` / 写 1 到 `msip[i]` 以为 hart `i` 断言 MSIP
- Write 0 to `msip[i]` to clear MSIP / 写 0 到 `msip[i]` 以清除 MSIP
- Used for IPI (inter-processor interrupt) / 用于 IPI（处理器间中断）

**IPI Example / IPI 示例：**
```c
// Send IPI to hart 1 / 向 hart 1 发送 IPI
void send_ipi(int target_hart) {
    ACLINT_MSIP(target_hart) = 1;
}

// Clear IPI / 清除 IPI
void clear_ipi(void) {
    ACLINT_MSIP(current_hartid()) = 0;
}
```

#### 3. **SSWI (Supervisor Software Interrupt)** / S 模式软件中断

Optional component providing S-mode software interrupts (SSIP), allowing S-mode to trigger software interrupts without M-mode intervention.

可选组件，提供 S 模式软件中断（SSIP），允许 S 模式在没有 M 模式干预的情况下触发软件中断。

**MMIO Register Layout / MMIO 寄存器布局：**

| Offset / 偏移 | Register / 寄存器 | Size / 大小 | Description / 说明 |
|--------------|------------------|------------|-------------------|
| 0x0000 | `setsip[0]` | 32-bit | Set SSIP for hart 0 / 为 hart 0 设置 SSIP |
| 0x0004 | `setsip[1]` | 32-bit | Set SSIP for hart 1 / 为 hart 1 设置 SSIP |

**Behavior / 行为：**
- Write 1 to `setsip[i]` to set SSIP for hart `i` / 写 1 到 `setsip[i]` 以为 hart `i` 设置 SSIP
- Writing to `sip` CSR clears SSIP / 写入 `sip` CSR 清除 SSIP
- Enables S-mode IPI without SBI overhead / 无需 SBI 开销即可实现 S 模式 IPI

**ACLINT vs SiFive CLINT / ACLINT 与 SiFive CLINT 对比：**

| Feature / 特性 | SiFive CLINT | ACLINT |
|---------------|--------------|--------|
| Standardization / 标准化 | Vendor-specific / 供应商特定 | RISC-V official / RISC-V 官方 |
| MTIMER | ✅ | ✅ |
| MSWI | ✅ | ✅ |
| SSWI | ❌ | ✅ (optional / 可选) |
| Memory layout / 内存布局 | Fixed / 固定 | Flexible / 灵活 |
| Device tree binding / 设备树绑定 | "riscv,clint0" | "riscv,aclint-mtimer", "riscv,aclint-mswi" |

**Device Tree Example / 设备树示例：**
```dts
aclint_mtimer: timer@2000000 {
    compatible = "riscv,aclint-mtimer";
    reg = <0x2000000 0x10000>;
    interrupts-extended = <&cpu0_intc 7>, <&cpu1_intc 7>;
};

aclint_mswi: ipi@2010000 {
    compatible = "riscv,aclint-mswi";
    reg = <0x2010000 0x4000>;
    interrupts-extended = <&cpu0_intc 3>, <&cpu1_intc 3>;
};
```

---

### AIA (Advanced Interrupt Architecture) / 高级中断架构

The next-generation interrupt architecture specification for RISC-V, replacing PLIC, designed for high-performance servers and virtualization scenarios. AIA consists of two main components: APLIC and IMSIC.

RISC-V 新一代中断架构规范，替代 PLIC，专为高性能服务器和虚拟化场景设计。AIA 由两个主要组件组成：APLIC 和 IMSIC。

**AIA Architecture Overview / AIA 架构概览：**

```
Wired Interrupts          MSI (PCIe, etc.)
      ↓                          ↓
   [APLIC]                   Direct MSI
      ↓                          ↓
      └──────→ [IMSIC] ←────────┘
                  ↓
            Hart 0-N (M/S/VS)
```

**Key Advantages / 关键优势：**
- ✅ Native MSI (Message Signaled Interrupts) support / 原生 MSI（消息信号中断）支持
- ✅ Hardware virtualization support / 硬件虚拟化支持
- ✅ Better scalability (thousands of interrupt sources) / 更好的可扩展性（数千个中断源）
- ✅ Lower latency for MSI / MSI 的更低延迟
- ✅ PCIe compatibility / PCIe 兼容性

---

#### APLIC (Advanced Platform-Level Interrupt Controller) / 高级平台级中断控制器

Enhanced version of PLIC, handles wired interrupts and can forward them as MSI to IMSIC.

PLIC 的增强版本，处理线式中断并可将其作为 MSI 转发到 IMSIC。

**APLIC Operating Modes / APLIC 操作模式：**

| Mode / 模式 | Description / 说明 | Use Case / 使用场景 |
|------------|-------------------|---------------------|
| **Direct Mode** | Directly delivers interrupts to harts / 直接将中断交付给 hart | Legacy compatibility, simple systems / 传统兼容性，简单系统 |
| **MSI Mode** | Converts wired interrupts to MSI / 将线式中断转换为 MSI | Modern servers with IMSIC / 带 IMSIC 的现代服务器 |

**APLIC Register Layout (MSI Mode) / APLIC 寄存器布局（MSI 模式）：**

| Offset / 偏移 | Register / 寄存器 | Description / 说明 |
|--------------|------------------|-------------------|
| 0x0000 | `domaincfg` | Domain configuration / 域配置 |
| 0x0004 | `sourcecfg[0]` | Source 0 configuration / 源 0 配置 |
| 0x0008 | `sourcecfg[1]` | Source 1 configuration / 源 1 配置 |
| 0x1BC0 | `mmsiaddrcfg` | Machine MSI address configuration / 机器 MSI 地址配置 |
| 0x1BC8 | `mmsiaddrcfgh` | Machine MSI address configuration high / 机器 MSI 地址配置高位 |
| 0x1C00 | `smsiaddrcfg` | Supervisor MSI address configuration / 监管者 MSI 地址配置 |
| 0x3000 | `setip[0]` | Set interrupt pending bits / 设置中断挂起位 |
| 0x3C00 | `setipnum` | Set interrupt pending by number / 按编号设置中断挂起 |
| 0x4000 | `in_clrip[0]` | Clear interrupt pending bits / 清除中断挂起位 |
| 0x5000 | `setie[0]` | Set interrupt enable bits / 设置中断使能位 |
| 0x6000 | `clrie[0]` | Clear interrupt enable bits / 清除中断使能位 |
| 0x8000 | `target[0]` | Target configuration for source 0 / 源 0 的目标配置 |

**Source Configuration / 源配置：**
- **Delegate (D) bit**: Delegate to child domain / 委托给子域
- **Source Mode (SM)**: Inactive (0), Detached (1), Edge1 (4), Edge0 (5), Level1 (6), Level0 (7) / 源模式：非活动、分离、上升沿、下降沿、高电平、低电平

**APLIC Domains / APLIC 域：**
- **Machine Domain**: Handles M-mode interrupts / 处理 M 模式中断
- **Supervisor Domain**: Handles S-mode interrupts / 处理 S 模式中断
- Hierarchical delegation support / 层次委托支持

---

#### IMSIC (Incoming MSI Controller) / 传入 MSI 控制器

Per-hart interrupt controller that receives MSI (Message Signaled Interrupts) and delivers them to the hart. IMSIC is the core component enabling MSI support in RISC-V.

每个 hart 的中断控制器，接收 MSI（消息信号中断）并将其交付给 hart。IMSIC 是在 RISC-V 中启用 MSI 支持的核心组件。

**IMSIC Architecture / IMSIC 架构：**

```
MSI Write → [Interrupt File] → [Interrupt Delivery Logic] → Hart
                ↓
        [VS Interrupt Files] (for virtualization)
```

**Key Concepts / 关键概念：**

| Concept / 概念 | Description / 说明 |
|---------------|-------------------|
| **Interrupt File** | Per-hart storage for pending interrupts (2048-4096 IDs) / 每个 hart 的挂起中断存储（2048-4096 个 ID）|
| **Interrupt Identity** | Unique ID for each interrupt (0-2047 or 0-4095) / 每个中断的唯一 ID（0-2047 或 0-4095）|
| **EIID** | External Interrupt Identity / 外部中断标识 |
| **Privilege Level** | M-mode or S-mode interrupt file / M 模式或 S 模式中断文件 |
| **Guest Interrupt Files** | Per-VM interrupt files for virtualization / 每个虚拟机的中断文件用于虚拟化 |

**IMSIC CSRs / IMSIC CSR：**

| CSR Name / CSR 名称 | Address / 地址 | Privilege / 特权 | Description / 说明 |
|--------------------|---------------|-----------------|-------------------|
| `miselect` | 0x350 | M | Machine interrupt select / 机器中断选择 |
| `mireg` | 0x351 | M | Machine interrupt register / 机器中断寄存器 |
| `mtopei` | 0x35C | M | Machine top external interrupt / 机器顶部外部中断 |
| `mtopi` | 0xFB0 | M | Machine top interrupt / 机器顶部中断 |
| `siselect` | 0x150 | S | Supervisor interrupt select / 监管者中断选择 |
| `sireg` | 0x151 | S | Supervisor interrupt register / 监管者中断寄存器 |
| `stopei` | 0x15C | S | Supervisor top external interrupt / 监管者顶部外部中断 |
| `stopi` | 0xDB0 | S | Supervisor top interrupt / 监管者顶部中断 |
| `vsiselect` | 0x250 | VS | Virtual supervisor interrupt select / 虚拟监管者中断选择 |
| `vsireg` | 0x251 | VS | Virtual supervisor interrupt register / 虚拟监管者中断寄存器 |
| `vstopei` | 0x25C | VS | Virtual supervisor top external interrupt / 虚拟监管者顶部外部中断 |
| `vstopi` | 0xEB0 | VS | Virtual supervisor top interrupt / 虚拟监管者顶部中断 |

**Interrupt File Indirect Access / 中断文件间接访问：**

Using `miselect` + `mireg` (or S/VS variants):

使用 `miselect` + `mireg`（或 S/VS 变体）：

1. Write interrupt identity to `miselect` / 将中断标识写入 `miselect`
2. Read/write `mireg` to access interrupt file entry / 读写 `mireg` 以访问中断文件条目

**`mireg` Layout / `mireg` 布局：**
- Bit 0: `eip` (external interrupt pending) / 外部中断挂起
- Bit 1: `eie` (external interrupt enable) / 外部中断使能
- Bits 7:0: `eidelivery` (interrupt delivery configuration) / 中断交付配置
- Bits 31:24: `eithreshold` (interrupt priority threshold) / 中断优先级阈值

**Top Interrupt CSRs / 顶部中断 CSR：**

`mtopei` / `stopei` / `vstopei` provide atomic claim of the highest-priority pending and enabled interrupt:

`mtopei` / `stopei` / `vstopei` 提供最高优先级挂起且已启用中断的原子声明：

- Read returns interrupt identity (shifted left by 16 bits) + priority / 读取返回中断标识（左移 16 位）+ 优先级
- Reading automatically claims the interrupt (clears pending) / 读取自动声明中断（清除挂起）
- Write 0 to complete the interrupt / 写 0 以完成中断

**MSI Address Format / MSI 地址格式：**

```
MSI Write Address:
  [Base Address][Hart Index][Privilege][Interrupt Identity]
  
  MSI 写入地址：
  [基地址][Hart 索引][特权级][中断标识]
```

**IMSIC Interrupt Handler Example / IMSIC 中断处理程序示例：**
```c
void imsic_handler(void) {
    // Read and claim top interrupt / 读取并声明顶部中断
    unsigned long topei = read_csr(stopei);
    
    if (topei == 0)
        return; // No interrupt / 无中断
    
    // Extract interrupt ID / 提取中断 ID
    unsigned int irq_id = topei >> 16;
    
    // Handle interrupt / 处理中断
    handle_msi_interrupt(irq_id);
    
    // Complete interrupt / 完成中断
    write_csr(stopei, 0);
}
```

**IMSIC Virtualization Support / IMSIC 虚拟化支持：**

IMSIC provides **Guest Interrupt Files** for each VM, enabling direct interrupt delivery to guests without hypervisor intervention.

IMSIC 为每个虚拟机提供**客户中断文件**，无需虚拟化监控程序干预即可直接将中断交付给客户。

**Guest Interrupt File Selection / 客户中断文件选择：**
- Controlled by `hstatus.VGEIN` field / 由 `hstatus.VGEIN` 字段控制
- Each VM gets its own interrupt file with 2048-4096 interrupt IDs / 每个虚拟机获得自己的中断文件，包含 2048-4096 个中断 ID
- Hypervisor can configure MSI routing per-VM / 虚拟化监控程序可以为每个虚拟机配置 MSI 路由

**IMSIC Device Tree Example / IMSIC 设备树示例：**
```dts
imsic_m: interrupt-controller@24000000 {
    compatible = "riscv,imsics";
    reg = <0x24000000 0x4000>;
    interrupt-controller;
    #interrupt-cells = <0>;
    interrupts-extended = <&cpu0_intc 11>, <&cpu1_intc 11>;
    riscv,num-ids = <255>;
    riscv,guest-index-bits = <4>;
};

imsic_s: interrupt-controller@28000000 {
    compatible = "riscv,imsics";
    reg = <0x28000000 0x4000>;
    interrupt-controller;
    #interrupt-cells = <0>;
    interrupts-extended = <&cpu0_intc 9>, <&cpu1_intc 9>;
    riscv,num-ids = <255>;
    riscv,guest-index-bits = <4>;
};
```

---

### Interrupt Controller Comparison / 中断控制器对比

| Feature / 特性 | PLIC | ACLINT | AIA (APLIC+IMSIC) |
|---------------|------|--------|-------------------|
| **Wired Interrupts** / 线式中断 | ✅ | ❌ | ✅ (APLIC) |
| **Hart-Local Interrupts** / Hart 本地中断 | ❌ | ✅ (Timer+SW) | ✅ (IMSIC) |
| **MSI Support** / MSI 支持 | ❌ | ❌ | ✅ (IMSIC) |
| **Virtualization** / 虚拟化 | ❌ | ❌ | ✅ (Guest files) |
| **Scalability** / 可扩展性 | ~1023 sources / ~1023 源 | Fixed / 固定 | 2048-4096 per hart / 每个 hart 2048-4096 |
| **PCIe Compatibility** / PCIe 兼容性 | Needs bridge / 需要桥接 | N/A | ✅ Native MSI / 原生 MSI |
| **Latency** / 延迟 | Medium / 中等 | Low (local) / 低（本地）| Low (MSI) / 低（MSI）|
| **Deployment** / 部署 | ✅ Widespread / 广泛 | ✅ Standard / 标准 | 🔄 Emerging / 新兴 |
| **Use Case** / 使用场景 | Legacy systems / 传统系统 | MCU/Embedded / MCU/嵌入式 | Servers/HPC / 服务器/HPC |

**Recommended Interrupt Architecture / 推荐的中断架构：**

| System Type / 系统类型 | Recommended / 推荐 | Rationale / 理由 |
|-----------------------|-------------------|-----------------|
| **Microcontroller** / 微控制器 | ACLINT | Simple, low cost / 简单，低成本 |
| **Embedded Linux** / 嵌入式 Linux | ACLINT + PLIC | Mature, well-supported / 成熟，支持良好 |
| **Application Processor** / 应用处理器 | ACLINT + PLIC or AIA | Transitioning to AIA / 过渡到 AIA |
| **Server** / 服务器 | AIA (APLIC+IMSIC) | MSI, virtualization / MSI，虚拟化 |
| **HPC** / 高性能计算 | AIA (APLIC+IMSIC) | Scalability, low latency / 可扩展性，低延迟 |

### MSI (Message Signaled Interrupt) / 消息信号中断

A mechanism for triggering interrupts via memory write operations (rather than dedicated interrupt pins), commonly used by PCIe devices. AIA makes RISC-V natively support MSI, fundamental for modern high-performance SoCs and virtualization scenarios.

通过内存写操作（而非独立中断引脚）触发中断的机制（PCIe 设备常用）。AIA 使 RISC-V 原生支持 MSI，现代高性能 SoC 和虚拟化场景的基础。

---

## Platform Profiles / 平台配置文件（Platform Profiles）

### RVA22 Profile / RVA22 配置文件

"RISC-V Application 2022" profile, defines the minimum set of extensions that application processors (running Linux and other OSes) must support:

"RISC-V Application 2022" 配置文件，定义应用处理器（运行 Linux 等 OS）必须支持的最小扩展集合：

- Mandatory / 必须：RV64I + M + A + F + D + C + Zicsr + Zifencei + Zicntr + Ziccif + Ziccamoa + Zicclsm + Za128rs
- Recommended / 推荐：B extensions (Zba/Zbb), V extension / B 扩展（Zba/Zbb）、V 扩展

### RVA23 Profile / RVA23 配置文件

Upgraded version of RVA22, adds V extension (vector) as mandatory, along with H extension (virtualization). Target: application processors after 2023.

RVA22 的升级版，增加了 V 扩展（向量）作为必须项，以及 H 扩展（虚拟化）。目标：2023 年后的应用处理器。

### Server Platform Specification / 服务器平台规范

Defines specifications that RISC-V server-grade hardware must meet, including mandatory extension sets (based on RVA Profile), interrupt controllers (AIA), memory types (MMIO / DMA), firmware interfaces (SBI + UEFI), etc. Aims to align with Arm ServerReady.

定义 RISC-V 服务器级硬件必须满足的规范，包含必需的扩展集（基于 RVA Profile）、中断控制器（AIA）、内存类型（MMIO / DMA）、固件接口（SBI + UEFI）等，目标与 Arm ServerReady 对齐。

### Embedded Platform / 嵌入式平台

Platform specification for MCU/RTOS scenarios, typically requires only M+U modes, PMP replaces virtual memory, ACLINT provides timer.

针对 MCU/RTOS 场景的平台规范，通常只需 M+U 模式、PMP 替代虚拟内存、ACLINT 提供定时器。

---

## Device Tree / 设备树（Device Tree）

### Device Tree (DT) / 设备树

A data structure describing hardware topology and resources (memory addresses, interrupt numbers, clocks), existing in `.dts` (source) / `.dtb` (binary) forms. RISC-V platform firmware (OpenSBI) passes DTB address to kernel (via `a1` register), which uses it to initialize device drivers.

描述硬件拓扑和资源（内存地址、中断号、时钟）的数据结构，以 `.dts`（源码）/ `.dtb`（二进制）形式存在。RISC-V 平台固件（OpenSBI）将 DTB 地址传给内核（通过 `a1` 寄存器），内核据此初始化设备驱动。

### ACPI (Advanced Configuration and Power Interface) / 高级配置与电源接口

Hardware description standard used by x86/Arm servers, RISC-V server platform specification also requires ACPI support to reuse existing OS driver ecosystems.

x86/Arm 服务器使用的硬件描述标准，RISC-V 服务器平台规范也要求支持 ACPI，以复用现有操作系统驱动生态。

---

## Firmware and Boot / 固件与启动

### UEFI (Unified Extensible Firmware Interface) / 统一可扩展固件接口

Firmware interface standard for RISC-V server platforms (replacing BIOS), provides standardized Boot Services and Runtime Services. RISC-V UEFI specification has been approved.

RISC-V 服务器平台的固件接口标准（替代 BIOS），提供标准化启动服务（Boot Services）和运行时服务（Runtime Services）。RISC-V UEFI 规范已获批准。

### U-Boot / U-Boot

Open-source bootloader widely used for RISC-V embedded platforms, supports loading kernel from NAND/SD/NVMe, works with OpenSBI (OpenSBI in M-mode, U-Boot in S-mode).

广泛用于 RISC-V 嵌入式平台的开源 Bootloader，支持从 NAND/SD/NVMe 加载内核，与 OpenSBI 配合（OpenSBI 在 M 模式，U-Boot 在 S 模式）。

### GRUB2 / GRUB2

GNU general-purpose bootloader, already supports RISC-V, used for loading Linux on RISC-V server and desktop platforms.

GNU 通用 Bootloader，已支持 RISC-V，用于 RISC-V 服务器和桌面平台加载 Linux。

---

## Power Management and System Control / 电源管理与系统控制

### ACPI RISC-V / ACPI RISC-V

RISC-V Server Platform specification requires ACPI (Advanced Configuration and Power Interface) support, enabling existing OSes like Linux/Windows to discover hardware through standard means without needing to customize Device Tree for each platform. RISC-V ACPI extensions are defined in ACPI 6.6+.

RISC-V Server Platform 规范要求支持 ACPI（Advanced Configuration and Power Interface），使 Linux/Windows 等现有 OS 通过标准方式发现硬件，无需为每个平台定制 Device Tree。RISC-V ACPI 扩展在 ACPI 6.6+ 中定义。

### RISCV_EFI_BOOT_PROTOCOL / UEFI 启动协议

Boot protocol defined in RISC-V UEFI specification, UEFI firmware passes hartid and FDT/ACPI address through UEFI services to OS loader (e.g., GRUB), which then passes them to Linux kernel.

RISC-V UEFI 规范定义的启动协议，UEFI 固件将 hartid 和 FDT/ACPI 地址通过 UEFI 服务传递给 OS loader（如 GRUB），再由 loader 传递给 Linux 内核。

### SBI SRST (System Reset) / SBI 系统复位扩展

`sbi_system_reset(reset_type, reset_reason)` call, allows S-mode software to trigger system shutdown (`SHUTDOWN`) or reset (`COLD_REBOOT`/`WARM_REBOOT`) without directly manipulating hardware registers.

`sbi_system_reset(reset_type, reset_reason)` 调用，允许 S 模式软件触发系统关机（`SHUTDOWN`）或复位（`COLD_REBOOT`/`WARM_REBOOT`），无需直接操作硬件寄存器。

### SBI PMU (Performance Monitoring Unit) / SBI 性能监控扩展

Allows S-mode software (`perf` tool) to access and configure hardware performance counters (`hpmcounter`) via SBI calls, avoiding direct access to M-mode CSRs. Supports counter start/stop, event configuration, and overflow interrupts.

允许 S 模式软件（`perf` 工具）访问和配置硬件性能计数器（`hpmcounter`），通过 SBI 调用避免直接访问 M 模式 CSR。支持计数器启动/停止、事件配置、溢出中断。

### SBI SUSP (System Suspend) / SBI 系统挂起扩展
`sbi_system_suspend(sleep_type, resume_addr, opaque)` 调用，实现系统级挂起到内存（STR，S3 睡眠），M 模式固件负责保存/恢复平台状态，唤醒后从 `resume_addr` 继续执行。

---

## 内存布局与地址空间

### MMIO (Memory-Mapped I/O) / 内存映射 I/O
将外设寄存器映射到物理地址空间，通过普通 Load/Store 指令访问。RISC-V 平台通过 Device Tree 的 `reg` 属性或 ACPI 描述 MMIO 区域，内核据此 ioremap 后访问。

### physical memory map / 物理内存映射
典型 RISC-V 平台（QEMU virt）的物理地址布局：
```
0x0000_0000           Reset vector（只读，跳转到固件）
0x0200_0000           CLINT（mtime / mtimecmp / msip）
0x0C00_0000           PLIC（优先级/阈值/claim/complete）
0x1000_0000           UART 0（串口控制台）
0x8000_0000           DRAM 起始（固件/内核加载地址）
```

### firmware load address / 固件加载地址
大多数 RISC-V 平台将固件（OpenSBI）加载到 DRAM 起始地址（通常 `0x8000_0000`），内核加载到固件之后（`0x8020_0000` 或由 DTB 指定）。

---

## 设备树深入

### FDT (Flattened Device Tree) / 扁平化设备树
Device Tree Source (`.dts`) 编译后的二进制格式（`.dtb`），固件将其地址传递给内核（`a1` 寄存器）。内核通过 `libfdt` 解析，发现并初始化硬件。

### riscv,isa / riscv,isa-extensions / DT ISA 属性
设备树中描述 CPU 支持的 ISA 扩展：
- `riscv,isa`（旧格式）：如 `"rv64imafdc"`
- `riscv,isa-extensions`（新格式，Linux 6.5+）：字符串列表，如 `["rv64i", "m", "a", "zicsr"]`

### cpu node / cpu DT 节点
Device Tree 中描述每个 hart 的节点，包含 `device_type = "cpu"`、`reg`（hartid）、`riscv,isa`、`mmu-type`（sv39/sv48）等属性，Linux 据此配置每颗 CPU。

### chosen node / chosen DT 节点
Device Tree 中传递启动参数的特殊节点：
- `bootargs`：内核命令行（如 `"console=ttyS0 root=/dev/vda"`）
- `stdout-path`：早期控制台设备路径
- `riscv,boot-hart-id`：负责启动的主 hart ID

---

## RISC-V 平台合规

### riscv-arch-test framework / 架构一致性测试框架
→ 详见 [[software/vocabulary]]

### RISC-V Compatibility / RISC-V 兼容性认证
实现通过 riscv-arch-test 测试套件后可向 RISC-V International 申请兼容性认证，证明实现符合 ISA 规范，可使用 RISC-V 商标。

### HART Level ISA String / hart 级 ISA 字符串
在设备树中用于描述单个 hart 支持的完整 ISA（包含所有扩展），Linux 内核解析后用于：启用/禁用对应功能（如 V 扩展 SIMD）、选择最优代码路径、报告 `/proc/cpuinfo`。

---

## Boot Flow / 启动流程

### RISC-V Boot Flow / RISC-V 启动流程

The typical boot sequence for a RISC-V system:

RISC-V 系统的典型启动序列：

```
1. Reset → ROM Code (M-mode)
   复位 → ROM 代码（M 模式）
        ↓
2. Load Bootloader/Firmware (e.g., OpenSBI)
   加载 Bootloader/固件（如 OpenSBI）
        ↓
3. OpenSBI initializes platform (M-mode)
   OpenSBI 初始化平台（M 模式）
        ↓
4. Load & jump to U-Boot/UEFI (S-mode)
   加载并跳转到 U-Boot/UEFI（S 模式）
        ↓
5. Load kernel & Device Tree
   加载内核和设备树
        ↓
6. Jump to kernel (S-mode)
   跳转到内核（S 模式）
        ↓
7. Kernel initializes & starts init/systemd (U-mode)
   内核初始化并启动 init/systemd（U 模式）
```

**Register Convention at Kernel Entry / 内核入口的寄存器约定：**
- `a0` = `hartid` (current hart ID) / 当前 hart ID
- `a1` = FDT physical address / FDT 物理地址
- `satp` = 0 (MMU disabled) / MMU 禁用
- `sstatus.SIE` = 0 (interrupts disabled) / 中断禁用

---

### Firmware Handoff Protocol / 固件切换协议

Defines how firmware passes control and information to the next boot stage:

定义固件如何将控制权和信息传递给下一个启动阶段：

**M-mode → S-mode Transition / M 模式 → S 模式转换：**
- OpenSBI passes FDT address in `a1` / OpenSBI 在 `a1` 中传递 FDT 地址
- Jumps to S-mode entry point via `mret` / 通过 `mret` 跳转到 S 模式入口
- Sets `mstatus.MPP = 1` (S-mode), `mepc = entry_point` / 设置 `mstatus.MPP = 1`（S 模式）、`mepc = entry_point`

**S-mode → Kernel / S 模式 → 内核：**
- U-Boot/UEFI loads kernel image to memory / U-Boot/UEFI 将内核镜像加载到内存
- Prepares FDT with bootargs / 准备带 bootargs 的 FDT
- Jumps to kernel entry with FDT address in `a1` / 跳转到内核入口，`a1` 中包含 FDT 地址

---

### FW_PAYLOAD / FW_PAYLOAD

OpenSBI firmware format that includes a payload (e.g., U-Boot or Linux) embedded directly into the firmware binary. The firmware initializes the platform then jumps to the payload.

OpenSBI 固件格式，直接将有效负载（如 U-Boot 或 Linux）嵌入固件二进制中。固件初始化平台后跳转到有效负载。

**Usage / 使用：**
```bash
# Build OpenSBI with U-Boot payload
make PLATFORM=generic FW_PAYLOAD_PATH=u-boot.bin
```

---

### FW_JUMP / FW_JUMP

OpenSBI firmware format that jumps to a fixed address after initialization. The next stage bootloader must be loaded to that address by a previous boot stage (e.g., ROM code).

OpenSBI 固件格式，初始化后跳转到固定地址。下一阶段 bootloader 必须由前一启动阶段（如 ROM 代码）加载到该地址。

**Configuration / 配置：**
- `FW_JUMP_ADDR` = Address to jump to (e.g., 0x80200000) / 跳转地址（如 0x80200000）
- `FW_JUMP_FDT_ADDR` = FDT address (optional) / FDT 地址（可选）

---

### FW_DYNAMIC / FW_DYNAMIC

OpenSBI firmware format that receives information about the next boot stage via a dynamic info structure passed by the previous boot stage. This is the most flexible format.

OpenSBI 固件格式，通过前一启动阶段传递的动态信息结构接收下一启动阶段的信息。这是最灵活的格式。

**Dynamic Info Structure / 动态信息结构：**
```c
struct fw_dynamic_info {
    unsigned long magic;          // Magic value / 魔数
    unsigned long version;        // Structure version / 结构版本
    unsigned long next_addr;      // Next stage address / 下一阶段地址
    unsigned long next_mode;      // Next stage privilege mode / 下一阶段特权模式
    unsigned long options;        // Options / 选项
    unsigned long boot_hart;      // Boot hart ID / 启动 hart ID
};
```

---

## Platform Security / 平台安全

### PMP (Physical Memory Protection) / 物理内存保护

Hardware mechanism to restrict physical memory access by privilege level, configured by M-mode to protect firmware and critical regions from S/U-mode access.

硬件机制，通过特权级限制物理内存访问，由 M 模式配置以保护固件和关键区域免受 S/U 模式访问。

**Common PMP Configurations / 常见 PMP 配置：**
- Region 0: M-mode only (firmware) / 仅 M 模式（固件）
- Region 1: M+S-mode (kernel) / M+S 模式（内核）
- Region 2: All modes (user memory) / 所有模式（用户内存）

---

### sPMP (Supervisor PMP) / 监管者物理内存保护

Extension allowing S-mode to configure PMP for U-mode isolation without M-mode intervention. Useful for embedded systems without virtualization.

扩展允许 S 模式为 U 模式隔离配置 PMP，无需 M 模式干预。对没有虚拟化的嵌入式系统有用。

---

### TEE (Trusted Execution Environment) / 可信执行环境

Secure execution environment isolated from the normal OS. RISC-V implementations include:

与正常 OS 隔离的安全执行环境。RISC-V 实现包括：

- **Keystone**: Open-source TEE framework for RISC-V / RISC-V 的开源 TEE 框架
- **MultiZone**: Commercial TEE solution using PMP / 使用 PMP 的商业 TEE 解决方案
- **RISC-V AP-TEE**: Standard TEE architecture specification / 标准 TEE 架构规范

---

### Secure Boot / 安全启动

Process of verifying the integrity and authenticity of firmware and software before execution. RISC-V platforms implement secure boot through:

在执行前验证固件和软件的完整性和真实性的过程。RISC-V 平台通过以下方式实现安全启动：

- **ROM Code Verification**: First-stage bootloader verifies firmware signature / 第一阶段 bootloader 验证固件签名
- **Chain of Trust**: Each stage verifies the next stage / 每个阶段验证下一阶段
- **Measured Boot**: Records hashes in TPM/hardware / 在 TPM/硬件中记录哈希值

---

## Platform Debugging / 平台调试

### Debug Module / 调试模块

Hardware component implementing the RISC-V Debug Specification, allowing external debuggers (GDB, OpenOCD) to control hart execution, read/write memory and registers.

实现 RISC-V 调试规范的硬件组件，允许外部调试器（GDB、OpenOCD）控制 hart 执行、读写内存和寄存器。

**Debug Transport Module (DTM) / 调试传输模块：**
- **JTAG**: Standard debug interface / 标准调试接口
- **USB**: Direct USB debug connection / 直接 USB 调试连接

**Debug Module Interface (DMI) / 调试模块接口：**
- Abstract Commands: Read/write CSRs and GPRs / 读写 CSR 和 GPR
- System Bus Access: Direct memory access / 直接内存访问
- Program Buffer: Execute small programs / 执行小程序

---

### Trigger Module / 触发器模块

Hardware breakpoint/watchpoint mechanism defined in the RISC-V Debug Specification. Supports:

RISC-V 调试规范中定义的硬件断点/观察点机制。支持：

**Trigger Types / 触发器类型：**
- Type 2: Address/Data Match (instruction/data breakpoints) / 地址/数据匹配（指令/数据断点）
- Type 3: Instruction Count / 指令计数
- Type 4: Interrupt Trigger / 中断触发
- Type 5: Exception Trigger / 异常触发
- Type 6: Address/Data Match (enhanced) / 地址/数据匹配（增强）

**Trigger CSRs / 触发器 CSR：**
- `tselect`: Select trigger / 选择触发器
- `tdata1`: Trigger control / 触发器控制
- `tdata2`: Trigger data (address/value) / 触发器数据（地址/值）
- `tdata3`: Additional trigger data / 附加触发器数据

---

### JTAG (Joint Test Action Group) / 联合测试行动组

Standard debug interface for RISC-V platforms, uses 4-5 pins (TDI, TDO, TCK, TMS, TRST) to communicate with the Debug Module.

RISC-V 平台的标准调试接口，使用 4-5 个引脚（TDI、TDO、TCK、TMS、TRST）与调试模块通信。

**Common JTAG Operations / 常见 JTAG 操作：**
- Halt/Resume hart execution / 停止/恢复 hart 执行
- Single-step execution / 单步执行
- Read/Write registers and memory / 读写寄存器和内存
- Set hardware breakpoints / 设置硬件断点

---

### OpenOCD (Open On-Chip Debugger) / 开放片上调试器

Open-source debugger supporting RISC-V via JTAG. Provides GDB remote protocol interface for debugging with GDB.

通过 JTAG 支持 RISC-V 的开源调试器。为使用 GDB 调试提供 GDB 远程协议接口。

**OpenOCD Configuration Example / OpenOCD 配置示例：**
```tcl
# RISC-V target configuration
adapter driver ftdi
adapter speed 1000

set _CHIPNAME riscv
jtag newtap $_CHIPNAME cpu -irlen 5 -expected-id 0x10e31913

set _TARGETNAME $_CHIPNAME.cpu
target create $_TARGETNAME riscv -chain-position $_TARGETNAME

init
halt
```

---

### Trace Encoder / 跟踪编码器

Hardware component that captures instruction execution traces for analysis. RISC-V defines two trace specifications:

捕获指令执行跟踪以供分析的硬件组件。RISC-V 定义了两种跟踪规范：

**N-Trace (Nexus Trace) / N-跟踪（Nexus 跟踪）：**
- Industry standard trace format / 行业标准跟踪格式
- Compressed trace output / 压缩的跟踪输出
- Supports instruction and data tracing / 支持指令和数据跟踪

**E-Trace (Efficient Trace) / E-跟踪（高效跟踪）：**
- RISC-V-specific trace format / RISC-V 特定的跟踪格式
- Highly compressed / 高度压缩
- Lower bandwidth requirements / 更低的带宽要求

---

## Platform Peripherals / 平台外设

### UART (Universal Asynchronous Receiver-Transmitter) / 通用异步收发器

Serial communication peripheral commonly used for console output on RISC-V platforms.

RISC-V 平台上常用于控制台输出的串行通信外设。

**Common UART Controllers / 常见 UART 控制器：**
- **16550 UART**: Industry standard, widely supported / 行业标准，广泛支持
- **SiFive UART**: Used in SiFive SoCs / 用于 SiFive SoC
- **8250 UART**: Legacy compatible / 传统兼容

**Device Tree Example / 设备树示例：**
```dts
uart0: serial@10000000 {
    compatible = "ns16550a";
    reg = <0x10000000 0x100>;
    interrupts = <10>;
    clock-frequency = <3686400>;
    reg-shift = <2>;
    reg-io-width = <1>;
};
```

---

### SPI (Serial Peripheral Interface) / 串行外设接口

Synchronous serial communication protocol for connecting microcontrollers to peripheral devices (flash, SD cards, sensors).

用于将微控制器连接到外设（闪存、SD 卡、传感器）的同步串行通信协议。

**SPI Signals / SPI 信号：**
- **SCLK**: Serial clock / 串行时钟
- **MOSI**: Master out, slave in / 主机输出，从机输入
- **MISO**: Master in, slave out / 主机输入，从机输出
- **CS/SS**: Chip select / 片选

---

### I2C (Inter-Integrated Circuit) / 集成电路间总线

Two-wire serial communication protocol for low-speed peripheral devices (sensors, EEPROMs, RTCs).

用于低速外设（传感器、EEPROM、RTC）的双线串行通信协议。

**I2C Signals / I2C 信号：**
- **SDA**: Serial data / 串行数据
- **SCL**: Serial clock / 串行时钟

**Features / 特性：**
- Multi-master support / 支持多主机
- 7-bit or 10-bit addressing / 7 位或 10 位寻址
- Standard (100 kHz), Fast (400 kHz), High-speed (3.4 MHz) modes / 标准（100 kHz）、快速（400 kHz）、高速（3.4 MHz）模式

---

### GPIO (General Purpose Input/Output) / 通用输入输出

Configurable digital I/O pins for interfacing with simple devices (LEDs, buttons, sensors).

用于与简单设备（LED、按钮、传感器）接口的可配置数字 I/O 引脚。

**GPIO Operations / GPIO 操作：**
- Set direction (input/output) / 设置方向（输入/输出）
- Read input value / 读取输入值
- Write output value / 写入输出值
- Configure interrupt (edge/level triggered) / 配置中断（边沿/电平触发）

---

### PCIe (PCI Express) / PCIe

High-speed serial bus for connecting peripherals (GPUs, NICs, NVMe). RISC-V server platforms require PCIe support.

用于连接外设（GPU、NIC、NVMe）的高速串行总线。RISC-V 服务器平台需要 PCIe 支持。

**PCIe on RISC-V / RISC-V 上的 PCIe：**
- Uses IMSIC for MSI delivery / 使用 IMSIC 进行 MSI 传递
- Supports standard PCIe BAR mapping / 支持标准 PCIe BAR 映射
- DMA requires IOMMU for address translation / DMA 需要 IOMMU 进行地址转换

---

### IOMMU (Input-Output Memory Management Unit) / 输入输出内存管理单元

Hardware component that translates DMA addresses from devices to physical addresses, providing memory protection and virtualization support.

将设备的 DMA 地址转换为物理地址的硬件组件，提供内存保护和虚拟化支持。

**RISC-V IOMMU Specification / RISC-V IOMMU 规范：**
- Supports Sv39/Sv48/Sv57 page tables / 支持 Sv39/Sv48/Sv57 页表
- Device contexts for per-device translation / 每个设备的设备上下文转换
- ATS (Address Translation Services) support / ATS（地址转换服务）支持
- Fault reporting / 故障报告

---

## Platform Virtualization / 平台虚拟化

### KVM (Kernel-based Virtual Machine) / 基于内核的虚拟机

Linux kernel module that turns the kernel into a hypervisor. RISC-V KVM support added in Linux 5.15+.

将内核转换为虚拟化监控程序的 Linux 内核模块。RISC-V KVM 支持在 Linux 5.15+ 中添加。

**RISC-V KVM Features / RISC-V KVM 特性：**
- H extension required / 需要 H 扩展
- Uses VS/VU modes for guest execution / 使用 VS/VU 模式执行客户
- Two-stage address translation (G-stage + VS-stage) / 两阶段地址转换（G 阶段 + VS 阶段）
- Virtual interrupt delivery via AIA / 通过 AIA 的虚拟中断传递

---

### QEMU / QEMU

Open-source emulator and virtualizer supporting RISC-V. Provides multiple machine types:

支持 RISC-V 的开源仿真器和虚拟化程序。提供多种机器类型：

**QEMU RISC-V Machine Types / QEMU RISC-V 机器类型：**
- **virt**: Generic virtual platform / 通用虚拟平台
- **sifive_u**: SiFive Unleashed board / SiFive Unleashed 开发板
- **spike**: Spike ISA simulator compatibility / Spike ISA 模拟器兼容

**QEMU virt Machine / QEMU virt 机器：**
- Configurable CPU count / 可配置 CPU 数量
- PLIC or AIA interrupt controller / PLIC 或 AIA 中断控制器
- VirtIO devices (disk, network, GPU) / VirtIO 设备（磁盘、网络、GPU）
- OpenSBI firmware / OpenSBI 固件

---

### VirtIO / VirtIO

Virtualization standard for I/O devices, provides efficient paravirtualized drivers for guests.

I/O 设备的虚拟化标准，为客户提供高效的半虚拟化驱动程序。

**Common VirtIO Devices / 常见 VirtIO 设备：**
- **virtio-blk**: Block device (disk) / 块设备（磁盘）
- **virtio-net**: Network device / 网络设备
- **virtio-gpu**: Graphics device / 图形设备
- **virtio-rng**: Random number generator / 随机数生成器
- **virtio-console**: Console device / 控制台设备

---

## Platform Testing and Validation / 平台测试与验证

### riscv-arch-test / RISC-V 架构一致性测试

Official RISC-V architecture conformance test suite, validates ISA implementation correctness.

官方 RISC-V 架构一致性测试套件，验证 ISA 实现的正确性。

**Test Categories / 测试类别：**
- Base integer instructions (RV32I/RV64I) / 基本整数指令（RV32I/RV64I）
- Standard extensions (M, A, F, D, C, etc.) / 标准扩展（M、A、F、D、C 等）
- Privilege modes and CSRs / 特权模式和 CSR
- Virtual memory / 虚拟内存

---

### ACT (Architecture Compatibility Testing) / 架构兼容性测试

Framework for validating that a RISC-V implementation adheres to the ISA specification. Required for RISC-V compatibility certification.

用于验证 RISC-V 实现是否符合 ISA 规范的框架。RISC-V 兼容性认证所需。

---

### Compliance Framework / 合规框架

Testing infrastructure for platform-level compliance (SBI, device tree, boot protocol). Ensures interoperability between firmware, bootloaders, and OSes.

平台级合规性的测试基础设施（SBI、设备树、启动协议）。确保固件、bootloader 和 OS 之间的互操作性。

---

**Last Updated / 最后更新：** 2026-07-23

---

## Quality-of-Service Extensions / 服务质量扩展

### Ssqosid (Supervisor QoS Identifiers) / 监管者QoS标识符扩展

Ratified June 2024. Provides Quality-of-Service (QoS) identifiers that can be associated with memory transactions, enabling resource partitioning and prioritization in shared systems.

2024年6月批准。提供服务质量（QoS）标识符可关联到内存事务，在共享系统中启用资源分区和优先级。

**CSRs / 控制状态寄存器：**
- `srmcfg` (Supervisor Resource Management Configuration): Controls QoS ID assignment
- QoS IDs are attached to memory requests from each hart
- Platform-specific QoS controllers use these IDs for resource management

`srmcfg`（监管者资源管理配置）：控制QoS ID分配
QoS ID附加到每个hart的内存请求
平台特定QoS控制器使用这些ID进行资源管理

**Use Cases / 使用场景：**

1. **Cache Partitioning / 缓存分区：**
   - Assign different cache ways to different QoS IDs
   - Prevent cache thrashing between tenants
   - Guarantee cache allocation for critical workloads
   
   将不同缓存路分配给不同QoS ID
   防止租户间缓存抖动
   保证关键工作负载的缓存分配

2. **Memory Bandwidth QoS / 内存带宽QoS：**
   - Prioritize memory requests by QoS ID
   - Throttle low-priority traffic
   - Guarantee bandwidth for high-priority workloads
   
   按QoS ID优先化内存请求
   限制低优先级流量
   保证高优先级工作负载的带宽

3. **Data Center / Server Applications / 数据中心/服务器应用：**
   - Multi-tenant isolation and fairness
   - Service-level agreement (SLA) enforcement
   - Critical vs best-effort traffic separation
   
   多租户隔离和公平性
   服务级别协议（SLA）执行
   关键与尽力而为流量分离

**Platform Integration / 平台集成：**
- Requires platform-specific QoS controllers (cache controllers, memory controllers, interconnect)
- Software configures QoS policies via platform registers
- OS kernel assigns QoS IDs to processes/VMs/containers

需要平台特定QoS控制器（缓存控制器、内存控制器、互连）
软件通过平台寄存器配置QoS策略
OS内核为进程/VM/容器分配QoS ID

**Example / 示例：**
```c
// Assign QoS ID to current context
csrw(srmcfg, (qos_id << QOS_ID_SHIFT) | QOS_ENABLE);

// All subsequent memory transactions from this hart
// will carry the QoS ID to platform controllers
```

