# KV缓存存储技术文档

SGLang实现了多层次的KV缓存存储系统，通过Radix Tree、分层缓存和智能驱逐策略，最大化缓存命中率和内存利用效率。

## 技术概览

KV缓存存储系统采用多层架构设计：
- **Radix Tree缓存**: 基于前缀的高效缓存索引结构
- **分层缓存**: 内存和存储的多级缓存体系
- **Chunk缓存**: 分块的细粒度缓存管理
- **LoRA缓存**: 针对LoRA适配器的专门缓存
- **SWA缓存**: Sliding Window Attention的特殊缓存

## 核心组件

### 缓存类型
- **RadixCache**: 标准Radix Tree缓存
- **ChunkCache**: 分块缓存实现
- **HiRadixCache**: 分层Radix缓存
- **LoRARadixCache**: LoRA专用缓存
- **SWARadixCache/SWAChunkCache**: SWA专用缓存

### 内存管理
- **TokenToKVPoolAllocator**: KV缓存内存分配器
- **ReqToTokenPool**: 请求到Token的映射池
- **页面管理**: 基于页面的内存管理机制

## 文档规划

### 核心内容（待编写）
1. **KV缓存架构设计** - 多层缓存体系和设计原理
2. **Radix Tree实现** - 前缀树的构建和查询算法
3. **内存分配与管理** - 页面分配和内存池管理
4. **缓存驱逐策略** - LRU、LFU等驱逐算法实现
5. **分层缓存机制** - 内存-存储的多级缓存
6. **Chunk缓存优化** - 分块缓存的性能优化
7. **LoRA缓存管理** - 多LoRA的缓存策略
8. **性能监控与调优** - 缓存命中率和性能分析

### 技术特性
- **高命中率**: 智能的前缀匹配和缓存策略
- **内存高效**: 精确的内存分配和回收机制
- **并发安全**: 多线程环境下的缓存一致性
- **可扩展性**: 支持分布式缓存和存储后端

## 开发状态

🚧 **文档开发中** - 此目录下的详细技术文档正在编写中，将涵盖SGLang KV缓存存储的完整实现细节。

## 相关代码路径
- `python/sglang/srt/mem_cache/` - KV缓存核心实现
- `python/sglang/srt/mem_cache/allocator.py` - 内存分配器
- `python/sglang/srt/mem_cache/radix_cache.py` - Radix Tree缓存
