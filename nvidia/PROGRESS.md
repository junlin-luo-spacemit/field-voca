# 学习进度追踪

## 状态说明
- ⬜ 未开始
- 🔄 进行中
- ✅ 已完成（Overview + 词汇整理）
- 📝 已精读

---

## CUDA

| 文档 | 状态 | 完成日期 | 词条数 | 备注 |
|------|------|----------|--------|------|
| Programming Guide - Part 1: Introduction & Programming Model | ✅ | 2026-06-29 | ~50 | 含 tile 编程模型（Hopper+ 新特性） |
| Programming Guide - Part 2: Writing CUDA Kernels | ✅ | 2026-06-29 | ~35 | C++基础、内存类型、coalescing、bank conflict、streams、events、CUDA graphs、统一内存 |
| Programming Guide - Part 3: Advanced CUDA | ✅ | 2026-07-01 | ~60 | Cluster、PDL、独立线程调度、异步屏障、Pipeline、TMA、Driver API、多 GPU P2P |
| Programming Guide - Part 4: CUDA Features | ✅ | 2026-07-02 | ~80 | Unified Memory、CUDA Graphs、TMA、VMM、Green Contexts 等20章 |
| CUDA C++ Best Practices Guide | ✅ | 2026-07-02 | ~45 | APOD方法论、带宽/Occupancy/指令/内存/控制流优化、兼容性 |
| CUDA Runtime API Reference | ✅ | 2026-07-02 | ~90 | 10模块：Device/Stream/Event/Memory/Unified/Graph/Texture/Surface/P2P/Occupancy |
| Compute Capabilities Appendix | ✅ | 2026-07-02 | ~55 | CC 7.5~12.x 规格、Tensor Core 矩阵、编译目标、Occupancy 计算 |

## TensorRT

| 文档 | 状态 | 完成日期 | 词条数 | 备注 |
|------|------|----------|--------|------|
| TensorRT Overview + Architecture + Capabilities | ✅ | 2026-07-03 | ~85 | 两阶段模型、精度矩阵、动态形状、内存管理、线程模型、工具链 |
| Developer Guide (Inference Library) | ✅ | 2026-07-14 | ~105 | 量化(PTQ/QAT/MXFP8)/Plugin V3/动态形状/Transformer(Attention/KV/MoE)/多设备推理/Weight Streaming/Refitting/trtexec/Polygraphy |
| API Reference | ✅ | 2026-07-21 | ~45 | Core类(Builder/Engine/ExecutionContext/Refitter等)/Network层类型速查/Plugin V2-V3/tensorrt.plugin原生插件DSL/OnnxParser |

## cuDNN

| 文档 | 状态 | 完成日期 | 词条数 | 备注 |
|------|------|----------|--------|------|
| cuDNN Overview + Core Concepts + Graph API | ✅ | 2026-07-21 | ~95 | Frontend/Backend API、张量布局、Graph工作流、启发式算法、融合引擎、Flash Attention、归一化、Kernel Cache |
| Developer Guide (其他章节) | ✅ | 2026-07-21 | ~25 | Operations参考：Block Scaling(MXFP8/NVFP4量化)、Causal Conv1d(Mamba/SSM)、MoE Grouped Matmul(None/Gather/Scatter路由)、RoPE(旋转位置编码，attn_scale折叠优化) |

## NCCL

| 文档 | 状态 | 完成日期 | 词条数 | 备注 |
|------|------|----------|--------|------|
| NCCL Overview + Usage | ✅ | 2026-07-21 | ~80 | 集合操作(AllReduce/Broadcast等)、点对点通信、通信器管理、Group Call、CUDA流语义、拓扑优化、性能调优 |

## Triton Inference Server

| 文档 | 状态 | 完成日期 | 词条数 | 备注 |
|------|------|----------|--------|------|
| Triton Overview + User Guide | ✅ | 2026-07-21 | ~90 | 架构/模型仓库/config.pbtxt/动态批处理/序列批处理/inflight batching/Ensemble/BLS/并发执行/后端/云存储 |

## AI Software Stack

| 文档 | 状态 | 完成日期 | 词条数 | 备注 |
|------|------|----------|--------|------|
| NIM Overview + Concepts | ✅ | 2026-07-21 | ~65 | 容器化微服务/NGC/model profile/TRT-LLM/vLLM后端/OpenAI API兼容/LoRA适配器/NIM Operator/RAG架构 |
| NeMo Overview + Platform | ✅ | 2026-07-21 | ~70 | Framework(预训练/SFT/LoRA/RLHF/DPO/3D并行)/Platform(Customizer/Evaluator/Guardrails/Data Designer)/评估指标/部署 |
| Dynamo Overview + Architecture | ✅ | 2026-07-21 | ~55 | 编排层/Frontend-Router-Planner-Grove-KVBM/分解式服务(P/D拆分)/KV感知路由/Flash Indexer/SLA自动扩缩容/容错/Agentic优化 |
| AI Enterprise Overview + Software + Lifecycle | ✅ | 2026-07-21 | ~50 | 双层架构(Application/Infrastructure)/发布分支(FB/PB/LTSB/Infra)/vGPU与MIG/Run:ai/K8s Operators/合规(FedRAMP/FIPS/STIG) |

