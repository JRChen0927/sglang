# 注意力后端技术文档

SGLang支持多种注意力后端实现，通过模块化设计为不同硬件平台和性能需求提供最优的注意力计算方案。

## 技术概览

SGLang的注意力后端系统采用插件式架构，支持：
- **FlashInfer**: 高性能GPU注意力kernels，支持PagedAttention
- **FlashAttention**: 标准的Flash Attention实现
- **Torch Native**: PyTorch原生注意力实现
- **Triton**: 基于Triton的自定义注意力kernels  
- **AITER**: AMD GPU专用的注意力实现
- **MLA**: Multi-head Latent Attention支持

## 文档规划

### 核心内容（待编写）
1. **注意力后端架构设计** - 后端抽象层和选择机制
2. **FlashInfer后端详解** - PagedAttention和性能优化
3. **FlashAttention后端实现** - 标准Flash Attention集成
4. **Torch Native后端** - 兼容性后端和调试支持
5. **MLA后端支持** - Multi-head Latent Attention实现
6. **硬件适配与优化** - 不同GPU架构的适配策略
7. **性能对比与选择指南** - 后端性能特征和选择建议
8. **自定义后端开发** - 扩展新注意力后端的开发指南

### 关键特性
- **动态后端选择**: 根据模型和硬件自动选择最优后端
- **统一接口**: 通过AttentionBackend基类提供一致的API
- **性能优化**: 针对不同场景的专门优化策略
- **硬件适配**: 支持CUDA、ROCm、NPU等多种硬件平台

## 开发状态

🚧 **文档开发中** - 此目录下的详细技术文档正在编写中，将涵盖SGLang注意力后端的完整实现细节。

## 相关代码路径
- `python/sglang/srt/layers/attention/` - 注意力后端实现
- `python/sglang/srt/model_executor/model_runner.py` - 后端选择逻辑
