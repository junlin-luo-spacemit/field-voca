---
tags: [nvidia, cudnn, vocabulary]
domain: cudnn
last_updated: 2026-07-21
part: Overview + Core Concepts + Graph API ✅
source: https://docs.nvidia.com/deeplearning/cudnn/latest/
---

# cuDNN 词汇表

> 来源：https://docs.nvidia.com/deeplearning/cudnn/latest/
> 版本：cuDNN 9.x (Jul 2026)
> 格式：英文术语 → 中文译名 + 官方定义 + 例句

---

## 核心概念（Core Concepts）

### cuDNN (CUDA Deep Neural Network library) / cuDNN 深度神经网络库
NVIDIA 提供的 GPU 加速深度神经网络基础操作库。提供高度优化的卷积、矩阵乘法、归一化、注意力等 DNN 常见操作，并支持多操作融合优化。
> *The NVIDIA CUDA Deep Neural Network library (cuDNN) is a GPU-accelerated library of primitives for deep neural networks.*

### primitives / 基础操作（primitives）
DNN 中频繁出现的底层计算操作，如卷积、矩阵乘法、池化、归一化等。cuDNN 为这些操作提供高性能实现。
> *It provides highly tuned implementations of operations arising frequently in deep neural network (DNN) applications.*

### operation graph / 操作图
张量上操作构成的数据流图。用户将计算描述为操作图，cuDNN 从中选择最优引擎执行。
> *An operation graph is a dataflow graph of operations on tensors.*

### frontend API / 前端 API
cuDNN 提供的简化编程模型（Python/C++ 前端），封装了 backend API 的复杂性。大多数用例使用前端 API 即可。
> *The NVIDIA cuDNN frontend API provides a simplified programming model that is sufficient for most use cases.*

### backend API / 后端 API
cuDNN 的底层 C 接口。仅在需要使用前端未暴露的遗留固定函数例程或纯 C 接口时使用。
> *Use the NVIDIA cuDNN backend API only if you want to use the legacy fixed-function routines that are not graph-based interfaces.*

---

## 张量与布局（Tensors and Layouts）

### cuDNN handle / cuDNN 句柄
库上下文的句柄。使用 cuDNN 的应用必须初始化一个 handle。GPU 操作所需数据必须可从设备直接访问。
> *An application using cuDNN must initialize a handle to the library context.*

### tensor descriptor / 张量描述符
描述张量格式的 n 维描述符，包含：维度数（3-8）、数据类型（FP32/FP16/INT8 等）、每个维度的尺寸数组、每个维度的步幅（stride）数组。
> *The cuDNN library describes data with a generic n-D tensor descriptor.*

### stride / 步幅（stride）
从一个元素跳到同一维度下一个元素需要的元素数量。例如在 NCHW 中，C 维步幅为 H×W。
> *An integer array defining the stride of each dimension (the number of elements to add to reach the next element from the same dimension).*

### NCHW / 批次-通道-高度-宽度 布局
4D 张量的标准布局：先排列所有通道、再按行主序排列空间维度、最后处理批次。步幅：[C×H×W, H×W, W, 1]。
> *Beginning with the first channel (c=0), the elements are arranged contiguously in row-major order.*

### NHWC / 批次-高度-宽度-通道 布局
4D 张量的另一种布局：对每个空间位置，先排列所有通道。步幅：[H×W×C, W×C, C, 1]。在许多 GPU 上性能更优。
> *For the NHWC memory layout, the corresponding elements in all the C channels are laid out first.*

### NC/32HW32 / 分组通道布局
将通道维度分组（每组 32 个通道），每组使用 NHWC 布局。用于 INT8 量化卷积的向量化加载。
> *The NC/32HW32 is similar to NHWC, with a key difference. The 64 channels are grouped into two groups of 32 channels each.*

### fully-packed tensor / 完全紧凑张量
张量被定义为 XYZ-fully-packed 当且仅当：张量维度数等于字母数、第 i 维步幅等于第 (i+1) 维×其步幅、最后一维步幅为 1。
> *A tensor is defined as XYZ-fully-packed if the stride of the i-th dimension is equal to the product of the (i+1)-th dimension by the (i+1)-th stride.*

