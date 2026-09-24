<!-- 把下面内容存为你 GitHub profile 仓库（用户名同名的 public 仓库）的 README.md -->

# 关于我

AI Infra / LLM Inference 方向工程师。关注大模型推理引擎：调度、KV Cache、流式协议、多模态处理。

## 开源贡献

我持续向主流 LLM 推理与服务框架提交真实的 bug 修复与核心逻辑改进，修复均带回归测试。

- **vLLM**（vllm-project/vllm）
  - 修复 v1 异步调度器 `num_output_placeholders` 在 chunked prefill 并发下 underflow 导致 EngineCore 崩溃的问题（防御性降级 + 真机验证）
  - 修复 streaming/parser 层多模态与 thinking-budget 边界处理
  - 修复 KV cache / prefix cache 提升链路
- **SGLang**（sgl-project/sglang）：多模态媒体错误分类修正（5xx → 4xx）
- **FlashInfer**（flashinfer-ai/flashinfer）：JIT 编译 flag 的 CUDA 版本门控
- **LiteLLM**（BerriAI/litellm）：跨 provider 采样参数传递一致性

> 详细的一个 bug 分析过程见：[给 vLLM 修了个崩引擎的竞态 bug](https://juejin.cn/post/7688885975998726180)

## 技术栈

- Python, PyTorch, CUDA/Triton（阅读与 kernel 级定位）
- LLM Serving：vLLM, SGLang, TensorRT-LLM, FlashInfer
- Agent / 应用层：LangGraph, Pydantic-AI, LiteLLM, OpenAI-compatible API
- 工程：CPU/GPU 测试定位、CI/DCO、Git Data API、并发与调度系统

## 我在解决什么问题

一句话：让大模型在生产环境里**不崩、不悄悄丢数据、跨 provider 行为一致**。

📫 可通过 GitHub issue / 邮件联系我
