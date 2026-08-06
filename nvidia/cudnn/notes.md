---
tags: [nvidia, cudnn, notes]
domain: cudnn
last_updated: 2026-07-21
source: https://docs.nvidia.com/deeplearning/cudnn/latest/
---

# cuDNN Overview 精读笔记

> cuDNN 9.x | 文档更新 Jul 2026

---

## 一句话总结

cuDNN = **GPU 加速的 DNN 基础操作库** + **灵活的操作融合框架**。提供卷积、矩阵乘法、注意力、归一化等高度优化的 primitives，并通过 **graph API** 支持多操作融合以进一步提升性能。

---

## 核心架构

```
训练框架 (PyTorch/TF/JAX)
    │
    ▼
用户代码调用 cuDNN
    │
    ├──► Frontend API (Python/C++)  ← 推荐：简化编程模型
    │      │
    │      ├─ Graph 对象（容纳操作图）
    │      ├─ Tensor_attributes（描述张量）
    │      ├─ Operations（conv, matmul, sdpa...）
    │      └─ 自动推断属性、简化 API
    │
    └──► Backend API (C only)       ← 仅在需要遗留例程或纯 C 时使用
           │
           └─ 固定函数例程（非图模式）
```

---

## Graph API 工作流

### 11 步构建和执行

```
1. 创建 Graph 并指定全局属性
   graph.set_io_data_type(FP16)
   graph.set_compute_data_type(FP32)

2. 创建并添加输入张量
   q = graph.tensor(dims=[B,H,S,D], strides=...)

3. 创建并添加操作节点
   o, stats = graph.sdpa(q, k, v, attn_scale=1/√d)

4. 验证操作图
   graph.validate()

5. 构建 cuDNN 操作图
   graph.build_operation_graph(handle)

6. 创建执行计划（基于启发式）
   graph.get_execution_plans([HEUR_MODE_A, FALLBACK])

7. [可选] 检查图支持
   graph.check_support(handle)

8. [可选] 过滤计划
   graph.select_numeric_notes([...])

9. 构建执行计划
   graph.build_plan(handle, BUILD_POLICY_HEURISTIC)

10. [可选] 自动调优
    graph.autotune(...)

11. 执行图
    graph.execute(handle, var_pack, workspace)
```

---

## 启发式与引擎选择

| 模式 | 特点 | 使用场景 |
|------|------|----------|
| **Heuristics A** | 快速，处理大多数模式 | 默认首选 |
| **Heuristics B** | 更准确，CPU 延迟更高 | 需要更精确的性能预测 |
| **Fallback** | 快速功能回退，性能非最优 | A/B 都不支持时 |

**推荐工作流**：
1. 查询 A 或 B 模式
2. `check_support()` 检查第一个引擎配置
3. 如果支持，使用它（预期最佳性能）
4. 如果都不支持，使用 Fallback 模式

**自动调优（可选）**：
- 迭代所有引擎配置
- 实际测量运行时性能
- 选择最快的配置
- 用于生产部署前验证

---

## 张量布局详解

### 4D 卷积张量布局

| 布局 | 维度顺序 | 步幅公式 | 特点 |
|------|----------|----------|------|
| **NCHW** | [N, C, H, W] | [C×H×W, H×W, W, 1] | 标准布局，通道连续 |
| **NHWC** | [N, H, W, C] | [H×W×C, W×C, C, 1] | 每空间位置的通道连续 |
| **NC/32HW32** | [N, C/32, H, W, 32] | 分组布局 | INT8 向量化加载 |

**NHWC 优势**：
- 在许多 GPU 架构上性能更优
- 更好的内存访问模式
- Tensor Core 友好

### 3D Matmul 张量布局

| 布局 | 维度 | 步幅 | 说明 |
|------|------|------|------|
| **Packed Row-major** | [B, M, N] | [M×N, N, 1] | 行主序 |
| **Packed Column-major** | [B, M, N] | [M×N, 1, M] | 列主序 |