### partially-packed tensor / 部分紧凑张量
在更高维张量中，部分维度是紧凑的。例如 NHWC 中的 WC-packed 表示 c_stride=1，w_stride=c_dim×c_stride。
> *A WXYZ tensor is defined as XYZ-packed if the stride of each dimension referenced in the -packed suffix is equal to the product of the (i+1)-st dimension by the (i+1)-st stride.*

### overlapping tensor / 重叠张量
遍历所有维度时同一地址被访问多次。实际中 stride[i-1] < stride[i]×dim[i]。cuDNN 对输入张量支持重叠维度，但输出张量不能重叠。
> *A tensor is defined to be overlapping if iterating over a full range of dimensions produces the same address more than once.*

### virtual tensor / 虚拟张量
图中间张量，不占用实际内存。cuDNN 可将虚拟张量融合到单个 kernel 中，避免内存读写。
> *All the intermediate tensors must be virtual.*

### ragged tensor / 不规则张量（ragged layout）
变长序列紧凑打包（无填充）的布局。使用 THD 格式，T=Σ(seq_len) 是总有效 token 数。需设置 ragged offset 张量。
> *For memory efficiency, variable-length sequences can be packed together without padding. This is called THD layout.*

### ragged offset / 不规则偏移
形状为 (B+1,1,1,1) 的张量，包含累积 token 偏移（以元素为单位）。最后一个元素是总 token 数。
> *Shape: (B+1,1,1,1). Contains cumulative token offsets in elements (not bytes). Last element is the total number of tokens.*

---

## 图 API 工作流（Graph API Workflow）

### Graph / 图（Graph 对象）
cuDNN 前端的最高级接口，用于容纳张量和操作。可设置全局属性（IO 数据类型、中间数据类型、计算数据类型）。
> *Instantiate an object of class cudnn_frontend::graph::Graph which will house tensors and operations.*

### Tensor_attributes / 张量属性
描述张量的轻量结构，包含 setters：数据类型、维度、步幅、是否虚拟、是否按值传递、重排序类型、名称。
> *Tensor attributes is a lightweight structure with setters for each attribute.*

### operation / 操作（Operation）
通过位置参数接收必需输入张量，通过 setters 提供可选输入。返回有序输出张量数组。
> *Operations take in mandatory input tensors via positional arguments. Optional input tensors are provided using corresponding setters.*

### validate() / 校验图
确保 API 使用正确、无悬空张量、推断未指定的属性（维度、步幅等）。
> *The validate API ensures the API usage is sound, checks against dangling tensors, and so on.*

### build_operation_graph() / 构建后端图
为图的所有组成部分创建 cuDNN backend descriptors。
> *This method creates the cuDNN backend descriptors for all constituents of the graph.*

### execution plan / 执行计划
查询启发式算法返回的引擎配置列表。每个 plan 有索引 0 到 #plans-1，0 优先级最高。
> *This method internally queries the heuristics for engine configs for the given heuristics modes.*

### engine / 引擎
能够执行给定操作图的候选实现。一个图可能有多个引擎，通过启发式查询获得候选列表。
> *For a given operation graph, there are some number of engines that are candidates for implementing that graph.*

### engine configuration / 引擎配置
引擎可配置的旋钮（knobs）。专家用户可根据引擎属性（数值注释、行为注释、可调旋钮）过滤引擎配置。
> *An engine has knobs for configuring the engine.*

### heuristics / 启发式算法
获取引擎配置列表的方法，按预期性能从高到低排序。有三种模式：A（快速，处理大多数模式）、B（更准确但 CPU 延迟更高）、Fallback（快速功能回退）。
> *A heuristic is a way to get a list of engine configurations that are intended to be sorted from the most performant to least performant.*

### heuristics mode A / 启发式模式 A
快速且能处理大多数操作图模式。返回按预期性能排序的引擎配置列表。
> *Intended to be fast and be able to handle most operation graph patterns.*

