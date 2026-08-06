---
tags: [nvidia, dynamo, notes]
domain: dynamo
last_updated: 2026-07-21
source: https://docs.nvidia.com/dynamo/ | https://github.com/ai-dynamo/dynamo
---

# NVIDIA Dynamo 精读笔记

> Dynamo v1.2.1 | 文档更新 2026

---

## 一句话总结

Dynamo = **数据中心级推理编排层**。不做推理计算本身，而是协调 SGLang/TRT-LLM/vLLM 等引擎跨多节点、多 GPU 协同工作：分解式 prefill/decode、KV 感知路由、SLA 驱动自动扩缩容。

---

## Dynamo 的定位：编排层，不是推理引擎

```
常见误解：Dynamo 是另一个推理引擎（像 TRT-LLM/vLLM 一样）
实际情况：Dynamo 是运行在这些引擎「之上」的协调层

单 GPU / 单节点场景：
  推理引擎自己就够了（TRT-LLM/vLLM/SGLang）
  不需要 Dynamo

多 GPU / 多节点场景：
  需要协调 KV cache 路由、独立扩展 prefill/decode、
  自动扩缩容满足 SLA → 这就是 Dynamo 的价值
```

**何时使用 Dynamo**（官方判断标准）：
- 跨多 GPU/多节点服务 LLM，需要协调
- 需要 KV 感知路由避免重复 prefill 计算
- 需要独立扩展 prefill 和 decode（分解式服务）
- 需要满足延迟 SLA 且最小化 TCO 的自动扩缩容
- 需要新副本快速冷启动

---

## 整体架构

```
                    客户端请求
                        │
                        ▼
        ┌───────────────────────────────┐
        │   Dynamo Frontend              │
        │   (OpenAI-compatible HTTP API) │
        │   端口 8000                     │
        └───────────────────────────────┘
                        │
                        ▼
        ┌───────────────────────────────┐
        │   Dynamo Router                │
        │   - KV 感知路由决策              │
        │   - Flash Indexer (全局KV索引)  │
        │   - 负载均衡                    │
        └───────────────────────────────┘
                        │
            ┌───────────┴───────────┐
            ▼                       ▼
    ┌───────────────┐      ┌───────────────┐
    │ Prefill Pool  │      │ Decode Pool   │
    │ (计算密集型GPU) │      │ (带宽密集型GPU) │
    │ SGLang/TRT-LLM│      │ SGLang/TRT-LLM│
    │ /vLLM worker  │      │ /vLLM worker  │
    └───────────────┘      └───────────────┘
            │                       │
            └───────────┬───────────┘
                        ▼
            ┌───────────────────────┐
            │   KVBM                │
            │   GPU→CPU→SSD→远程存储 │
            └───────────────────────┘

后台协调组件：
├─ Planner   ← SLA驱动自动扩缩容
├─ Grove     ← K8s拓扑感知调度
├─ ModelExpress ← 权重流式传输（冷启动加速）
└─ AIConfigurator ← 部署配置模拟
```

---

## 分解式服务（Disaggregated Serving）详解

### 为什么要拆分 Prefill 和 Decode？

```
Prefill（预填充）特性：
- 处理整个输入 prompt
- 计算密集（compute-bound）
- 大矩阵乘法，GPU 算力是瓶颈
- 适合：高算力 GPU（如 H100/B200 的 FLOPS）

Decode（解码）特性：
- 逐 token 自回归生成
- 内存带宽密集（memory-bound）
- 需要频繁读取 KV cache
- 适合：高显存带宽 GPU

传统聚合式服务的问题：
同一 GPU 池处理两种不同特性的负载
→ 资源利用不均衡，要么算力浪费，要么带宽浪费
```

### 分解式架构收益

```
聚合式（Aggregated）：
[GPU Pool] ← 处理 prefill + decode 混合负载
问题：GPU 配置无法同时最优适配两种负载

分解式（Disaggregated）：
[Prefill Pool] ← 专用高算力 GPU，独立扩展
[Decode Pool]  ← 专用高带宽 GPU，独立扩展
优势：
- 每种硬件按其特性发挥最大效率
- Prefill 和 Decode 可根据实际负载比例独立扩缩容
- 例如：长文档摄入场景多加 Prefill worker
       长对话生成场景多加 Decode worker
```

### KV Cache 在 Prefill→Decode 间的传输

