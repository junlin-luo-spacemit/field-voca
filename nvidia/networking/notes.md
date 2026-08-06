# 网络互连学习笔记

> 文档入口：
> - Networking: https://docs.nvidia.com/networking/
> - NCCL: https://docs.nvidia.com/deeplearning/nccl/index.html
> - DOCA: https://docs.nvidia.com/doca/sdk/index.html
> - GPUDirect: https://docs.nvidia.com/gpudirect-storage/

---

## AI 数据中心网络分层

现代 AI 超算集群的互连分为三层，速度依次递减：

```
Tier 1 — 节点内 GPU 互连（NVLink / NVSwitch）
  8× H100 ←NVLink 4.0→ NVSwitch（非阻塞全互连，900 GB/s per GPU）

Tier 2 — 机柜内互连（NVLink Network，Blackwell 起）
  NVL72：72× B200 通过 NVLink Switch 组成单一 NVLink 域

Tier 3 — 节点间 / 机柜间互连（InfiniBand / RoCE）
  Quantum-2 InfiniBand Switch（400 Gb/s NDR）或 Spectrum-X（RoCE）
       ↕
  其他计算节点 / 存储节点（NVMe-oF）
```

**关键认知**：训练大模型时，通信瓶颈往往不在算力而在互连带宽。NVLink 带宽（900 GB/s~）比 InfiniBand（400 Gb/s = 50 GB/s）高一个数量级以上，这就是为什么大模型训练要尽量让通信密集的并行策略（如 Tensor Parallel）限制在 NVLink 域内，而通信量小的策略（如 Data Parallel）才跨节点走 InfiniBand。

---

## 分布式训练通信模式与并行策略的对应关系

| 并行策略 | 主要通信操作 | 通信量特征 | 推荐互连层 |
|----------|-------------|-----------|-----------|
| Tensor Parallel (TP) | All-Reduce（层内） | 高频、大数据量 | NVLink（节点内） |
| Pipeline Parallel (PP) | Point-to-Point（层间激活值） | 中等，流水线气泡敏感 | NVLink / IB 均可 |
| Data Parallel (DP) | All-Reduce（梯度同步） | 训练末尾集中爆发 | InfiniBand（跨节点） |
| ZeRO / FSDP | Reduce-Scatter + All-Gather | 参数分片，通信换内存 | NVLink 优先 |
| Expert Parallel (MoE) | All-to-All（专家路由） | 不规则、突发性强 | NVLink 优先，对延迟敏感 |

这张表是理解"为什么 3D 并行要这样切分"的关键：TP 通信最密集所以放在 NVLink 域内（一个节点 8 卡），DP 通信频率最低所以放在最外层跨节点。

---

## NCCL 重要概念

### Communicator / 通信子
NCCL 中标识一组参与集合通信的 GPU 的对象（类似 MPI Communicator）。每个 GPU 在通信子中有唯一 rank。

### Ring vs Tree 算法选择
- **Ring**：GPU 连成逻辑环，带宽利用率最优（趋近线路峰值），适合大消息 All-Reduce
- **Tree**：二叉树拓扑，延迟更低（O(log N) vs O(N)），适合小消息或大规模节点数场景
- NCCL 会根据消息大小和拓扑自动切换算法（`NCCL_ALGO` 可手动指定）

### GDR (GPUDirect RDMA) / GPU 直连 RDMA
允许 InfiniBand 网卡直接读写 GPU 显存，绕过 CPU 和系统内存，降低延迟约 30%。这是 NCCL 跨节点通信性能的基础，需要 NIC 与 GPU 在同一 PCIe 交换域下才能发挥最佳效果（拓扑亲和性）。

### SHARP 与 NCCL 的配合
NCCL 可以调用 SHARP 将 All-Reduce 的规约计算下沉到 Quantum 交换机执行，训练脚本无需修改，只需网络硬件和 NCCL 版本支持（`NCCL_COLLNET_ENABLE=1`）。这是"应用层无感知，基础设施层加速"的典型案例。

---

## GPUDirect 技术族的实际应用场景

- **GPUDirect RDMA**：多节点分布式训练的梯度通信路径（NCCL 底层依赖）
- **GPUDirect Storage**：从存储直接加载训练数据到 GPU 显存，避免"存储→CPU内存→GPU显存"两次拷贝，对大规模数据加载（如 LLM 预训练海量语料）收益明显
- **GPUDirect P2P**：单机多卡场景，同 PCIe Switch 下的 GPU 直接交换数据，是节点内 NVLink 不可用时的退化路径

---

## BlueField DPU 系列与零信任架构

| 型号 | 网络接口 | ARM 核心数 | 定位 |
|------|----------|------------|------|
| BlueField-2 | 100GbE / HDR IB | 8× Cortex-A72 | 网络/存储卸载 |
| BlueField-3 | 400GbE / NDR IB | 16× Cortex-A78AE | 零信任安全、AI 基础设施 |
| BlueField-3 SuperNIC | 400GbE | 无 ARM（精简版） | Spectrum-X 专用网络加速卡 |

**Isolated Mode（零信任）的意义**：在多租户云环境中，主机（租户可控）无法访问 DPU 上运行的基础设施软件（网络策略、防火墙、监控），即使租户获得主机 root 权限也无法绕过网络安全策略。这是云服务商采用 DPU 的核心安全动机。

---

## Spectrum-X 与 InfiniBand 的选型对比

| 维度 | InfiniBand (Quantum-2) | Spectrum-X (以太网) |
|------|------------------------|----------------------|
| 协议 | 原生 InfiniBand | RoCE v2（以太网） |
| 生态兼容性 | 需专用 IB 网卡/交换机 | 复用现有以太网运维经验 |
| AI 性能 | 略优（原生拥塞控制+SHARP） | 接近 IB（自适应路由+专用调度） |
| 典型场景 | 超算中心、专用 AI 集群 | 云服务商、已有以太网基础设施 |

---

## 待深入阅读

- [ ] NCCL Developer Guide（拓扑感知调优、NCCL_ALGO/NCCL_PROTO 环境变量）
- [ ] GPUDirect Storage 性能白皮书
- [ ] BlueField-3 DPU Architecture（DOCA Flow 编程模型）
- [ ] DOCA SDK Programming Guide
- [ ] SHARP In-Network Computing 深度原理
- [ ] Spectrum-X 与 InfiniBand 实测性能对比数据