### heuristics mode B / 启发式模式 B
比模式 A 更准确，但 CPU 延迟更高。在已知 A 能做得更好的情况下可能回退到 A。
> *Intended to be more generally accurate than mode A, but with the tradeoff of higher CPU latency.*

### fallback heuristics mode / 回退启发式模式
快速提供功能回退，不期望达到最优性能。当所有引擎配置不支持时使用。
> *Intended to be fast and provide functional fallbacks without expectation of optimal performance.*

### check_support() / 检查支持
保证使用查询到的 plan 执行图会成功。建议先查询 A 或 B 模式并检查支持，第一个支持的引擎配置预期性能最佳。
> *This method guarantees that executing the graph using plans queried will succeed.*

### build_plan() / 构建执行计划
构建通过 create_execution_plan() 查询的执行计划。两种方式：按策略构建（让 cuDNN 选最佳）或按索引构建（用于并行自动调优）。
> *This function builds execution plans queried with the create_execution_plan(...) API.*

### autotuning / 自动调优
为给定图执行不同执行计划并测量运行时性能。通常有助于验证和改进启发式结果。
> *Autotuning provides a way to execute different execution plans for a given graph and measure your relative performance under runtime conditions.*

### execute() / 执行图
执行图需要所有输入输出张量的设备指针和用户分配的设备 workspace 指针。
> *Executing the graph requires device pointers to all input output tensors and a user-allocated device workspace pointer.*

### workspace / 工作空间
用户分配的设备内存，cuDNN 用于中间计算。通过 get_workspace_size() 查询所需大小。
> *Use get_workspace to execute the currently selected execution plan.*

### numerical note / 数值注释
告知用户引擎的数值属性，如是否在输入或输出归约时进行数据类型向下转换。
> *Numerical notes inform the user about the numerical properties of the engine.*

### behavior note / 行为注释
提示底层实现的特性，如是否使用运行时编译。
> *The behavior notes can signal something about the underlying implementation like whether or not it uses runtime compilation.*

---

## 融合模式（Fusion Patterns）

### operation fusion / 操作融合
将多个操作融合到单个或少数 kernel 中执行，减少内存传输，提高性能。cuDNN 支持多种预定义融合模式。
> *Beyond just providing high-performance implementations of individual operations, cuDNN also supports a flexible set of multi-operation fusion patterns.*

### runtime fusion engine / 运行时融合引擎
在运行时根据图模式生成 kernel 的引擎。支持通用融合模式，灵活性高。带有 CUDNN_BEHAVIOR_NOTE_RUNTIME_COMPILATION 行为注释。
> *cuDNN has runtime fusion engines that generate the kernel (or kernels) at runtime based on the graph pattern.*

### pre-compiled engine / 预编译引擎
针对特定模式预先编译的引擎。不需要运行时编译，但支持的模式有限。
> *The pre-compiled specialized engines target and optimize for a specialized graph pattern.*

### generic runtime fusion engine / 通用运行时融合引擎
支持通用模式的运行时融合引擎。覆盖 Matmul/Convolution 融合、Pointwise/Reduction 融合，支持多种 compute capability。
> *The generic runtime fusion engine includes three independent support surfaces indexed as 90, 80, and 70.*

### specialized runtime fusion engine / 专用运行时融合引擎
针对常见模式（NormAddRelu、Fused Flash Attention 等）的运行时编译引擎。支持面有限但高度优化。
> *The specialized runtime fusion engines target and optimize specialized graph patterns that commonly occur in popular deep learning models.*

### mainloop fusion / 主循环融合（g₁）
应用到 matmul/convolution 操作输入的融合操作。例如在 Matmul 前对 A、B 输入应用 pointwise 操作。
> *g₁ indicates fusion operations applied to the inputs of the matmul and convolution operation.*

### epilogue fusion / 后处理融合（g₂）
应用到 matmul/convolution 操作输出的融合操作。例如在 Matmul 后应用 pointwise、reduction 操作。g₂ 可有多个输出。
> *g₂ indicates fusion operations apply to the output of the matmul and convolution operation.*

### ConvolutionFwd / 前向卷积
计算 Y=ConvolutionFwd(X,W)，即图像张量与滤波器张量的卷积响应。
> *This operation represents forward convolution, that is, computing the response tensor of image tensor convoluted with filter tensor.*

