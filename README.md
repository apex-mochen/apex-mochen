# LLM Serving

最近在看 vLLM 和 SGLang，主要做 KV Cache、权重加载和推理服务相关的修复，也做了一些单卡并发实验。

## 开源贡献

已合并：

- [vLLM #59959](https://github.com/vllm-project/vllm/pull/59959)：修复 `LLM.score()` 改写调用方参数，导致参数跨模型复用失败的问题。
- [vLLM #59399](https://github.com/vllm-project/vllm/pull/59399)：补充多模态参数覆盖的回归测试，确保请求覆盖后不会丢掉视频 `fps` 等配置。

还在 review 的几个：

- [vLLM #59790](https://github.com/vllm-project/vllm/pull/59790)：P2P KV 传输超时后，等传输真正结束再回收槽位。
- [SGLang #42745](https://github.com/sgl-project/sglang/pull/42745)：让 Rust TreeCore 只锁定 SWA 窗口内的部分，同时保护还在备份的节点。
- [SGLang #42077](https://github.com/sgl-project/sglang/pull/42077)：HiCache 查少量文件时，不再扫描整个缓存目录。
- [SGLang #42474](https://github.com/sgl-project/sglang/pull/42474)：给 DeepSeek 权重加载增加拷贝线程数配置。
- [LiteLLM #43293](https://github.com/BerriAI/litellm/pull/43293)：修复 Redis 缓存误用内存层默认 TTL 的问题。

## 实验

用 RTX 4060 Laptop 8GB 和 Qwen2.5-0.5B-Instruct 跑了 vLLM / SGLang 并发对比：固定 1024/256 token，并发 1、8、32、64，每组重复三次，共 24 轮、1536 个请求。

记录了吞吐、首 token 延迟和 GPU 状态，也保留了配置和原始数据。结果是这组机器与负载下的有限批次对比。

[实验记录](https://zhuanlan.zhihu.com/p/2090575637522265099)

## 写过的复盘

- vLLM 调度器 underflow：[掘金](https://juejin.cn/post/7688885975998726180) · [知乎](https://zhuanlan.zhihu.com/p/2090576214847125367)
- MiniMax M2 流式输出的结束标签：[掘金](https://juejin.cn/post/7688910933352939556) · [知乎](https://zhuanlan.zhihu.com/p/2090577301230690831)

代码和实验有 AI 辅助，具体测试与验证记录见 PR。