### 变长序列布局

**Padded（填充）**：
```
序列 "aa" (len=2), "bbb" (len=3), max_len=8
Q[b=0] = aa000000  (6个填充token)
Q[b=1] = bbb00000  (5个填充token)

需要：
- seq_len_q = [2, 3]
- 标准 BHSD 步幅
```

**Ragged（不规则，无填充）**：
```
序列紧凑打包：
Q = aabbb  (T=5 总tokens，无填充)

需要：
- ragged_offset = [0, 2×H×D, 5×H×D] = [0, 128, 320]
- THD 格式，T = Σ(seq_len)
```

---

## 操作融合模式

### 三大引擎类别

```
┌─────────────────────────────────────────────────┐
│ 1. Pre-compiled Single Operation Engines       │
│    - ConvFwd, ConvBwdData, ConvBwdFilter       │
│    - 预编译，快速，单操作                        │
└─────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────┐
│ 2. Generic Runtime Fusion Engines              │
│    - Matmul fusions: g₂(C=Matmul(g₁ₐ(A), g₁ᵦ(B)))│
│    - ConvFwd fusions: g₂(Y=Conv(g₁(X), W))     │
│    - ConvBwdFilter/Data fusions                 │
│    - Pointwise & Reduction fusions              │
│    - 运行时编译，灵活，支持 SM70/80/90+          │
└─────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────┐
│ 3. Specialized Runtime/Pre-compiled Engines     │
│    - Fused Flash Attention (fprop/bprop)        │
│    - NormAddRelu / DReluForkDNorm               │
│    - NormalizationForward/Backward              │
│    - ConvBNfprop / ConvBNwgrad                  │
│    - 针对常见模式高度优化                        │
└─────────────────────────────────────────────────┘
```

### Generic Fusion 模式结构

```
主循环融合 (g₁)          核心操作          后处理融合 (g₂)
     │                     │                    │
     ▼                     ▼                    ▼
 Pointwise ops      Matmul/Conv         Pointwise/Reduction
  (预处理)         (compute-bound)        (后处理激活等)
     │                     │                    │
     └─────────────────────┴────────────────────┘
              单个融合kernel（减少内存传输）
```

**支持面对比（SM 代表 compute capability）**：

| 特性 | SM90 | SM80 | SM70 |
|------|------|------|------|
| Matmul 融合 | ✅ | ✅ | ✅ |
| ConvFwd 融合 | ✅ | ✅ | ✅ |
| ConvBwdFilter | ✅ | ❌ | ✅ |
| ConvBwdData | 部分 | ❌ | ✅ |
| FP8 支持 | ✅ | SM89+ | ❌ |
| g₁ 融合 | ✅ | ✅ | 部分 |
| 分组卷积 | ✅ | ✅ | ❌ |

---

## Fused Flash Attention 详解

### SDPA 公式

```
Attention(Q,K,V) = softmax(QK^T / √d) V

其中：
- Q: [B, H_q, S_q, D_qk]
- K: [B, H_k, S_kv, D_qk]
- V: [B, H_v, S_kv, D_v]
- 输出 O: [B, H_q, S_q, D_v]
```

### 支持的注意力变体

| 变体 | H_q | H_k, H_v | 说明 |
|------|-----|----------|------|
| **MHA** | 64 | 64, 64 | 多头注意力（标准） |
| **GQA** | 64 | 8, 8 | 分组查询注意力 |
| **MQA** | 64 | 1, 1 | 多查询注意力（极致KV cache节省） |

### 掩码选项

```python
# 1. Causal Mask (因果掩码 - GPT风格)
diagonal_alignment=TOP_LEFT, diagonal_band_right_bound=0

# 2. Sliding Window (滑动窗口)
diagonal_band_left_bound=128  # 窗口大小

# 3. Padding Mask (变长序列)
use_padding_mask=True
seq_len_q=[actual_lens], seq_len_kv=[actual_lens]

# 4. ALiBi (线性偏置位置编码)
use_alibi_mask=True  # 必须配合 causal mask

# 5. Block Mask (128×128 块级掩码)
block_mask=mask_tensor  # 仅 UNIFIED 实现支持
```