### ConvolutionBwdData / 卷积数据梯度
从响应和滤波器张量计算输入数据梯度，即 dX=ConvolutionBwdData(dY,W)。
> *This operation represents convolution backward data, that is, computing input data gradients from a response and a filter tensor.*

### ConvolutionBwdFilter / 卷积滤波器梯度
从响应和图像张量计算滤波器梯度，即 dW=ConvolutionBwdFilter(dY,X)。
> *This operation represents convolution backward filters, that is, computing filter gradients from a response and an image tensor.*

### Pointwise / 逐点操作
实现 Y=op(α₁×X) 或 Y=op(α₁×X, α₂×B) 的操作。包括 scale、bias、add、mul、ReLU、tanh、sigmoid 等。
> *Represents a pointwise operation that implements the equation Y = op(alpha1 * X) or Y = op(alpha1 * X, alpha2 * B).*

### Reduction / 归约操作
在一个或多个维度上归约张量值。支持的归约算子：ADD、MIN、MAX。
> *This operation represents reducing values of a tensor in one or more dimensions.*

### GenStats / 生成统计
生成每通道统计信息。用于批归一化等操作。
> *Represents an operation that generates per-channel statistics.*

### Matmul / 矩阵乘法
A×B=C 的矩阵-矩阵乘法。使用 3D 张量（BMN 维度）。
> *This operation represents matrix-matrix multiplication: A * B = C.*

---

## Fused Flash Attention

### Scaled Dot Product Attention (SDPA) / 缩放点积注意力
计算 Attention(Q,K,V)=softmax(QKᵀ/√d)V，使用 FlashAttention-2 算法。适用于训练和推理。
> *This operation computes the scaled dot product attention (SDPA) using the FlashAttention-2 algorithm.*

### FlashAttention-2 / FlashAttention-2 算法
SDPA 的高效实现算法，通过分块计算和融合操作减少内存访问。cuDNN 的 Fused Flash Attention 基于此算法。
> *It is applicable for both training and inference phases using the FlashAttention-2 algorithm.*

### stats tensor / 统计张量
前向传播中生成的 softmax 统计信息（Max、Sum）。训练时必须生成，用于反向传播计算。
> *An option to generate a stats tensor to be used for backwards training computation.*

### causal mask / 因果掩码
将注意力矩阵的上三角填充为负无穷，确保位置 i 只能注意到 ≤i 的位置。用于自回归模型（GPT 等）。
> *Fills the upper triangular matrix of attention scores with negative infinity.*

### padding mask / 填充掩码
掩盖填充的时间步使其在计算中被忽略。需传递每批次的序列长度张量。
> *Masks out padded time steps to ignore them in computation.*

### sliding window attention / 滑动窗口注意力
限制每个位置只能注意到窗口范围内的其他位置。通过 left_bound 和 right_bound 控制。
> *Limits each position to attend only to other positions within a window range.*

### ALiBi (Attention with Linear Biases) / 线性偏置注意力
在注意力分数上应用线性偏置的加性掩码，用于位置编码。使用 ALiBi 时必须设置因果掩码。
> *Attention with Linear Biases (ALiBi) is an additive mask applied to the attention scores.*

### GQA (Grouped Query Attention) / 分组查询注意力
K、V 的头数是 Q 头数的因子。允许 H_k 和 H_v 可以是 H_q 的因子，减少 KV cache 内存。
> *The second dimension in K^T corresponding to the number of heads can be a factor of the number of heads of Q.*

### MQA (Multi-Query Attention) / 多查询注意力
GQA 的特例，所有查询头共享单个 K、V 头（H_k=H_v=1）。
> *All attention flavors MHA, MQA, GQA are supported.*

### paged attention / 分页注意力
非连续 K/V 缓存以减少内存碎片。使用 page_table 指定块偏移。
> *Enables non-contiguous K/V caches to reduce memory fragmentation.*