```
Prefill worker 计算完成
    │
    ▼
生成 KV cache（Key/Value 张量）
    │
    ▼
通过 NIXL/NVLink/RDMA 传输到 Decode worker
    │
    ▼
Decode worker 使用该 KV cache 继续生成
```

**Topology-Aware KV Transfer**：将该传输限制在同一拓扑域（如同机架）内，减少跨域网络延迟。

---

## KV 感知路由（KV-Aware Routing）

### 核心问题：重复 Prefill 计算

```
场景：多用户共享相同的系统提示词（system prompt）

用户 A 请求："[系统提示词 500 tokens] 用户问题1"
用户 B 请求："[系统提示词 500 tokens] 用户问题2"

如果路由到不同 worker：
- Worker 1 计算 A 的 prefill（含 500 tokens 系统提示词）
- Worker 2 计算 B 的 prefill（含相同 500 tokens 系统提示词，重复！）

KV 感知路由：
- Flash Indexer 追踪每个 worker 已缓存的 KV block
- Router 发现 Worker 1 已有该系统提示词的 KV cache
- B 的请求也路由到 Worker 1，复用已缓存的 500 tokens
- 结果：B 只需计算新增的用户问题部分，TTFT 大幅降低
```

### Flash Indexer 性能

```
"Inter-Galactic KV Routing" 技术深度剖析：
- 追踪集群内所有 worker 的每个缓存 KV block
- 支持 > 100M ops/sec 的并发查询
- 经过 6 次迭代设计演化达到该性能水平
- 使 KV 感知路由在超大规模集群仍保持低延迟决策
```

---

## SLA 驱动的 Planner

### 零配置部署（DGDR）

```yaml
apiVersion: nvidia.com/v1beta1
kind: DynamoGraphDeploymentRequest
metadata:
  name: my-model
spec:
  model: Qwen/Qwen3-0.6B
  backend: vllm
  sla:
    ttft: 200.0   # 首token延迟目标（毫秒）
    itl: 20.0     # token间延迟目标（毫秒）
  autoApply: true
```

**背后的自动化流程**：
```
1. 用户只需声明模型 + 硬件 + SLA
        │
        ▼
2. AIConfigurator 自动分析工作负载特性
   （模拟数千种部署配置组合）
        │
        ▼
3. Planner 根据分析结果优化拓扑
   （决定 Prefill:Decode worker 比例）
        │
        ▼
4. Dynamo 自动部署该拓扑
        │
        ▼
5. 运行时持续监控，若 SLA 违反则自动调整
```

### 实测效果（官方数据）

| 指标 | 提升幅度 | 场景 |
|------|---------|------|
| 每 GPU 吞吐量 | 7x | DeepSeek R1 on GB200 NVL72 |
| 模型启动速度 | 7x | ModelExpress 权重流式传输 |
| 首 token 延迟 | 2x 更快 | KV 感知路由，Qwen3-Coder 480B |
| SLA 违反率 | 减少 80% | Planner 自动扩缩容，TCO 降低 5% |
| 吞吐量 | 750x | DeepSeek-R1 on GB300 NVL72 |

---

## 快速上手

### 容器方式（最快）

```bash
# 拉取 SGLang runtime 容器
docker run --gpus all --network host --rm -it \
  nvcr.io/nvidia/ai-dynamo/sglang-runtime:1.2.1

# 容器内：启动 Frontend
python3 -m dynamo.frontend --http-port 8000 --discovery-backend file > /dev/null 2>&1 &

# 容器内：启动 Worker
python3 -m dynamo.sglang --model-path Qwen/Qwen3-0.6B --discovery-backend file &

# 测试
curl -s localhost:8000/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "Qwen/Qwen3-0.6B",
    "messages": [{"role": "user", "content": "Hello!"}],
    "max_tokens": 100
  }'
```

### PyPI 方式

```bash
# 安装 uv
curl -LsSf https://astral.sh/uv/install.sh | sh

# 安装 Dynamo（选择后端）
uv pip install --prerelease=allow "ai-dynamo[sglang]"   # 或 [vllm]

# 启动同上
```

### 三种后端支持

| 后端 | 容器镜像 | 特点 |
|------|---------|------|
| **SGLang** | `sglang-runtime:1.2.1` | RadixAttention，适合 Agent 场景 |
| **TensorRT-LLM** | `tensorrtllm-runtime:1.2.1` | NVIDIA 最优化，最高吞吐 |
| **vLLM** | `vllm-runtime:1.2.1` | PagedAttention，生态最广 |

---