### Paged Attention（分页注意力）

**解决问题**：减少 KV cache 内存碎片

```
传统连续 KV cache:
K = [B, H, S_max, D]  ← 每个batch分配max_len，浪费

Paged KV cache:
K_container = [num_blocks, H, block_size, D]
page_table_k = [B, 1, ceil(s_kv/block_size), 1]  # INT32索引

访问公式：
K_cache[b,h,s,d] = K_container[page_table_k[b,1,s/bs,1], h, s%bs, d]
```

**优势**：
- 按需分配块
- 减少内存浪费
- 支持非连续存储

### FP8 SDPA 特殊性

**额外张量**：
- Descale因子：descale_q, descale_k, descale_v, descale_s
- Scale因子：scale_s, scale_o
- Amax输出：amax_s, amax_o（用于下一层计算scale）

**量化因子计算**：
```
E4M3: scale = 448.0 / prev_layer_amax
E5M2: scale = 57344.0 / prev_layer_amax
descale = 1.0 / scale
```

---

## 归一化操作对比

| 归一化类型 | 归一化维度 | Mean输出 | 用途 |
|-----------|-----------|---------|------|
| **LayerNorm** | 最后 D' 维 | ✅ | Transformer标准 |
| **RMSNorm** | 最后 D' 维 | ❌ | 仅RMS，更快 |
| **AdaLayerNorm** | 最后 D' 维 | ✅ | 条件生成模型 |
| **BatchNorm** | 批次+空间 | ✅ | CNN标准 |
| **InstanceNorm** | 空间维度 | ✅ | 风格迁移 |

**Prologue & Epilogue 融合**：
- g₁: 归一化前的 pointwise 操作
- g₂: 归一化后的 pointwise 操作
- 支持融合为单个 kernel

---

## cuDNN JIT vs FULL

| 对比项 | cuDNN JIT | cuDNN FULL |
|--------|-----------|------------|
| **二进制大小** | 小（仅RTC引擎） | 大（包含所有预编译kernel） |
| **支持操作** | 仅运行时融合引擎 | 所有引擎 |
| **Legacy API** | ❌ | ✅ |
| **预编译引擎** | ❌ | ✅ |
| **Flash Attention** | ✅ 无差异 | ✅ |
| **Normalization** | ✅ 无差异 | ✅ |
| **Matmul/Conv融合** | ✅ 但有对齐约束 | ✅ |
| **性能** | 可能略差 | 最优 |

**环境变量**：
```bash
CUDNN_LIB_CONFIG=AUTO    # 自动检测
CUDNN_LIB_CONFIG=FULL    # 强制FULL
CUDNN_LIB_CONFIG=JIT     # 强制JIT
```

---

## Kernel Cache 机制

### 默认 Kernel Cache

**特点**：
- 每个 CUDA 设备一个默认缓存
- 同设备所有 plan 共享
- 默认最大 100 MB
- LRU 驱逐策略

**启用条件**：
1. 未调用 `graph.set_kernel_cache(custom_cache)`
2. 环境变量 `CUDNN_ENABLE_DEFAULT_KERNEL_CACHE != 0`

**动态形状依赖**：
```
启用 dynamic shapes:
  → 缓存键基于拓扑（引擎+配置）
  → 不同形状可复用同一编译kernel

未启用 dynamic shapes:
  → 缓存键包含具体形状
  → 每种形状都需重新编译
```

### 环境变量控制

```bash
# 启用/禁用默认缓存
CUDNN_ENABLE_DEFAULT_KERNEL_CACHE=1  # 默认启用
CUDNN_ENABLE_DEFAULT_KERNEL_CACHE=0  # 禁用（每次重编译）

# 缓存大小
CUDNN_KERNEL_CACHE_SIZE_MB=100   # 默认100MB，LRU驱逐
CUDNN_KERNEL_CACHE_SIZE_MB=0     # 禁用缓存
CUDNN_KERNEL_CACHE_SIZE_MB=-1    # 无限制（小心内存）
```