### dropout / Dropout
softmax 后随机将部分注意力权重置零作为正则化。两种方式：Philox RNG dropout（更高性能）或自定义 dropout 掩码。
> *Randomly zeros some of the attention weights after the softmax as a form of regularization.*

### attention scale / 注意力缩放
应用到注意力分数的缩放因子，通常为 1/√d。在 softmax 前应用。
> *Applies a scaling factor to attention scores before the softmax, such as 1/√d.*

### BHSD layout / 批次-头-序列-维度 布局
注意力张量的默认逻辑布局，维度顺序为 (B,H,S,D)。步幅：[H×S×D, S×D, D, 1]。
> *The default logical layout where dimensions are ordered as (B,H,S,D).*

### BSHD layout / 批次-序列-头-维度 布局
物理布局，序列在内存中位于头之前，但逻辑维度顺序仍为 (B,H,S,D)。步幅：[S×H×D, D, H×D, 1]。
> *A physical layout where sequence comes before heads in memory, while maintaining the logical (B,H,S,D) dimension order.*

---

## 归一化（Normalizations）

### NormalizationForward / 归一化前向
从输入 X 计算归一化输出 Y。用于推理和训练阶段。支持的归一化模式：LayerNorm、RMSNorm、AdaLayerNorm。
> *NormalizationForward computes the normalization output Y from the input X.*

### NormalizationBackward / 归一化反向
计算梯度 dX 以及 scale 和 bias 梯度 dScale、dBias。使用前向传播保存的 mean 和 variance。
> *NormalizationBackward computes the gradient dX and the scale and bias gradients dScale and dBias.*

### LayerNorm / 层归一化
在最后 D' 维度上计算均值和标准差的归一化模式。例如 [N,C,H,W] 中在 C,H,W 上归一化。
> *CUDNN_LAYER_NORM: Mean and standard deviation are calculated over the last D' dimensions.*

### RMSNorm / 均方根归一化
类似 LayerNorm 但仅使用 RMS（不减去均值）。不输出 Mean 张量。
> *CUDNN_RMS_NORM: Similar to LayerNorm but only uses RMS (does not subtract mean).*

### AdaLayerNorm / 自适应层归一化
LayerNorm 的变体，用于条件生成模型。需要 Scale 和 Bias 输入（非可选）。
> *CUDNN_ADA_LAYER_NORM: A variant of LayerNorm used in conditional generation models.*

---

## cuDNN JIT

### cuDNN JIT / cuDNN 即时编译配置
cuDNN 子库配置，仅支持通过 graph API 的运行时融合引擎，显著减少下载和安装的二进制大小。
> *cuDNN JIT is a configuration of cuDNN sub-libraries that provides a significant reduction in the download and installed binary sizes.*

### cuDNN FULL / cuDNN 完整配置
提供完整 cuDNN 功能，包括所有预编译 kernel。需要所有 cuDNN 库。
> *Provides full usage of cuDNN functionality, including all precompiled kernels.*

### CUDNN_LIB_CONFIG / cuDNN 运行时配置环境变量
确定加载哪些子库以及如何提供 cuDNN 功能。可选值：AUTO（默认）、FULL、JIT。
> *The cuDNN runtime configuration determines which sub-libraries are loaded and how cuDNN features are made available.*

### runtime compilation / 运行时编译
在运行时根据图模式生成和编译 kernel。cuDNN JIT 仅支持运行时编译引擎。
> *Support is limited to runtime fusion engines through the graph API (no precompiled kernels).*

---

## Kernel Cache

### kernel cache / 内核缓存
存储已编译 CUBIN 的缓存，使后续具有相同缓存键的 plan 可重用而无需重新编译，显著减少 plan 构建时间。
> *The kernel cache stores compiled CUBINs so that later plans with the same cache key can reuse them instead of recompiling.*

### default kernel cache / 默认内核缓存
每个 CUDA 设备一个默认缓存。同一设备上的所有 plan 和 handle 共享该缓存。缓存不跨设备共享。
> *When the default kernel cache is used, one default cache exists per CUDA device (by device ordinal).*

### LRU eviction / 最近最少使用驱逐
当缓存达到最大大小时，移除最近最少使用的条目。默认最大缓存大小为 100 MB。
> *When the cache reaches this size, cuDNN uses LRU eviction to remove the least-recently-used entries.*

