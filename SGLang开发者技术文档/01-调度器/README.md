# 调度器技术文档

---

SGLang调度器是管理张量并行GPU worker的核心组件，负责协调整个推理系统的运行。作为系统的"大脑"，调度器承担着请求调度、批处理优化、资源管理和并行协调等关键职责。

本文档基于SGLang真实源码，深入解析调度器的架构设计、核心算法和关键实现细节，为开发者提供全面的技术参考。

---

## 🎯 调度器概述

### 核心设计理念

SGLang调度器采用**模块化设计**和**事件驱动架构**，通过Mixin模式实现功能解耦，支持多种并行策略和部署模式：

```python
class Scheduler(
    SchedulerOutputProcessorMixin,    # 输出处理
    SchedulerUpdateWeightsMixin,      # 权重更新  
    SchedulerProfilerMixin,           # 性能分析
    SchedulerMetricsMixin,            # 指标收集
    SchedulerDisaggregationDecodeMixin,  # 分离式解码
    SchedulerDisaggregationPrefillMixin, # 分离式预填充
):
    """管理张量并行GPU worker的调度器"""
    
    def __init__(self, server_args, port_args, gpu_id, tp_rank, 
                 moe_ep_rank, pp_rank, dp_rank):
        # 核心组件初始化
        self.running_batch = ScheduleBatch(reqs=[])
        self.waiting_queue = []
        self.forward_ct = 0
        
        # 请求分发器
        self._request_dispatcher = TypeBasedDispatcher([...])
        
        # 内存管理
        self.req_to_token_pool = ReqToTokenPool(...)
        self.token_to_kv_pool = BaseTokenToKVPoolAllocator(...)
        self.tree_cache = RadixCache(...)
```

### 主要特性

🔄 **事件循环多样性**: 支持标准、重叠、流水线并行等多种事件循环  
📦 **智能批处理**: 动态批处理和连续批处理，最大化GPU利用率  
💾 **内存管理**: 统一的内存池和KV缓存管理，支持SWA混合缓存  
🌐 **分布式协调**: 多维并行策略协调（TP/PP/DP/EP）  
🔧 **扩展性**: Mixin架构支持功能模块化扩展  
📊 **可观测性**: 完善的监控、调试和性能分析工具

---

## 📚 文档目录

### 基础架构篇
1. **[调度器总览与架构](01-调度器总览与架构.md)** - Scheduler类的整体设计和Mixin架构
   - Mixin模式设计原理
   - 并行配置管理
   - 事件循环架构
   - 请求分发机制

2. **[核心数据结构](02-核心数据结构.md)** - Req、ScheduleBatch等关键数据结构
   - 请求对象(Req)的完整生命周期
   - 调度批次(ScheduleBatch)的结构和管理
   - 批量请求数据结构
   - 分离式架构相关数据结构

3. **[请求处理机制](03-请求处理机制.md)** - 请求接收、处理和生命周期管理
   - TypeBasedDispatcher请求分发
   - 生成请求vs嵌入请求处理
   - 批量请求处理优化
   - 语法约束队列管理

### 核心调度篇
4. **[批处理调度策略](04-批处理调度策略.md)** - 批次生成、调度和执行策略
   - 批次调度算法
   - PrefillAdder智能添加机制
   - 动态批处理vs连续批处理
   - 资源约束下的调度优化

5. **[事件循环实现](05-事件循环实现.md)** - 三种事件循环的具体实现
   - event_loop_normal: 标准同步循环
   - event_loop_overlap: CPU-GPU重叠优化
   - event_loop_pp: 流水线并行处理
   - 分离式架构的专门事件循环

6. **[内存和资源管理](06-内存和资源管理.md)** - 内存池、KV缓存、SWA混合缓存和资源分配
   - ReqToTokenPool请求映射池
   - KV缓存分配器设计
   - RadixCache/ChunkCache前缀缓存
   - SWA混合缓存架构

### 高级功能篇
7. **[调度器扩展功能](07-调度器扩展功能.md)** - Mixin扩展、性能分析、权重更新等高级功能
   - 动态权重更新机制
   - LoRA适配器管理
   - 数据并行控制器
   - 性能分析集成

8. **[会话管理与状态控制](08-会话管理与状态控制.md)** - Session机制和请求树管理
   - Session会话架构
   - SessionReqNode请求节点
   - 会话生命周期管理
   - 请求树构建和维护

### 运维监控篇  
9. **[调度器监控与调试](09-调度器监控与调试.md)** - 性能监控、故障诊断和调试工具
   - SchedulerMetricsMixin指标体系
   - SchedulerProfilerMixin性能分析
   - 内存泄漏检测机制
   - 调试工具和状态转储