## 请求路由拓扑

### 拓扑 1：Dynamo 原生路由

```
client → Frontend → Router → workers

特点：
- Dynamo 完全掌控请求入口
- 无需外部网关
- 适合：本地开发、单集群部署
```

### 拓扑 2：Gateway API 路由（GAIE）

```
client → Gateway → EPP → Frontend sidecar (direct) → workers

特点：
- 标准 Kubernetes Gateway API
- 网关层可做统一鉴权、限流、可观测性
- EPP (Endpoint Picker Plugin) 复用 Dynamo 路由智能
- 适合：已标准化 Gateway API 的 K8s 平台
```

两种拓扑都支持相同的：OpenAI 兼容 API、分解式服务、KV 感知路由。

---

## 容错机制

```
故障检测：
Canary Health Check ── 持续探测 worker 健康状态
        │
        ▼ 发现故障
故障处理：
In-flight Request Migration ── 将进行中请求迁移到健康 worker
        │
        ▼
用户侵扰最小化：
"Workers fail; user requests don't."
```

**优雅关闭流程**：
```
接收到关闭信号
    │
    ▼
停止接受新请求
    │
    ▼
完成或迁移进行中的请求
    │
    ▼
释放 GPU 资源
```

---

## Agentic 推理优化

Dynamo 1.0 新增的三层 Agent 优化：

```
1. Frontend API 层
   - Per-request hints（优先级、预期输出长度、推测预填充）
   - Session metadata（追踪多轮对话）

2. Router 层
   - Priority Scheduling（优先级调度）
   - SGLang subagent KV isolation（子 Agent KV 隔离）

3. KV Cache 管理层
   - 多轮对话上下文的 KV 复用
   - Tool calling 结果不污染主对话 KV
```

**集成生态**：LangChain + NeMo Agent Toolkit 原生支持 Dynamo 作为推理后端。

---

## Dynamo vs 直接使用推理引擎

| 维度 | 直接用 vLLM/TRT-LLM | + Dynamo |
|------|---------------------|----------|
| 单 GPU 场景 | ✅ 足够 | 无需引入 |
| 多节点协同 | ❌ 需自己实现 | ✅ 原生支持 |
| KV 感知路由 | ❌ | ✅ Flash Indexer |
| 分解式 P/D | ❌ 引擎内不支持跨节点 | ✅ |
| SLA 自动扩缩容 | ❌ 需自己写脚本 | ✅ Planner |
| 冷启动速度 | 标准 | 7x（ModelExpress） |
| 故障容错 | 引擎自身机制 | ✅ 请求迁移 |
| Agent 优化 | 有限 | ✅ 三层优化 |

---

## 与 NVIDIA 栈的关系

```
                     完整技术栈
┌───────────────────────────────────────────────────┐
│  应用层：LangChain / NeMo Agent Toolkit             │
└───────────────────────────────────────────────────┘
                        │
┌───────────────────────────────────────────────────┐
│  编排层：NVIDIA Dynamo（本文档）                     │
│  - Frontend / Router / Planner / KVBM              │
└───────────────────────────────────────────────────┘
                        │
        ┌───────────────┼───────────────┐
        ▼               ▼               ▼
┌─────────────┐ ┌──────────────┐ ┌─────────────┐
│   SGLang    │ │ TensorRT-LLM │ │    vLLM     │
└─────────────┘ └──────────────┘ └─────────────┘
                        │
              ┌─────────┴─────────┐
              ▼                   ▼
        cuDNN + NCCL          Triton（可选）
              │
              ▼
            CUDA
              │
              ▼
        GPU 硬件（DGX/NVL72）

对比 NIM：
- NIM = 单模型的容器化打包（面向易用性）
- Dynamo = 跨节点集群编排（面向大规模生产）
- 两者可结合：NIM Operator 已支持部署 Dynamo CRD
```

---

## 关键要点总结

1. **不是推理引擎，是编排层**：Dynamo 协调 SGLang/TRT-LLM/vLLM，不替代它们
2. **分解式服务是核心创新**：Prefill/Decode 拆分带来数量级吞吐提升
3. **KV 感知路由降低延迟**：Flash Indexer 实现全局 KV 缓存可见性
4. **SLA 驱动而非手动调参**：声明延迟目标，Planner 自动优化拓扑
5. **生产级容错**：请求迁移确保 worker 故障不影响用户
6. **原生 Agent 优化**：优先级调度、会话隔离面向智能体工作负载