### dynamic shapes / 动态形状
启用动态形状后，缓存键基于拓扑（相同引擎和引擎配置）而非具体形状。必须启用才能缓存运行时编译 kernel。
> *If dynamic shapes are enabled, matching is topology-based (same engine and engine configuration).*

### CUDNN_ENABLE_DEFAULT_KERNEL_CACHE / 启用默认缓存环境变量
控制默认缓存是否启用。0=禁用，1 或未设置=启用。禁用时 RTC 引擎每次使用都会重新编译。
> *Specifies whether the default cache is enabled or disabled.*

### CUDNN_KERNEL_CACHE_SIZE_MB / 缓存大小环境变量
指定每设备默认缓存的最大大小（MB）。正值设置最大值并启用 LRU 驱逐；0=禁用；负值=无限制。默认 100 MB。
> *Specifies the maximum size of the default cache per device in MB and controls LRU eviction behavior.*

---

## 其他操作（Other Operations）

### Convolution / 卷积
计算图像与滤波器的卷积。支持前向、数据梯度、滤波器梯度。可使用 α、β 缩放因子混合结果。
> *ConvolutionFwd computes the convolution of X with filter data W.*

### Pooling / 池化
空间维度的下采样操作。支持 MaxPool、AvgPool（包含/排除填充）。
> *Resampling is supported in both directions, upsampling and downsampling.*

### Batch Normalization / 批归一化
跨批次和空间维度归一化。支持前向和反向，可选 per-GPU 统计。
> *Batch normalization forward and backward.*

### Attention / 注意力
缩放点积注意力（SDPA），支持 FP16/BF16/FP8。包含多种掩码、dropout、分页注意力等选项。
> *Scaled dot product attention (SDPA) with FlashAttention-2 algorithm.*

### Matmul / 矩阵乘法
批量矩阵乘法。支持 A×B=C，可选 Packed Row-major 或 Column-major 布局。
> *Matrix-matrix multiplication with 3D tensors (BMN dimensions).*

### Resample / 重采样
空间维度的重采样（上采样或下采样）。包括 Pooling（下采样）和插值（上采样，如 bilinear）。
> *Resampling of the spatial dimensions of an image to a desired value.*

### Concat / 拼接
沿指定维度拼接多个张量。
> *Concatenate multiple tensors along a specified dimension.*

### Reshape / 重塑
改变张量形状而不改变数据。
> *Change tensor shape without changing data.*

### Slice / 切片
提取张量的子区域。
> *Extract a sub-region of a tensor.*

### Transpose / 转置
交换张量的维度。
> *Swap dimensions of a tensor.*

---

## 新兴算子（Emerging Operations）

### RoPE (Rotary Position Embedding) / 旋转位置编码
cuDNN 的一等后端操作（非 SDPA 属性），对 Q/K 张量按位置角度进行旋转变换。算子图中 RoPE(Q)+RoPE(K)+SDPA(Q',K',V) 会被 SDPA 融合引擎识别并作为单个执行单元围绕 attention kernel 运行。
> *RoPE is a first-class backend operation, not an SDPA attribute. The SDPA fusion engine pattern-matches and runs the RoPE kernels around the attention kernel as a single execution unit.*

### rope_dim / 旋转维度
头维度 D 中参与旋转的段长度。前 D-rope_dim 部分为 nope（不旋转）段，仅做缩放直通；用于 DeepSeek-V3 MLA 等部分旋转配置（如 D=192 中仅 64 维参与旋转）。
> *For a partial-rotation config, the user supplies a smaller freqs tensor and sets rope_dim; the leading D-rope_dim positions become a scaled passthrough.*

### output_scale (RoPE) / 输出缩放因子
RoPE 前向核对旋转后两半和 nope 直通段统一施加的缩放系数 α。可用于把 SDPA 的 attn_scale（1/√D）折叠进 RoPE，减少对 S 矩阵（O(BHS²)）的逐元素乘法，改为对 Q/K（O(BHSD)）的乘法，从而降低计算量。
> *Because RoPE is linear, the scale can be hoisted into the RoPE Q output without changing the math.*

