# API入口点技术文档

SGLang提供了多样化的API入口点，支持OpenAI兼容接口、原生SGLang接口和多种协议，为不同应用场景提供灵活的集成方案。

## 技术概览

API入口点系统采用分层设计：
- **HTTP服务器**: 基于FastAPI的高性能HTTP接口
- **TokenizerManager**: 请求预处理和token化
- **DetokenizerManager**: 响应后处理和去token化
- **协议适配**: OpenAI、SGLang等多种API协议支持
- **流式响应**: 支持Server-Sent Events的实时流式输出

## 核心组件

### 服务架构
```
HTTP Server (FastAPI)
    ↓
TokenizerManager (主进程)
    ↓
Scheduler (子进程)
    ↓
DetokenizerManager (子进程)
    ↓
Response Stream
```

### API接口类型
- **Chat Completions**: OpenAI兼容的对话接口
- **Completions**: 文本补全接口
- **Embeddings**: 向量嵌入接口
- **Generate**: SGLang原生生成接口
- **Batch**: 批量处理接口

## 文档规划

### 核心内容（待编写）
1. **API架构设计** - 服务层次和组件交互
2. **HTTP服务器实现** - FastAPI集成和中间件
3. **TokenizerManager详解** - 请求预处理和分发
4. **DetokenizerManager实现** - 响应处理和流式输出
5. **协议适配层** - OpenAI兼容性和扩展协议
6. **流式响应机制** - SSE和WebSocket支持
7. **认证与安全** - API密钥和访问控制
8. **监控与日志** - API调用监控和审计日志

### 技术特性
- **高并发**: 异步处理和连接池管理
- **协议兼容**: 无缝替换OpenAI API
- **实时流式**: 低延迟的流式响应
- **可扩展**: 插件化的中间件和协议扩展

## 开发状态

🚧 **文档开发中** - 此目录下的详细技术文档正在编写中，将涵盖SGLang API入口点的完整实现细节。

## 相关代码路径
- `python/sglang/srt/entrypoints/` - API入口点实现
- `python/sglang/srt/entrypoints/http_server.py` - HTTP服务器
- `python/sglang/srt/managers/tokenizer_manager.py` - Token化管理器
