# 分离式架构技术文档

SGLang分离式架构（PD-Disaggregation）将预填充（Prefill）和解码（Decode）阶段分离到不同的节点执行，实现超大模型的高效分布式推理。

## 技术概览

分离式架构通过将计算密集的预填充阶段和内存密集的解码阶段分离，优化资源利用：
- **预填充节点**: 专门处理输入token的并行计算，优化吞吐量
- **解码节点**: 专门处理自回归生成，优化延迟和内存效率
- **KV传输**: 高效的KV缓存跨节点传输机制
- **状态同步**: 分布式状态管理和一致性保证

## 核心组件

### DisaggregationMode
```python
class DisaggregationMode:
    NULL = "null"           # 标准模式
    PREFILL = "prefill"     # 预填充模式  
    DECODE = "decode"       # 解码模式
```

### 关键Mixin类
- **SchedulerDisaggregationPrefillMixin**: 预填充节点调度逻辑
- **SchedulerDisaggregationDecodeMixin**: 解码节点调度逻辑

## 文档规划

### 核心内容（待编写）
1. **分离式架构设计原理** - 架构理念和技术优势
2. **预填充节点实现** - 预填充调度器和批处理策略
3. **解码节点实现** - 解码调度器和状态管理
4. **KV缓存传输机制** - 跨节点KV传输的实现细节
5. **Bootstrap队列管理** - 请求状态同步和转移
6. **分离式事件循环** - 特殊的事件循环实现
7. **性能优化策略** - 传输优化和延迟控制
8. **部署配置指南** - 分离式部署的最佳实践

### 技术特性
- **异构部署**: 不同节点可使用不同硬件配置
- **弹性扩展**: 预填充和解码节点可独立扩展
- **负载均衡**: 智能的跨节点负载分配
- **故障恢复**: 节点故障的检测和恢复机制

## 开发状态

🚧 **文档开发中** - 此目录下的详细技术文档正在编写中，将涵盖SGLang分离式架构的完整实现细节。

## 相关代码路径
- `python/sglang/srt/disaggregation/` - 分离式架构核心实现
- `python/sglang/srt/managers/scheduler.py` - Disaggregation Mixin集成
