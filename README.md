# LLM Serving / AI Infra

关注大模型推理服务。近期在 vLLM、SGLang 修复缓存生命周期、调度与接口正确性问题，在 Mooncake 维护 C++ 测试基础设施，并在 Data-Juicer 优化去重与选择器的数据读取路径。

已合并 10 个本人发起的 PR（vLLM 6 · SGLang 1 · Mooncake 1 · Data-Juicer 2），另有 1 个共同作者补丁。

## 已合并的代表贡献

| 项目 | 工作 |
|---|---|
| [Data-Juicer #1125](https://github.com/datajuicer/data-juicer/pull/1125) | 四个字段选择器改为按 10,000 行分批读取惰性 Column，避免逐行解码；1M 行 topk 8.55s→2.20s、range 32.33s→13.56s，峰值内存与改动前持平。 |
| [Data-Juicer #1123](https://github.com/datajuicer/data-juicer/pull/1123) | exact / simhash 去重改为一次读取 hash 列后 `select`，不再解码整行；1M 行 exact 5.70s→1.01s、simhash 36.06s→5.88s，峰值内存 1045MB→496MB，保留行完全一致。 |
| [Mooncake #4895](https://github.com/kvcache-ai/Mooncake/pull/4895) | 修复 session_tag_test 在 ASan 下的分配／释放不匹配：补齐 nothrow 与数组 new/delete 重载，统一 malloc/free 路径并保留分配计数；仅修改测试代码。 |
| [vLLM #61976](https://github.com/vllm-project/vllm/pull/61976) | 修复优先级调度中的自抢占提前退出：自抢占命中当前请求时循环提前 break，使其后更高优先级请求整步拿不到 token；改为继续调度，CPU 回归检查修复前后行为。 |
| [SGLang #40899](https://github.com/sgl-project/sglang/pull/40899) | 共同作者：收窄图像/音频路径读取的 OSError 捕获范围，保留 EMFILE 等服务端故障，补充回归测试；补丁由原作者 cherry-pick。 |
| [vLLM #59790](https://github.com/vllm-project/vllm/pull/59790) | P2P KV 超时或中止时，等待 NIXL 传输终态后释放 pins 和发送 AbortAck；取消请求本身不作为停止证明；消费者超时释放另由 #61532 跟踪。 |
| [SGLang #42077](https://github.com/sgl-project/sglang/pull/42077) | 将 HiCache 无元数据缓存路径的全目录扫描改为目标文件检查，保留 KV/Mamba 命中规则。 |
| [vLLM #59962](https://github.com/vllm-project/vllm/pull/59962) | 修复 RecoverSSM 在整块边界选错状态槽的问题，统一 KDA 提交规划与 V2 后处理的索引语义；边界与请求重映射用例由 4 failed 转为 19 passed。 |
| [vLLM #59263](https://github.com/vllm-project/vllm/pull/59263) | 补齐 Batch Chat 的 reasoning 配置传递，修复关闭 thinking 后结构化输出约束失效的问题。 |
| [vLLM #59959](https://github.com/vllm-project/vllm/pull/59959) | 修复 `LLM.score()` 修改调用方参数的问题，保持参数复用和显式冲突检查的行为。 |
| [vLLM #59399](https://github.com/vllm-project/vllm/pull/59399) | 补充多模态参数优先级回归，验证请求覆盖时保留 `fps` 等同级配置。 |

## 在审工作

- [SGLang #46923](https://github.com/sgl-project/sglang/pull/46923)：抢占可能选中 beam group leader，其成员行仍会写入已释放的 KV 槽并覆盖其他请求的缓存；改为把 leader 排除在抢占候选之外。
- [Data-Juicer #1127](https://github.com/datajuicer/data-juicer/pull/1127)：行级去重只从 hash 列统计，不再为计数解码整行。
- [Mooncake #5060](https://github.com/kvcache-ai/Mooncake/pull/5060)：把 offload 完成的 LOCAL_DISK 副本写入 OpLog，避免恢复时丢失副本记录。

状态以对应 PR 为准。[本人发起的 PR](https://github.com/pulls?q=is%3Apr+author%3Aapex-mochen)

## 项目实践

**[LLM Serving Benchmark Lab](https://github.com/apex-mochen/llm-serving-benchmark-lab)**

在同机、同 GPU、同模型条件下比较 vLLM / SGLang 的并发响应，保存逐请求数据、运行配置和 GPU 遥测。提供命令行工具、数据校验、FastAPI / SQLite 只读结果浏览器，支持请求时间线、原始文件与证据报告导出。采用 AI 辅助协作，本人参与服务运行与 SGLang 补跑；有限批次结果不外推稳态容量，trace 测量可信度仍在验证。

**推理服务接口正确性回归**

本地 Frontend Regression Lab，检查 reasoning、JSON Schema、流式响应与工具调用的组合行为。用事件控制 Batch 会话乱序完成，检查结果和 usage 的会话归属，并通过故障注入检验断言。尚未发布独立仓库；批量乱序验证任务由本人指定，代码实现与测试由 Codex 完成。

## 技术复盘

- vLLM 调度器 underflow：[掘金](https://juejin.cn/post/7688885975998726180) · [知乎](https://zhuanlan.zhihu.com/p/2090576214847125367)
- MiniMax M2 流式输出的结束标签：[掘金](https://juejin.cn/post/7688910933352939556) · [知乎](https://zhuanlan.zhihu.com/p/2090577301230690831)

代码与实验有 AI 辅助，具体改动、验证范围和剩余限制见各 PR。
