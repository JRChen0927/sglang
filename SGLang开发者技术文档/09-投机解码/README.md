# 投机解码技术文档

SGLang实现了先进的投机解码（Speculative Decoding）技术，通过EAGLE和EAGLE3算法，使用小型Draft模型加速大型Target模型的推理，显著提升生成速度。

## 技术概览

投机解码系统采用双模型架构：
- **Draft模型**: 小型高速模型生成候选token序列
- **Target模型**: 大型精确模型验证和修正候选序列
- **验证机制**: 并行验证和接受长度优化
- **Tree Attention**: 树状注意力机制优化候选生成
- **性能监控**: 接受率和加速比的实时监控

## 核心组件

### 投机算法
```python
class SpeculativeAlgorithm:
    EAGLE = auto()    # EAGLE算法
    EAGLE3 = auto()   # EAGLE3改进算法
```

### 关键组件
- **EAGLEWorker**: EAGLE算法的工作节点
- **SpeculativeVerifier**: 投机结果验证器
- **DraftInput**: Draft模型输入管理
- **VerifyInput**: 验证阶段输入管理

## 文档规划

### 核心内容（待编写）
1. **投机解码原理** - 投机解码的理论基础和算法原理
2. **EAGLE算法详解** - EAGLE和EAGLE3的实现细节
3. **Draft模型管理** - 小型模型的选择和优化
4. **验证机制实现** - 并行验证和接受策略
5. **Tree Attention优化** - 树状注意力的性能优化
6. **性能调优** - 接受率和速度的平衡优化
7. **监控与调试** - 投机解码的性能监控
8. **部署配置** - 投机解码的部署和配置指南

### 技术特性
- **显著加速**: 2-4倍的推理速度提升
- **质量保证**: 与原模型完全一致的输出质量
- **自适应**: 动态调整投机步数和策略
- **监控完善**: 详细的性能指标和调试信息

## 开发状态

🚧 **文档开发中** - 此目录下的详细技术文档正在编写中，将涵盖SGLang投机解码的完整实现细节。

## 相关代码路径
- `python/sglang/srt/speculative/` - 投机解码核心实现
- `python/sglang/srt/speculative/eagle_worker.py` - EAGLE算法实现
- `python/sglang/srt/speculative/eagle_utils.py` - EAGLE工具函数