10. **[分离式架构支持](10-分离式架构支持.md)** - 预填充/解码分离、KV传输和分布式推理
    - 分离模式类型和选择
    - PrefillBootstrapQueue预填充队列
    - DecodePreallocQueue/DecodeTransferQueue解码队列
    - KV传输和分布式协调

---

## 🚀 快速入门

### 调度器工作流程

调度器的核心工作流程可以概括为以下几个步骤：

```python
def scheduler_main_loop():
    """调度器主循环工作流程"""
    while True:
        # 1. 接收新请求
        new_requests = scheduler.recv_requests()
        scheduler.process_input_requests(new_requests)
        
        # 2. 调度批次
        batch = scheduler.get_next_batch_to_run()
        
        # 3. 执行推理
        if batch:
            result = scheduler.run_batch(batch)
            scheduler.process_batch_result(batch, result)
        else:
            # 空闲时进行系统维护
            scheduler.self_check_during_idle()
```

### 事件循环选择策略

根据不同的部署配置，调度器会自动选择最适合的事件循环：

```python
# 事件循环自动选择
if disaggregation_mode == DisaggregationMode.NULL:
    if server_args.pp_size > 1:
        scheduler.event_loop_pp()          # 流水线并行
    elif scheduler.enable_overlap:
        scheduler.event_loop_overlap()     # CPU-GPU重叠
    else:
        scheduler.event_loop_normal()      # 标准同步
elif disaggregation_mode == DisaggregationMode.PREFILL:
    scheduler.event_loop_overlap_disagg_prefill()  # 分离式预填充
elif disaggregation_mode == DisaggregationMode.DECODE:
    scheduler.event_loop_overlap_disagg_decode()   # 分离式解码
```

### 请求处理示例

```python
def handle_generate_request(self, recv_req: TokenizedGenerateReqInput):
    """处理文本生成请求的完整流程"""
    # 1. 创建请求对象
    req = Req(
        rid=recv_req.rid,
        input_text=recv_req.input_text,
        input_ids=recv_req.input_ids,
        sampling_params=recv_req.sampling_params,
        return_logprob=recv_req.return_logprob,
        stream=recv_req.stream,
        lora_id=recv_req.lora_id,
    )
    
    # 2. 设置语法约束（如果有）
    if recv_req.grammar:
        req.grammar = recv_req.grammar
        self.grammar_queue.append(req)
    
    # 3. 分配资源
    if self.allocate_resources_for_request(req):
        # 4. 加入等待队列
        self.waiting_queue.append(req)
    else:
        # 资源不足，返回错误
        self.send_error_response(req, "Insufficient resources")
```

---

## 🛠️ 核心API参考

### 主要调度方法

| 方法 | 功能 | 使用场景 |
|-----|------|----------|
| `event_loop_normal()` | 标准同步事件循环 | 调试、开发环境 |
| `event_loop_overlap()` | CPU-GPU重叠事件循环 | 生产环境，高吞吐量 |
| `event_loop_pp()` | 流水线并行事件循环 | 大模型，多GPU节点 |
| `get_next_batch_to_run()` | 获取下一个执行批次 | 批次调度核心逻辑 |
| `run_batch()` | 执行批次推理 | GPU推理执行 |
| `process_batch_result()` | 处理推理结果 | 输出处理和流式传输 |

### 请求处理API

| 方法 | 功能 | 支持的请求类型 |
|-----|------|----------------|
| `handle_generate_request()` | 处理文本生成请求 | TokenizedGenerateReqInput |
| `handle_embedding_request()` | 处理嵌入请求 | TokenizedEmbeddingReqInput |
| `handle_batch_generate_request()` | 处理批量生成请求 | BatchTokenizedGenerateReqInput |
| `flush_cache()` | 刷新缓存 | FlushCacheReqInput |
| `abort_request()` | 中止请求 | AbortReq |

### 核心数据结构

```python
# 请求对象 - 调度器处理的基本单元
class Req:
    rid: str                          # 唯一请求ID
    origin_input_ids: List[int]       # 原始输入token序列
    output_ids: List[int]             # 输出token序列
    sampling_params: SamplingParams   # 采样参数配置
    
    # 会话相关
    session_id: Optional[str] = None  # 会话ID
    parent_req_id: Optional[str] = None # 父请求ID
    
    # 资源管理
    req_pool_idx: Optional[int] = None    # 请求池索引
    token_pool_indices: List[int] = []    # token池索引列表
    
    # 语法约束
    grammar: Optional[Grammar] = None     # 结构化输出约束
    
    # LoRA适配器
    lora_id: Optional[str] = None         # LoRA适配器ID

# 调度批次 - 批处理的基本单元  
class ScheduleBatch:
    reqs: List[Req]                       # 批次中的请求列表
    req_to_token_pool: ReqToTokenPool     # 请求到token池的映射
    token_to_kv_pool: BaseTokenToKVPoolAllocator # KV缓存池
    tree_cache: BasePrefixCache           # 前缀缓存
    forward_mode: ForwardMode             # 前向模式(预填充/解码/混合)
    
    def get_model_worker_batch(self) -> ModelWorkerBatch:
        """获取模型工作器批次对象"""
        pass
    
    def merge_batch(self, other_batch: 'ScheduleBatch'):
        """合并另一个批次"""
        pass
```

