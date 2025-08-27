# 调度器技术文档

SGLang调度器是管理张量并行GPU worker的核心组件。本文档基于SGLang真实源码，详细解析调度器的关键实现。

## 文档目录

1. [调度器总览与架构](01-调度器总览与架构.md) - Scheduler类的整体设计和Mixin架构
2. [核心数据结构](02-核心数据结构.md) - Req、ScheduleBatch等关键数据结构  
3. [请求处理机制](03-请求处理机制.md) - 请求接收、处理和生命周期管理
4. [批处理调度策略](04-批处理调度策略.md) - 批次生成、调度和执行策略
5. [事件循环实现](05-事件循环实现.md) - 三种事件循环的具体实现
6. [内存和资源管理](06-内存和资源管理.md) - 内存池、KV缓存和资源分配
7. [调度器扩展功能](07-调度器扩展功能.md) - Mixin扩展、性能分析、权重更新等高级功能
8. [会话管理与状态控制](08-会话管理与状态控制.md) - Session机制和请求树管理
9. [调度器监控与调试](09-调度器监控与调试.md) - 性能监控、故障诊断和调试工具

## 核心特性

**管理职责**: 调度器负责管理张量并行GPU worker，协调请求处理、批次调度和资源分配。

**Mixin架构**: 采用多重继承设计，将输出处理、权重更新、性能分析、指标收集等功能模块化。

**事件循环**: 支持标准、重叠、流水线并行等多种事件循环，适应不同的性能需求。

**批处理优化**: 实现动态批处理和连续批处理，最大化GPU利用率。

## 快速参考

### 主要方法
```python
# 事件循环
scheduler.event_loop_normal()      # 标准同步循环
scheduler.event_loop_overlap()     # CPU-GPU重叠循环
scheduler.event_loop_pp()          # 流水线并行循环

# 请求处理
scheduler.recv_requests()          # 接收新请求
scheduler.handle_generate_request() # 处理生成请求
scheduler.handle_embedding_request() # 处理嵌入请求

# 批次管理
scheduler.get_next_batch_to_run()  # 获取下一批次
scheduler.run_batch()              # 执行批次推理
scheduler.process_batch_result()   # 处理批次结果
```

### 核心数据结构
```python
# 请求对象
class Req:
    rid: str                       # 请求ID
    origin_input_ids: List[int]    # 输入token序列
    output_ids: List[int]          # 输出token序列
    sampling_params: SamplingParams # 采样参数

# 调度批次
class ScheduleBatch:
    reqs: List[Req]                # 批次中的请求
    req_to_token_pool: ReqToTokenPool # token池映射
    forward_mode: ForwardMode      # 前向模式(预填充/解码)
```

基于SGLang源码编写，确保技术准确性。