---

## 最佳实践建议

### 1. 选择正确的 API

```
✅ 推荐：Frontend API (Python/C++)
  - 简化的编程模型
  - 自动属性推断
  - 更好的错误提示

❌ 避免：Backend API
  - 仅在需要遗留例程时使用
  - 需要纯 C 接口时使用
```

### 2. 布局选择

```
卷积：优先 NHWC
  - 更好的内存访问模式
  - Tensor Core 友好
  - 大多数现代 GPU 上更快

Matmul：根据操作链选择
  - 上游/下游操作的布局匹配
  - 减少 transpose 开销
```

### 3. 启发式策略

```
默认工作流：
1. 查询 Heuristics A
2. check_support() 第一个配置
3. 如果支持，直接使用（最快路径）
4. 如果都不支持，Fallback

生产部署：
- 首次运行时自动调优
- 缓存最优配置
- 后续直接使用
```

### 4. 内存管理

```
虚拟张量：
  - 中间结果标记为 virtual
  - 让 cuDNN 融合到单个 kernel
  - 避免不必要的内存读写

Workspace：
  - 查询所需大小：get_workspace_size()
  - 预分配并复用
  - 避免每次推理都分配
```

### 5. Attention 优化

```
推理场景：
  - generate_stats=False
  - 使用 paged attention（长序列）
  - 启用 kernel cache + dynamic shapes

训练场景：
  - generate_stats=True（必须）
  - use_deterministic_algorithm=True（可复现）
  - 考虑 GQA/MQA 减少 KV cache
```

---

## 性能优化 Checklist

- [ ] 使用 NHWC 布局（卷积）
- [ ] 启用操作融合（虚拟中间张量）
- [ ] 使用 Heuristics A 快速选择引擎
- [ ] 生产部署前自动调优
- [ ] 启用 Kernel Cache + Dynamic Shapes
- [ ] 预分配 workspace 并复用
- [ ] 使用 cuDNN JIT 减少二进制大小（如果可接受性能差异）
- [ ] 长序列使用 Paged Attention
- [ ] FP8 训练使用正确的 scale/descale
- [ ] 检查 compute capability 选择正确的融合引擎

---

## 架构支持矩阵

| 架构 | SM | Flash Attention | FP8 | 头维度上限 |
|------|----|--------------------|-----|----------|
| **Volta** | 7.0 | ❌ | ❌ | - |
| **Turing** | 7.5 | ❌ | ❌ | - |
| **Ampere** | 8.0/8.6 | ✅ | ❌ | Prefill:256, Decode:128 |
| **Ada** | 8.9 | ✅ | SM89+ | Prefill:256, Decode:128 |
| **Hopper** | 9.0 | ✅ | ✅ | 256 (d_qk=192, d_v=128) |
| **Blackwell** | 10.0 | ✅ | ✅ | B200/B300: 256, Consumer: 256 |

---

## 版本历史关键节点

| 版本 | 关键特性 |
|------|----------|
| **9.21** | SM100 Unfuse FMA 支持 |
| **9.19** | FP8 deterministic on Blackwell |
| **9.18** | Ampere/Ada THD/Ragged 支持 |
| **9.15** | Unified SDPA Padding mask |
| **9.13** | Unified SDPA 实现 |
| **9.5** | Paged Attention 支持 |
| **9.3** | Bottom-right causal mask |
| **9.1** | FP8 SDPA (Hopper+) |
| **9.0** | Sequence length 灵活性 |
| **8.9.6** | Padding/ALiBi/Bias mask |
| **8.9.3** | Flash Attention 首次支持 |

---

## 常见陷阱与解决

### 陷阱 1：忘记 generate_stats

