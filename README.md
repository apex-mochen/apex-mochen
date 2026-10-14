# LLM Serving / AI Infra

关注大模型推理服务，近期主要参与 vLLM、SGLang 的缓存生命周期、状态索引与接口正确性修复。

## 已合并的代表贡献

| 项目 | 工作 |
|---|---|
| [vLLM #59962](https://github.com/vllm-project/vllm/pull/59962) | 修复 RecoverSSM 在整块边界选错状态槽的问题，统一 KDA 提交规划与 V2 后处理的索引语义。 |
| [vLLM #59263](https://github.com/vllm-project/vllm/pull/59263) | 补齐 Batch Chat 的 reasoning 配置传递，修复关闭 thinking 后结构化输出约束失效的问题。 |
| [vLLM #59959](https://github.com/vllm-project/vllm/pull/59959) | 修复 `LLM.score()` 修改调用方参数的问题，保持参数复用和显式冲突检查的行为。 |
| [vLLM #59399](https://github.com/vllm-project/vllm/pull/59399) | 补充多模态参数优先级回归，验证请求覆盖时保留 `fps` 等同级配置。 |
| [SGLang #42077](https://github.com/sgl-project/sglang/pull/42077) | 将 HiCache 无元数据缓存路径的全目录扫描改为目标文件检查，保留 KV/Mamba 命中规则。 |

## 在审工作

- [vLLM #59790](https://github.com/vllm-project/vllm/pull/59790)：P2P KV 传输超时后主动取消，确认传输停止后再释放槽位，处理共享 job、consumer abort 与迟到结果。
- [SGLang #42745](https://github.com/sgl-project/sglang/pull/42745)：按页拆出 SWA 窗口尾部，减少 Rust TreeCore 的过量锁定，保护 D2H 备份尚未完成的节点。

状态以对应 PR 为准。[全部贡献](https://github.com/pulls?q=is%3Apr+author%3Aapex-mochen)

## 项目实践

**推理服务并发评测**

在同机、同 GPU、同模型条件下比较 vLLM / SGLang 的并发响应，保存逐请求数据、运行配置和 GPU 遥测，分析吞吐、首 token 延迟及尾延迟。采用 AI 辅助协作，参与服务运行与 SGLang 补跑；结果属于有限批次对比。

**推理服务接口正确性回归**

本地 Frontend Regression Lab，检查 reasoning、JSON Schema、流式响应与工具调用的组合行为。用事件控制 Batch 会话乱序完成，检查结果和 usage 的会话归属，并通过故障注入检验断言。尚未发布独立仓库；批量乱序验证任务由本人指定，代码实现与测试由 Codex 完成。

## 技术复盘

- vLLM 调度器 underflow：[掘金](https://juejin.cn/post/7688885975998726180) · [知乎](https://zhuanlan.zhihu.com/p/2090576214847125367)
- MiniMax M2 流式输出的结束标签：[掘金](https://juejin.cn/post/7688910933352939556) · [知乎](https://zhuanlan.zhihu.com/p/2090577301230690831)

代码与实验有 AI 辅助，具体改动、验证范围和剩余限制见各 PR。