### freqs tensor / 频率张量
RoPE 核直接接收的原始角度张量（非预计算的 cos+sin），形状 [S,1,1,rope_dim]，FP32。核内部对每个位置执行一次 sincosf 计算并存入共享内存，在多头间复用。
> *The kernel takes raw freqs (angles), not pre-computed cos+sin, computing sincosf(freqs) once per position amortized across heads.*

### inv_ln2 folding / 自然对数换底折叠
将 softmax 核内部 exp↔exp2 换底所需的 1/ln(2) 常数折叠进 RoPE 的 output_scale，避免对最大的中间张量 S 做额外逐元素乘法。attn_scale（控制 softmax 形状）不可折叠，但 inv_ln2（纯换底伪影）可以。
> *softmax is not scale-invariant, so attn_scale stays mandatory. inv_ln2 is a base-conversion artifact, removable only because the kernel switches exp↔exp2.*

### Block Scale Quantize / 块缩放量化
计算量化输出和缩放因子张量。MXFP8 方案：沿行（可选列）对 32 个 FP32 元素分块量化，产出 32 个 FP8 值（E4M3/E5M2）+ 1 个 FP8 缩放因子（E8M0）。NVFP4 方案：16 个 FP32 元素产出 16 个 FP4 值（E2M1）+ 1 个 E4M3 缩放因子。
> *The MXFP8 recipe quantizes across 32 FP32 elements to produce 32 FP8 output values and 1 FP8 scaling factor (E8M0). The NVFP4 recipe quantizes across 16 elements to produce 16 FP4 values and 1 E4M3 scaling factor.*

### Block Scale Dequantize / 块缩放反量化
从量化输入和缩放张量计算反量化输出：output = dequantize(vals × scale)，scale 广播到块大小。
> *The block scale dequantize operation computes the dequantized output tensor from quantized input and scale tensors.*

### MoE Grouped Matmul / 混合专家分组矩阵乘法
计算跨专家的分组矩阵乘法，每个专家有自己的权重矩阵，token 通过 first_token_offset 路由到对应专家。支持三种路由模式：None（已预路由）、Gather（从未路由布局收集后计算）、Scatter（计算后按原 token 顺序散射回去）。
> *Each expert has its own weight matrix, and tokens are routed to experts via first_token_offset. Three routing modes are supported: None, Gather, and Scatter.*

### first_token_offset / 首 token 偏移
INT32 张量，形状 [B×E,1,1]，第 i 个值是分配给专家 i 的首个 token 的索引，用于确定每个专家在 token 序列中的切片边界。
> *FirstTokenOffset contains B*E values; the i-th entry is the index of the first token assigned to expert i.*

### top_k (MoE) / 专家路由数
每个 token 被路由到的专家数量。Scatter 模式下必须显式提供，用于散射回写计算。
> *TopK: the number of experts each token is routed to. Required in Scatter mode for the scatter-back computation.*

### Causal Conv1d / 因果一维卷积
深度可分离的因果一维卷积，带可选融合激活：y=activation(conv1d_causal(x,w)+b)。因果填充为左侧 (K-1) 个零、右侧 0 个零，每个通道独立卷积（depthwise）。支持 Blackwell (SM100+)，identity/silu 激活。
> *Depthwise causal 1-D convolution with optional fused activation. Causal padding: (K-1) zeros on the left, 0 on the right. Each channel is convolved independently (depthwise).*

---

## 来源文档

- cuDNN Overview: https://docs.nvidia.com/deeplearning/cudnn/latest/
- Developer Guide - Overview: https://docs.nvidia.com/deeplearning/cudnn/latest/developer/overview.html
- Developer Guide - Core Concepts: https://docs.nvidia.com/deeplearning/cudnn/latest/developer/core-concepts.html
- Developer Guide - Graph API: https://docs.nvidia.com/deeplearning/cudnn/latest/developer/graph-api.html
- Operations - Attention: https://docs.nvidia.com/deeplearning/cudnn/latest/operations/Attention.html
