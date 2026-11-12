# LLM Serving / AI Infra

vLLM core contributor — **KV cache, prefix caching, KV events, P/D KV connectors**.

- **6 merged** vLLM core PRs
- **52 reviews** on external vLLM PRs, **50** I would defend as substantive
- Focus: cache lifecycle, prefix-cache correctness, connector pinning, and event ordering

## vLLM core — merged

| PR | What changed |
|---|---|
| [#61976](https://github.com/vllm-project/vllm/pull/61976) | Priority scheduling: keep scheduling running requests after a priority self-preemption instead of exiting early |
| [#59962](https://github.com/vllm-project/vllm/pull/59962) | RecoverSSM: choose the correct state slot at exact block boundaries; unify KDA commit planning and V2 post-processing indices |
| [#59790](https://github.com/vllm-project/vllm/pull/59790) | KV offload P2P: hold store pins and AbortAck until NIXL transfers reach a terminal state |
| [#59959](https://github.com/vllm-project/vllm/pull/59959) | `LLM.score()`: stop mutating caller-owned sampling params |
| [#59263](https://github.com/vllm-project/vllm/pull/59263) | Batch Chat: forward the reasoning wiring so constrained output still works when thinking is off |
| [#59399](https://github.com/vllm-project/vllm/pull/59399) | Multimodal: regression coverage for scoped processor kwargs precedence |

## Selected reviews

These are reviews where the finding changed a decision, not just a comment.

- [#63936](https://github.com/vllm-project/vllm/pull/63936) — blocked merge on an unexplained **3/128 output mismatch** in NIXL/Mooncake early KV receive; asked for a deterministic root cause or a narrower gate before treating it as safe.
- [#66523](https://github.com/vllm-project/vllm/pull/66523) — caught the extensible-KV warmup `commit(1, shrink=True)` edge; the author then escalated it to **block-outermost layout** and **DCP capture-table** contracts.
- [#66502](https://github.com/vllm-project/vllm/pull/66502) — found the **zero-missing-page / full-prefix-hit** boundary in Mooncake HiSparse imports, where host registration or completion bookkeeping can run with no transfer.
- [#66738](https://github.com/vllm-project/vllm/pull/66738) — found the **sticky prefill backoff cap**: one transient idle step can halve a request's chunk for its whole lifetime.
- [#65132](https://github.com/vllm-project/vllm/pull/65132) — challenged a "no accuracy regression" claim when GPQA run 1 had **p=0.013**; asked for a third paired run or an equivalence interval.
- [#66595](https://github.com/vllm-project/vllm/pull/66595) — flagged an AIME25 exact-match drop (0.800 → 0.783) in a RecoverSSM deferred-commit PR and asked whether it was noise or a regression.
- [#62278](https://github.com/vllm-project/vllm/pull/62278) — found the `F < A` / `F == A` boundary in the shared-prefix junction set used by SWA, Mamba and offloading.
- [#66954](https://github.com/vllm-project/vllm/pull/66954) — challenged an SM100 warmup KV-shrink formula that had only GB200 evidence; asked for a CPU test of the target block count and its no-shrink/gate branches.

## Current work

- [#64875](https://github.com/vllm-project/vllm/pull/64875) — EAGLE/no-draft-group and periodic-SWA prefix-cache fixes (open, CI green)
- [#66968](https://github.com/vllm-project/vllm/pull/66968) — publish `AllBlocksCleared` immediately after a successful prefix-cache reset (open, CI green)
- KV-event idle-reset fix (local, in preparation): publish `AllBlocksCleared` immediately after a successful prefix-cache reset instead of waiting for the next scheduler step
- vLLM-Omni: serializer/connector work ([#9977](https://github.com/vllm-project/vllm-omni/pull/9977) merged)

## Other merged contributions

| Project | Work |
|---|---|
| [Data-Juicer #1119](https://github.com/datajuicer/data-juicer/pull/1119) / [#1123](https://github.com/datajuicer/data-juicer/pull/1123) / [#1125](https://github.com/datajuicer/data-juicer/pull/1125) / [#1127](https://github.com/datajuicer/data-juicer/pull/1127) / [#1128](https://github.com/datajuicer/data-juicer/pull/1128) / [#1129](https://github.com/datajuicer/data-juicer/pull/1129) | Dedup operators: read only hash columns, batch lazy columns, fix MinHash band retention and long-chain path compression; add boundary regressions |
| [Mooncake #4895](https://github.com/kvcache-ai/Mooncake/pull/4895) | ASan allocation/deallocation mismatch in `session_tag_test`: align nothrow and array new/delete with malloc/free |
| [SGLang #42077](https://github.com/sgl-project/sglang/pull/42077) | HiCache: replace a full-directory scan with a target-file check on the no-metadata path |
| [SGLang #40899](https://github.com/sgl-project/sglang/pull/40899) | Co-author: narrow image/audio OSError handling so EMFILE and server faults stay visible |

## Project

**[LLM Serving Benchmark Lab](https://github.com/apex-mochen/llm-serving-benchmark-lab)**

Same host, same GPU, same model comparisons of vLLM / SGLang concurrency: per-request data, run config, GPU telemetry, FastAPI/SQLite result browser, and raw evidence export. AI-assisted; finite batches are not extrapolated to steady-state capacity, and trace measurement is still being validated.

## Technical notes

- vLLM scheduler underflow: [Juejin](https://juejin.cn/post/7688885975998726180) · [Zhihu](https://zhuanlan.zhihu.com/p/2090576214847125367)
- MiniMax M2 streaming end tag: [Juejin](https://juejin.cn/post/7688910933352939556) · [Zhihu](https://zhuanlan.zhihu.com/p/2090577301230690831)

Code and experiments include AI assistance; each PR states the exact change, validation scope, and remaining limits.
