# LLM Data + Serving Systems

2028 届 · 湖北大学软件工程本科在读 · 目标：2027 暑期实习

I work on the systems side of foundation-model data and inference: peak memory, dedup
algorithms, and KV-cache lifecycle.

**Data-Juicer** (initiated by Alibaba Tongyi Lab) — **19 merged PRs** in the last 30 days,
and 8 of the last 30 commits.

**vLLM core** — **6 merged PRs** (KV cache, prefix caching, KV events); 53 external reviews,
51 I would defend as substantive.

**llm-data-curation-lab** — streaming cleaning, MinHash/SimHash near-dedup, 13-gram
decontamination, weighted mixing; 54 unit tests.

## Data-Juicer — selected merged

| Area | PRs | What changed |
|---|---|---|
| MinHash / SimHash dedup | [#1119](https://github.com/datajuicer/data-juicer/pull/1119), [#1123](https://github.com/datajuicer/data-juicer/pull/1123), [#1127](https://github.com/datajuicer/data-juicer/pull/1127), [#1128](https://github.com/datajuicer/data-juicer/pull/1128), [#1153](https://github.com/datajuicer/data-juicer/pull/1153) | Union MinHash band collisions while scanning; select kept rows from the hash column; read only line-hash columns; integer hash keys in SimHash clustering |
| Selectors | [#1145](https://github.com/datajuicer/data-juicer/pull/1145)–[#1148](https://github.com/datajuicer/data-juicer/pull/1148) | Stream top-k heaps and two-pass frequency selection instead of materializing all indices/values |
| Filters / format | [#1129](https://github.com/datajuicer/data-juicer/pull/1129), [#1164](https://github.com/datajuicer/data-juicer/pull/1164), [#1169](https://github.com/datajuicer/data-juicer/pull/1169), [#1171](https://github.com/datajuicer/data-juicer/pull/1171) | Skip the null-text path when no text column has nulls; generator expressions in alphanumeric/refinement; precompile table-text regexes |

## Data-Juicer — merged 2026-11-16

| PR | What changed | Evidence |
|---|---|---|
| [#1191](https://github.com/datajuicer/data-juicer/pull/1191) | Select kept rows after MinHash clustering | Process stage −28.8% / −18.3% / −27.8% at 20k / 100k / 300k |
| [#1190](https://github.com/datajuicer/data-juicer/pull/1190) | Validate SimHash `num_blocks` / `hamming_distance` | 4 regression tests |
| [#1192](https://github.com/datajuicer/data-juicer/pull/1192) | Drop the internal MinHash column for single-sample datasets | Boundary fix, 2 tests |

## Data-Juicer — open, under review

| PR | What it does | Evidence (local, operator-level) |
|---|---|---|
| [#1188](https://github.com/datajuicer/data-juicer/pull/1188) | Fold MinHash shingles in bounded blocks | 1M tokens peak RSS 345.44 → 267.98 MiB (−22.4%); 100k→1M growth −42.6%; digests identical; 24 tests |
| [#1189](https://github.com/datajuicer/data-juicer/pull/1189) | Bounded-block word counting | 1M tokens 244.92 → 184.98 MiB (−24.5%); counts identical |
| [#1198](https://github.com/datajuicer/data-juicer/pull/1198) | Fold Ray BTS MinHash permutations in bounded blocks | 1M tokens 3,958.7 → 75.4 MiB (−98.1%); 1M timing 3.52 → 3.18 s (1.11×); real Ray Dataset smoke 100 → 2; 5 differential tests |

> #1190–#1192 merged on 2026-11-16; #1188/#1189/#1198 are open/under review. The RSS/throughput
> numbers are local fresh-process operator benchmarks, not CI or production results.

vLLM core contributor — **KV cache, prefix caching, KV events, P/D KV connectors**.

- **6 merged** vLLM core PRs
- **53 reviews** on external vLLM PRs, **51** I would defend as substantive
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
- [#66523](https://github.com/vllm-project/vllm/pull/66523) — caught the extensible-KV warmup `commit(1, shrink=True)` edge; the author then escalated it to **block-outermost layout** and **DCP capture-table** contracts. Merged as `3319c629a1` on 2026-11-12.
- [#66502](https://github.com/vllm-project/vllm/pull/66502) — found the **zero-missing-page / full-prefix-hit** boundary in Mooncake HiSparse imports, where host registration or completion bookkeeping can run with no transfer.
- [#66738](https://github.com/vllm-project/vllm/pull/66738) — found the **sticky prefill backoff cap**: one transient idle step can halve a request's chunk for its whole lifetime.
- [#65132](https://github.com/vllm-project/vllm/pull/65132) — challenged a "no accuracy regression" claim when GPQA run 1 had **p=0.013**; asked for a third paired run or an equivalence interval.
- [#66595](https://github.com/vllm-project/vllm/pull/66595) — flagged an AIME25 exact-match drop (0.800 → 0.783) in a RecoverSSM deferred-commit PR and asked whether it was noise or a regression.
- [#62278](https://github.com/vllm-project/vllm/pull/62278) — found the `F < A` / `F == A` boundary in the shared-prefix junction set used by SWA, Mamba and offloading.
- [#66903](https://github.com/vllm-project/vllm/pull/66903) — on a scheduler idle-group optimization, asked for a spy test proving the per-step calls are actually skipped, not only that correctness is unchanged.

## Current work

- [#64875](https://github.com/vllm-project/vllm/pull/64875) — EAGLE/no-draft-group and periodic-SWA prefix-cache fixes (open, CI green)
- [#66968](https://github.com/vllm-project/vllm/pull/66968) — publish `AllBlocksCleared` immediately after a successful prefix-cache reset (open, CI green); A/B on local RTX 4060 + vLLM 0.30 backport: baseline delivered the event only after the next request (12.174 s), patched delivered it 0.002309 s after reset while idle.
- vLLM-Omni: serializer/connector work ([#9977](https://github.com/vllm-project/vllm-omni/pull/9977) merged); [#10412](https://github.com/vllm-project/vllm-omni/pull/10412) resolves the headless NixlConnector `sender_host="auto"` metadata failure (#9357); [#10470](https://github.com/vllm-project/vllm-omni/pull/10470) fixes the MRv2 tuple-output guard regression for CosyVoice3 + DSpark (#10425)
- Mooncake: [#5059](https://github.com/kvcache-ai/Mooncake/pull/5059) fixes a stranded pending admission after dropping an abandoned TCP queue head
- SGLang: [#42488](https://github.com/sgl-project/sglang/pull/42488) fixes HiCache eviction stopping after one failed unlink (approved)

## Other merged contributions

| Project | Work |
|---|---|
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