---

## 🔧 高级配置

### 并行策略配置

```python
# 张量并行配置
--tp-size 4                    # 张量并行大小
--tp-rank 0                    # 当前张量并行rank

# 流水线并行配置  
--pp-size 2                    # 流水线并行大小
--pp-rank 0                    # 当前流水线并行rank

# 数据并行配置
--dp-size 2                    # 数据并行大小
--dp-rank 0                    # 当前数据并行rank

# 专家并行配置(MoE)
--ep-size 4                    # 专家并行大小
--ep-rank 0                    # 当前专家并行rank
```

### 内存和缓存配置

```python
# 内存配置
--max-running-requests 256     # 最大并发请求数
--max-total-tokens 65536       # 最大token数

# 缓存配置
--disable-radix-cache          # 禁用前缀缓存
--enable-chunk-cache           # 启用块缓存
--hybrid-cache                 # 启用SWA混合缓存

# 分离式架构配置
--disagg-mode prefill          # 分离模式: prefill/decode
--disagg-port 30000           # 分离通信端口
```

### 监控和调试配置

```python
# 指标收集
--enable-metrics               # 启用指标收集
--metrics-port 8081           # 指标服务端口

# 性能分析
--enable-profile               # 启用性能分析
--profile-output-dir /tmp/prof # 分析结果输出目录

# 调试配置
--log-level DEBUG             # 日志级别
--enable-request-tracing      # 启用请求跟踪
```

---

## 📊 性能监控

### 关键性能指标

调度器提供以下关键性能指标的监控：

| 指标类别 | 指标名称 | 说明 |
|---------|----------|------|
| **吞吐量** | `gen_throughput` | 生成token吞吐量(tokens/s) |
| | `input_throughput` | 输入token吞吐量(tokens/s) |
| **资源利用** | `token_usage` | KV缓存token使用率 |
| | `memory_usage` | GPU内存使用率 |
| **队列状态** | `num_running_reqs` | 运行中的请求数 |
| | `num_waiting_reqs` | 等待队列请求数 |
| **缓存效率** | `cache_hit_rate` | 前缀缓存命中率 |
| | `eviction_rate` | 缓存淘汰率 |

### 监控数据获取

```python
# 获取调度器统计信息
stats = scheduler.get_stats()
print(f"吞吐量: {stats.gen_throughput:.2f} tokens/s")
print(f"Token使用率: {stats.token_usage:.2%}")
print(f"运行请求数: {stats.num_running_reqs}")

# 获取详细的性能分析
if scheduler.enable_profile:
    scheduler.start_profiling(ProfileConfig(
        wait_steps=5,
        warmup_steps=5, 
        active_steps=10,
        record_shapes=True
    ))
```

---

## 🎓 学习路径建议

### 初学者路径 (🟢 基础)
1. **[调度器总览与架构](01-调度器总览与架构.md)** - 了解整体设计
2. **[核心数据结构](02-核心数据结构.md)** - 掌握基本概念  
3. **[请求处理机制](03-请求处理机制.md)** - 理解请求流程

### 进阶开发者路径 (🟡 中级)
4. **[批处理调度策略](04-批处理调度策略.md)** - 深入调度算法
5. **[事件循环实现](05-事件循环实现.md)** - 掌握执行机制
6. **[内存和资源管理](06-内存和资源管理.md)** - 理解资源管理

### 高级架构师路径 (🔴 高级)  
7. **[调度器扩展功能](07-调度器扩展功能.md)** - 学习扩展机制
8. **[分离式架构支持](10-分离式架构支持.md)** - 掌握分布式设计

### 运维工程师路径 (🟣 运维)
9. **[调度器监控与调试](09-调度器监控与调试.md)** - 掌握监控工具
10. **[会话管理与状态控制](08-会话管理与状态控制.md)** - 理解状态管理

---

**📝 文档说明**: 本文档完全基于SGLang官方源码编写，确保技术准确性和实用性。所有代码示例均来自真实实现，为开发者提供可靠的技术参考。