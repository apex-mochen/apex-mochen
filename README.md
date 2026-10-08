# AI Infra / LLM Serving

近期围绕 vLLM / SGLang 的 KV Cache、缓存生命周期和权重加载参与开源贡献，同时在单卡环境中实践推理服务部署与并发评测。

## 已合并贡献

- **vLLM [#59959](https://github.com/vllm-project/vllm/pull/59959)**：修复 `LLM.score()` 修改调用方 `PoolingParams.task` 的问题。通过 clone 保留参数所有权，避免参数跨模型复用时失败；回归测试纳入已有 IO processor 测试。
- **vLLM [#59399](https://github.com/vllm-project/vllm/pull/59399)**：补充多模态 `mm_processor_kwargs` 的 flat/scoped 覆盖优先级回归，检查请求覆盖后仍保留视频 `fps` 等同级配置。

## 当前推进

以下 PR 截至 **2026-10-08** 均未合并，链接保留实现、测试和 review 过程。

| 项目 | 问题与改动 | 验证 |
| --- | --- | --- |
| [vLLM #59790](https://github.com/vllm-project/vllm/pull/59790) | P2P KV store 超时后主动请求取消，等待取消确认或传输终态后再释放槽位 pins；处理共享 job 和 consumer abort | tiering 测试 439 passed、1 skipped，本人复跑；物理 hung RDMA 未验证 |
| [SGLang #42745](https://github.com/sgl-project/sglang/pull/42745) | Rust TreeCore 按 SWA 窗口尾部锁定节点，并在 D2H buffer backup pending 时保持节点完整 | 具体回归锁定范围 39 → 8 token；[外部贡献者 GPU 复核](https://github.com/sgl-project/sglang/pull/42745#issuecomment-6045736823)覆盖 13 种配置，移除 pending 保护后 6 种失败 |
| [SGLang #42077](https://github.com/sgl-project/sglang/pull/42077) | HiCache 未启用元数据缓存时，按目标文件路径查询，避免小量查询扫描整个目录 | 35 项测试；暖文件系统、32 个目标路径、5 万文件时，查询中位耗时 22.107 → 0.090 ms，仅为文件查询微基准 |
| [SGLang #42474](https://github.com/sgl-project/sglang/pull/42474) | 为 V4 与共享 DeepSeek loader 增加权重拷贝并发参数，未设置时保留默认行为 | 真实线程与 CPU 张量的 11 项回归通过；完整模型、多 GPU 加载性能未验证 |
| [LiteLLM #43293](https://github.com/BerriAI/litellm/pull/43293) | DualCache 的 Redis 与内存层独立解析默认 TTL，覆盖直接、批量写入及 auth-cache 路径 | 10 月 5 日 main 组合测试 165 项通过；真实 Redis 12/12 检查，对照 main 为 8/12 |

贡献采用 AI 辅助协作；具体实现、验证执行者和环境边界在各 PR 中说明。外部贡献者复核与上游维护者批准分别记录。

## 推理服务并发评测

在 **RTX 4060 Laptop 8GB / Qwen2.5-0.5B-Instruct** 上完成 vLLM 与 SGLang 固定负载对比。工程采用 AI 辅助实现，本人参与服务运行和 SGLang 补跑。

- 同一官方客户端，输入/输出固定为 **1024 / 256 token**，并发 **1 / 8 / 32 / 64**，每档 64 请求、3 次重复；独立预热并关闭前缀缓存。
- 完成 **24 轮、1536 个请求**的结果校验，保存构建 commit、启动配置、逐请求数据与 GPU 遥测，生成 Markdown、CSV、JSON 和六指标图表。
- vLLM 并发 1 → 64 时，吞吐 **38.3 → 1024.4 token/s**，TTFT **108.8 → 4483.3 ms**。这是有限批次实验中的吞吐/延迟变化，不代表代码优化收益、稳态容量或通用引擎排名。

[实验方法、结果与边界](https://zhuanlan.zhihu.com/p/2090575637522265099)

## 技术复盘

- vLLM 调度器 underflow 问题排查：[掘金](https://juejin.cn/post/7688885975998726180) · [知乎](https://zhuanlan.zhihu.com/p/2090576214847125367)
- MiniMax M2 流式 parser 结束标签问题：[掘金](https://juejin.cn/post/7688910933352939556) · [知乎](https://zhuanlan.zhihu.com/p/2090577301230690831)

## 日常使用

Python · Linux / WSL · Git / GitHub · pytest / unittest。项目实践涉及 vLLM、SGLang、PyTorch 与 Redis，近期在具体贡献中阅读和修改 Rust TreeCore。