```python
# ❌ 训练时会失败
o, stats = graph.sdpa(..., generate_stats=False)
dQ, dK, dV = graph.sdpa_backward(..., stats=stats)

# ✅ 正确
o, stats = graph.sdpa(..., generate_stats=True)
```

### 陷阱 2：缓存键不匹配

```python
# ❌ 每次都重新编译
graph.set_dynamic_shape_enabled(False)  # 缓存键包含形状

# ✅ 正确
graph.set_dynamic_shape_enabled(True)   # 缓存键仅拓扑
```

### 陷阱 3：Paged Attention backward 不支持

```python
# ❌ backward 不支持 paged
o, stats = graph.sdpa(q, k_paged, v_paged, ...)
dQ, dK, dV = graph.sdpa_backward(...)  # dK/dV 必须连续

# ✅ 正确：forward 用 paged，backward 用连续
```

### 陷阱 4：FP8 缺少 scale/descale

```python
# ❌ FP8 必须提供所有 scale
o, stats = graph.sdpa_fp8(q, k, v)

# ✅ 正确
o, stats = graph.sdpa_fp8(q, k, v,
    descale_q, descale_k, descale_v, descale_s,
    scale_s, scale_o)
```

---

## 总结

cuDNN 是 NVIDIA GPU 上 DNN 操作的**事实标准库**。通过：

1. **高度优化的 primitives**（卷积、matmul、attention 等）
2. **灵活的 Graph API**（支持多操作融合）
3. **智能的引擎选择**（启发式 + 自动调优）
4. **先进的优化技术**（Flash Attention、Paged Attention、FP8）

为深度学习框架提供极致的推理和训练性能。掌握 cuDNN 的核心概念和最佳实践，是优化 GPU 深度学习应用的关键。

---

## 新兴算子精读（Operations Reference 补充）

> 来源：https://docs.nvidia.com/deeplearning/cudnn/latest/operations/

### RoPE：作为一等后端算子而非 SDPA 属性

cuDNN 将 RoPE 实现为独立的后端描述符（`CUDNN_BACKEND_OPERATION_ROPE_FWD/BWD_DESCRIPTOR`），而不是塞进 SDPA 的参数里。好处：算子图里 `RoPE(Q) + RoPE(K) + SDPA(Q',K',V)` 可以被融合引擎识别为一个整体执行单元，围绕 attention kernel 运行 RoPE kernel，减少一次数据往返。

**核心数学（非交错/halved 旋转）**：

```
头维度 D 拆分为：
- nope 段（前 D-rope_dim 维）：仅做缩放直通  y_nope = α · x_nope
- rope 段（后 rope_dim 维）：分两半旋转
    y_lo = α · (x_lo·cosθ − x_hi·sinθ)
    y_hi = α · (x_hi·cosθ + x_lo·sinθ)

反向 = 按 −θ 旋转（正向旋转的转置），用 sin_sign=−1 复用同一 kernel
```

**freqs 张量设计**：kernel 直接吃原始角度 `freqs`（不是预计算好的 cos+sin），内部对每个位置只算一次 `sincosf`，存入 shared memory，在多个 head 间复用。形状 `[S,1,1,rope_dim]`，FP32。

**部分旋转配置**：像 DeepSeek-V3 MLA 里 `D=192` 但只有 `qk_rope_head_dim=64` 参与旋转的场景，用户传入较小的 `freqs`（`rope_dim=64`），剩下的 `D-rope_dim=128` 维走直通缩放路径。

### attn_scale 折叠进 RoPE 的技巧

标准做法：`attn_scale = 1/√D` 在 SDPA 内部乘到 `QKᵀ` 上。因为 RoPE 是线性变换，这个缩放可以提前挪到 RoPE 的 Q 输出里：

```
⟨α·R(θ)Q, R(θ)K⟩ = α·⟨R(θ)Q, R(θ)K⟩
```

