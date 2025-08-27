# MoE与专家并行技术文档

SGLang实现了高效的MoE（Mixture of Experts）支持和专家并行计算，通过智能的专家路由和TBO（Two-Batch Overlap）技术，优化大规模稀疏模型的推理性能。

## 技术概览

MoE与专家并行系统包含：
- **专家路由**: 动态的token到专家的路由算法
- **专家并行**: 跨设备的专家分布和计算
- **TBO技术**: Two-Batch Overlap的性能优化
- **负载均衡**: 专家间的负载均衡和容量管理
- **通信优化**: 专家间的高效通信和数据传输

## 核心组件

### MoE架构
- **Expert Distribution**: 专家分布和路由管理
- **Expert Parallelism**: 专家并行计算策略
- **Load Balancing**: 专家负载均衡算法
- **Communication**: 专家间通信优化

### TBO技术
- **Two-Batch Overlap**: 双批次重叠处理
- **Pipeline Optimization**: 流水线优化策略
- **Memory Management**: TBO的内存管理

## 文档规划

### 核心内容（待编写）
1. **MoE架构设计** - 专家混合模型的架构原理
2. **专家路由算法** - Token到专家的动态路由
3. **专家并行策略** - 跨设备的专家分布计算
4. **TBO性能优化** - Two-Batch Overlap的实现细节
5. **负载均衡机制** - 专家间的智能负载分配
6. **通信优化** - 专家并行的通信优化策略
7. **容量管理** - 专家容量的动态调整
8. **性能分析** - MoE模型的性能特征和调优

### 技术特性
- **高效路由**: 智能的专家选择和路由算法
- **并行扩展**: 线性的专家并行扩展能力
- **负载均衡**: 自适应的专家负载均衡
- **性能优化**: TBO等先进的性能优化技术

## 开发状态

🚧 **文档开发中** - 此目录下的详细技术文档正在编写中，将涵盖SGLang MoE与专家并行的完整实现细节。

## 相关代码路径
- `python/sglang/srt/layers/moe/` - MoE层实现
- `python/sglang/srt/two_batch_overlap.py` - TBO实现
- `sgl-kernel/csrc/moe/` - MoE计算kernels