## Hardware Platform

| 文档 | 状态 | 完成日期 | 词条数 | 备注 |
|------|------|----------|--------|------|
| DGX Platform Overview (DGX H100/H200 User Guide) | ✅ | 2026-07-21 | ~15 | 硬件规格/BMC/NVSM/DCGM/SuperPOD/DensiLink |
| Jetson Overview (Developer Guide r36.4) | ✅ | 2026-07-21 | ~10 | Orin系列型号矩阵/开发套件vs模组/JetPack/L4T/A-B冗余 |
| vGPU Overview (NVIDIA vGPU Software Docs) | ✅ | 2026-07-21 | ~12 | vGPU Manager/软件分支策略/vWS-vCS-vPC/time-sliced/MIG-backed |
| GPU 规格速查表补全（A100/H100/H200 官方核实） | 🔄 | 2026-08-07 | — | H100 已有；新补 A100/H200 完整算力表（官方nvidia.com核实）；**B200 逐项规格待补**（官方产品页/HGX B200页均未成功抓取结构化数据，需人工提供Datasheet PDF） |

## Omniverse

| 文档 | 状态 | 完成日期 | 词条数 | 备注 |
|------|------|----------|--------|------|
| Omniverse Overview + Kit SDK Architecture | ✅ | 2026-07-21 | ~25 | USD/Hydra/Carbonite/RTX Renderer五大组件/Kit扩展架构/一切皆扩展设计哲学/Nucleus协作流 |
| Extension Development (extension.toml/依赖解析/热重载) | ✅ | 2026-07-21 | ~11 | 扩展ID/包ID/dev paths/依赖解析器/SemVer约束/可选依赖/IExt接口 |
| OmniGraph (可视化编程节点系统) | ✅ | 2026-07-21 | ~8 | Action Graph/Push Graph/Node-Attribute-Connection核心抽象/Authoring vs Execution Graph |
| Isaac Sim (机器人仿真平台) | ✅ | 2026-07-21 | ~9 | PhysX/Newton双物理后端/四阶段工作流(Import-Configure-Simulate-Connect)/Isaac Lab/SIL测试/URDF导入 |
## Networking

| 文档 | 状态 | 完成日期 | 词条数 | 备注 |
|------|------|----------|--------|------|
| Networking Overview (NVLink/NVSwitch/InfiniBand/RoCE/GPUDirect/SHARP/Spectrum-X/BlueField/DOCA) | ✅ | 2026-07-22 | ~47 | NVLink 3代规格/NVSwitch全互连/GDR/GDS/SHARP网内计算/Spectrum-X/BlueField DPU Isolated Mode/集合通信原语/PFC+ECN扫塞控制 |
---

## 词条统计

| 领域 | 词条数 | 最后更新 |
|------|--------|----------|
| CUDA | ~245 | 2026-07-02 |
| TensorRT | ~235 | 2026-07-21 |
| cuDNN | ~120 | 2026-07-21 |
| NCCL | ~80 | 2026-07-21 |
| Triton | ~90 | 2026-07-21 |
| NIM | ~65 | 2026-07-21 |
| NeMo | ~70 | 2026-07-21 |
| Dynamo | ~55 | 2026-07-21 |
| AI Enterprise | ~50 | 2026-07-21 |
| AI Software | ~20 | 2026-06-29 |
| Hardware | ~52 | 2026-07-21 |
| Networking | ~47 | 2026-07-22 |
| Omniverse | ~53 | 2026-07-21 |
| **合计** | **~1197** | **2026-07-22** |

---

## 下一步建议

1. **全领域已收屏**：~1197 词条，涵盖 CUDA/TensorRT/cuDNN/NCCL/Triton/NIM/NeMo/Dynamo/AI Enterprise/Hardware/Networking/Omniverse 共 12 个领域
2. **AI Software 待补充**：目前只有 ~20 词条，可考虑补充 CUDA-X 软件栈全景图、cuBLAS/cuSPARSE/cuFFT 等库
3. **Master Glossary 整合**：~1197 词条已分散在各领域，可考虑更新 glossary/master_glossary.md
4. **回顾总结**：可根据工作需要对某个领域的深层文档做精读深化