**为什么值得做**：`S` 矩阵（attention scores）大小是 `O(B×H×S_q×S_kv)`，是整个 attention 里最大的中间张量。如果缩放留在 SDPA 内部，就要对这个最大张量做一次逐元素乘法；如果提前折进 RoPE，缩放只需要在 `O(B×H×S×D)` 的 Q/K 加载阶段完成一次，且是"顺路"完成（不增加额外的显存遍历）。

**反向传播的非对称性**：只对 Q 侧折叠 α 时，K 侧的反向梯度会通过链式法则"自动"带上正确的 α（因为 `dK_sdpa = dSᵀ · Q_sdpa`，而 `Q_sdpa` 已经携带了 α），不需要在 `RoPE_bwd_K` 上再显式设置。规则：**RoPE_bwd 和 RoPE_fwd 用同一个 output_scale**，剩下的交给链式法则处理。

**进阶：顺带折叠 inv_ln2**。GPU 硬件只有 `exp2`，softmax kernel 内部要做 `exp(z) = exp2(z · inv_ln2)`，这是对最大张量 S 的又一次逐元素乘法。同样可以把 `inv_ln2` 塞进 RoPE 的 `output_scale`（`α · inv_ln2`），配合 kernel 的 `softmax_input_is_log2` 标志直接用 `exp2`，省掉这次乘法。注意 `attn_scale` 本身不可省（softmax 不是尺度不变的），但 `inv_ln2` 只是换底的纯技术产物，可以消除。

### Block Scaling：MXFP8 / NVFP4 量化的分块粒度

| 方案 | 分块大小 | 输出精度 | scale 精度 |
|------|----------|----------|-----------|
| MXFP8 | 32 元素（沿行，可选沿列） | E4M3 或 E5M2 | E8M0（纯指数，无尾数） |
| NVFP4 | 16 元素（沿行） | E2M1 | E4M3 |

```
量化：scale = round_up(amax(block) / vmax_dtype)
      output = round_to_even(vals / scale)

反量化：output = vals × scale（scale 广播到整个 block）
```

这是 block scaling（微缩放/block-wise scaling）量化方案的核心 —— 相比 per-tensor 或 per-channel scale，block 级别的 scale 能更精细地适应数据分布，减少低精度带来的精度损失，是 FP8/FP4 训练和推理精度稳定性的关键机制。

### MoE Grouped Matmul：三种路由模式

```
None 模式（token 已按专家预排序）：
  Output[1,S×topK,N] = Token[1,S×topK,K] × Weight[E,K,N]

Gather 模式（从未路由布局中收集后再乘）：
  Output[1,S×topK,N] = Token[1,S,K] × Weight[E,K,N]   （用 token_index 指定源）

Scatter 模式（乘完后按原 token 顺序散射回去）：
  需要 token_index + token_ks（专家索引）+ top_k
```

`first_token_offset` 是贯穿三种模式的路由核心：一个 `[B×E,1,1]` 的 INT32 张量，第 i 个值是分配给专家 i 的第一个 token 的索引，靠它确定每个专家在 token 序列里的切片边界。

反向传播只算权重梯度 `dWeight = Tokenᵀ × dOutput`（按专家切片），不需要对 token 或 offset 求梯度——这与 MoE 训练里通常路由决策本身不参与梯度回传的直觉一致。

### Causal Conv1d：Mamba/状态空间模型的常见构件

```
y = activation(conv1d_causal(x, w) + b)

因果填充：左侧 (K-1) 个零，右侧 0 个零
depthwise：每个通道独立卷积，不跨通道混合
```

仅支持 Blackwell（SM100+），这是一个相对新的算子，说明 cuDNN 团队开始为 State Space Model（Mamba 等线性注意力替代架构）提供专门的融合原语，而不仅仅是 Transformer/Attention 生态。高层 Python API `cudnn.ops.causal_conv1d` 自动处理 autograd 和 `torch.compile` 集成，用法上更贴近 PyTorch 原生算子而不是传统 cuDNN 的显式 graph 构建流程。
