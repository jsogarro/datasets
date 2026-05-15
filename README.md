# q/kdb+ Coding Assistant — Datasets

Datasets used to build, train, and evaluate a q/kdb+ specialized AI coding assistant: an 8K-example QLoRA fine-tuning corpus, a 254-document / 3,542-chunk RAG corpus with pre-built dual FAISS indices, and a 100-task q-interpreter-validated evaluation benchmark.

---

## Quick Facts

| Asset | Count | Location |
|---|---|---|
| Evaluation benchmark | 100 tasks | `data/benchmark/tasks.jsonl` |
| Fine-tuning train split | 8,000 examples | `processed/finetune/train_v2.jsonl` |
| Fine-tuning validation split | 1,000 examples | `processed/finetune/val_v2.jsonl` |
| Fine-tuning test split | 1,000 examples | `processed/finetune/test_v2.jsonl` |
| RAG corpus (chunked) | 3,542 chunks (254 docs) | `data/rag_corpus/rag_corpus_v2.jsonl` |
| RAG FAISS index (prose) | 671 chunks | `models/rag_index/prose_index/` |
| RAG FAISS index (code) | 2,871 chunks | `models/rag_index/code_index/` |
| Clean reference docs | 254 markdown files (RAG knowledge base) | `processed/docs/` |
| Source code (GitHub) | 856 `.q` + 11 `.k` source files across 12 repos | `data/raw/github/` |

---

## Directory Map

```
~/sandbox/datasets/
├── README.md                     This file
├── FILES_IN_USE.md               Tier-1/2/3 file usage guide
│
├── data/
│   ├── benchmark/                100-task evaluation benchmark
│   │   ├── tasks.jsonl           THE benchmark
│   │   └── schema.md             Per-task field documentation
│   ├── rag_corpus/               Final RAG corpus (input to the indices)
│   │   └── rag_corpus_v2.jsonl   3,542 chunks, semantically chunked
│   └── raw/
│       └── github/               12 cloned repos (856 .q + 11 .k source files):
│                                 collected, cookbook, embedPy, jupyterq, kdb,
│                                 kdb-tick, ml, nlp, qstudio, qutil, tick, TorQ
│
├── processed/
│   ├── docs/                     254 clean markdown files — the RAG knowledge base
│   │                             (input to the chunker that produced rag_corpus_v2.jsonl)
│   │   ├── 0_Overview.md … 14_*.md, A_*.md, B_*.md   ← q for Mortals chapters
│   │   ├── basics_*.md           ← 13 kx.com /q/basics/ pages
│   │   ├── learn_*.md            ←  2 kx.com /q/learn/ pages
│   │   ├── ref_*.md              ← 87 kx.com /q/ref/ pages
│   │   └── wp_*.md               ← 41 kx.com whitepapers
│   │
│   └── finetune/                 Fine-tuning data
│       ├── train_v2.jsonl        8,000 cleaned training examples
│       ├── val_v2.jsonl          1,000 validation examples
│       ├── test_v2.jsonl         1,000 held-out test examples
│       ├── CLEAN_DATA_MANIFEST.md   Provenance of the v2 build
│       │
│       └── (8 component files, combined and resplit to produce the v2 splits:)
│           ├── clean_coding_examples.jsonl
│           ├── crosslingual_sql_to_qsql.jsonl
│           ├── crosslingual_with_context.jsonl
│           ├── q_philosophy.jsonl
│           ├── docs_testable.jsonl
│           ├── docs_ipc_hdb_tick.jsonl
│           ├── curated_examples.jsonl
│           └── synthetic_mega_filtered.jsonl
│
└── models/
    └── rag_index/                Pre-built dual FAISS indices
        ├── prose_index/
        │   ├── faiss_index.bin   IndexFlatIP over 671 prose chunks (dim=384)
        │   └── metadata.jsonl    Per-chunk source/file/offset metadata
        └── code_index/
            ├── faiss_index.bin   IndexFlatIP over 2,871 code chunks (dim=384)
            └── metadata.jsonl    Per-chunk source/file/offset metadata
```

---

## Datasets in Detail

### 1. Benchmark — `data/benchmark/tasks.jsonl`

The single source of truth used to score every experimental condition. 100 hand-crafted q/kdb+ tasks distributed across four difficulty tiers:

| Tier | Count | Examples |
|---|---|---|
| Basic | 20 | Variable assignment, list ops, basic functions |
| Intermediate | 30 | qSQL aggregation, grouping, filtering |
| Advanced | 30 | Temporal joins (`aj`, `asof`), tables, attributes |
| Expert | 20 | Tickerplant architecture, IPC, optimization |

Each task carries the prompt, an `expected_output`, optional setup code, a difficulty label, and a category tag (qsql / syntax / temporal / tables / functional / optimization / ipc / tick / joins). Validation runs the model's generated code in the actual q interpreter and compares output against `expected_output` after normalization. Pass@1 reported.

Field schema is documented in `data/benchmark/schema.md`.

**Note**: This benchmark was fixed on 2026-05-10 — earlier versions contained 78 degenerate tasks (trivial solutions, malformed test cases, or syntax errors in ground truth) that were either repaired or replaced. Use only this version.

### 2. Fine-Tuning Data — `processed/finetune/`

The **v2** splits (`train_v2.jsonl`, `val_v2.jsonl`, `test_v2.jsonl`) are the cleaned, production-ready splits used by both fine-tuning runs in the project:

- **R1-Distill-Qwen-32B** — Alpaca format with `DataCollatorForCompletionOnlyLM` masking
- **Gemma 4 E4B (8B)** — Gemma native chat template (`<start_of_turn>user … <end_of_turn>`) with full-sequence loss

Both fine-tunes use the same underlying training records; only the instruction-format wrapper at training time differs.

Each example is `{"instruction": ..., "input": ..., "output": ...}`. Outputs were validated through:
1. Automated q interpreter execution where feasible
2. Documentation-sourcing for PARSE_ONLY topics (IPC, HDB, tick architecture) that cannot be unit-tested in isolation
3. Manual curation for idiomatic quality on hand-curated examples

**Component files** (combined and resplit to produce v2):

| File | Approx. count | What it contributes |
|---|---|---|
| `clean_coding_examples.jsonl` | ~7,000 | Validated code generation examples |
| `crosslingual_sql_to_qsql.jsonl` | ~1,000 | SQL → qSQL adaptation pairs |
| `crosslingual_with_context.jsonl` | ~1,800 | SQL → qSQL with schema context |
| `q_philosophy.jsonl` | ~120 | Conceptual q idiom explanations |
| `docs_testable.jsonl` | ~50 | Documentation-sourced runnable examples |
| `docs_ipc_hdb_tick.jsonl` | ~50 | PARSE_ONLY docs for IPC/HDB/tick (cannot unit-test) |
| `curated_examples.jsonl` | ~450 | Hand-curated seed examples |
| `synthetic_mega_filtered.jsonl` | ~5,000 | Synthetic examples that passed quality filters |

See `processed/finetune/CLEAN_DATA_MANIFEST.md` for full provenance.

### 3. RAG Corpus — `data/rag_corpus/`

The final flat corpus, semantically chunked. Each chunk record contains the chunk text, a `kind` field (`prose` or `code`), source file path, offset, and other metadata. The v2 chunker uses semantic boundaries (heading/code-block aware) rather than fixed token windows.

**Composition of v2** (254 documents, 3,542 chunks):
- 671 prose chunks (documentation, whitepapers, q for Mortals chapters)
- 2,871 code chunks (q source files, runnable examples extracted from docs)

### 4. RAG Indices — `models/rag_index/`

Pre-built FAISS indices ready for retrieval. Each subdirectory contains exactly two files: `faiss_index.bin` and `metadata.jsonl`.

- Embedder: `sentence-transformers/all-MiniLM-L6-v2` (384-dim)
- Index type: `IndexFlatIP` — exact inner-product search over L2-normalized embeddings (cosine similarity)
- Query routing: code / prose / hybrid, based on q-keyword and question-word heuristics

| Subdirectory | Chunks | Purpose |
|---|---|---|
| `prose_index/` | 671 | Documentation / explanatory retrieval |
| `code_index/` | 2,871 | Executable q pattern retrieval |

### 5. Clean Reference Markdown — `processed/docs/`

The 254 cleaned markdown documents that were chunked to produce `rag_corpus_v2.jsonl`. Kept as a clean reference set so users of this bundle can browse the underlying knowledge base without re-chunking.

| Source | Files | Origin |
|---|---|---|
| q for Mortals 3rd ed. | 17 chapters + appendices A, B | code.kx.com/q4m3/ |
| kx.com reference pages | 87 | code.kx.com/q/ref/ |
| kx.com basics + learn | 15 | code.kx.com/q/basics/, /q/learn/ |
| kx.com whitepapers | 41 | code.kx.com/q/wp/ |
| Other / overview | 94 | Mixed |

### 6. q Source Code — `data/raw/github/`

12 cloned repos with **856 `.q` + 11 `.k` source files**. These are the source of the 2,871 code chunks in `rag_corpus_v2.jsonl` and a primary training signal for the fine-tunes.

| Repo | Owner |
|---|---|
| `kdb` | KxSystems |
| `kdb-tick` | KxSystems |
| `cookbook` | KxSystems |
| `embedPy` | KxSystems |
| `ml` | KxSystems |
| `jupyterq` | KxSystems |
| `nlp` | KxSystems |
| `qstudio` | KxSystems |
| `tick` | KxSystems |
| `qutil` | KxSystems |
| `TorQ` | AquaQAnalytics |
| `collected` | Local snapshot |

All cleaned book/documentation markdown (q for Mortals, kx.com reference pages, whitepapers) lives in `processed/docs/`; this directory contains only raw q/k source code.

---

## Provenance and Licensing

| Source | License / Provenance |
|---|---|
| GitHub q files | Permissive licenses only (Apache 2.0, MIT, BSD). Repos screened manually. |
| q for Mortals (Smyth, 2020) | Used under fair-use for research / educational purposes. Not redistributed. |
| KX whitepapers | KX publishes whitepapers publicly at https://code.kx.com. Used under fair-use. |
| Hand-curated examples | Authored by the project author. |
| SQL→qSQL crosslingual pairs | Generated then validated (q interpreter where testable, manual review otherwise). |
| Synthetic examples | LLM-generated, then filtered through the quality pipeline; only the filtered survivors are in `synthetic_mega_filtered.jsonl`. |

Organizations using these datasets should perform their own IP audits before redistribution or commercial use.

---

## How These Datasets Are Used in the Project

| Stage | Dataset | Purpose |
|---|---|---|
| Fine-tuning | `processed/finetune/train_v2.jsonl` + `val_v2.jsonl` | QLoRA training of R1-Distill-32B and Gemma 4 E4B |
| RAG retrieval | `models/rag_index/{prose,code}_index/{faiss_index.bin, metadata.jsonl}` | Condition B (RAG+RLM) and AB (fine-tuned + RAG+RLM) |
| Evaluation | `data/benchmark/tasks.jsonl` | Every condition is scored against this (Pass@1, q-interpreter validated) |
| Rebuild RAG corpus | `processed/docs/` → `rag_corpus_v2.jsonl` → indices | Optional — only if re-chunking |
| Rebuild from source | `data/raw/` → `processed/docs/` | Optional — only if regenerating the clean markdown |

---

## Headline Results (for context)

The datasets above produced these Pass@1 scores on `data/benchmark/tasks.jsonl`:

| Condition | Score | Notes |
|---|---|---|
| Claude Code CLI | 100% | Frontier baseline |
| Codex CLI (GPT-5.5) | 99% | +26 pts over raw API (73%) — agentic harness effect |
| Claude Opus 4.7 raw API | 94% | +6 pts harness gap vs CLI |
| Gemini CLI (2.5 Pro) | 93% | |
| **R1-Distill-32B fine-tuned** | **90%** | **+40 pts** vs 50% zero-shot; beats its own 671B parent |
| Gemma 4 31B zero-shot | 89% | Beats R1-671B (83%) at 1/20 the size |
| **Gemma 4 E4B fine-tuned** | **89%** | **+5 pts**; matches 31B at 1/4 the params; runs local on 16GB MBP |
| DeepSeek-R1 + RLM only | 87% | Beats RAG+RLM — adding RAG dilutes context |
| DeepSeek-R1 + RAG + RLM | 86% | |
| Gemma 4 E4B zero-shot | 84% | |
| DeepSeek-R1 671B zero-shot | 83% | |
| DeepSeek-R1 + RAG only | 81% | |
| GPT-5.5 raw API | 73% | |
| R1-Distill-32B zero-shot | 50% | Distillation tax vs 83% parent |

---

## Reproducibility

- Embedding model: `sentence-transformers/all-MiniLM-L6-v2` (384-dim) — used for both chunking tokenization and embedding so token boundaries stay consistent.
- RNG seed: `42` across `random`, `numpy`, `torch`, `torch.cuda.manual_seed_all`, `transformers.set_seed`, and DataLoader generators (applies to the resplit step that produced the `_v2` splits).
- Splits: 80/10/10 train/val/test from a fixed-seed shuffle of the combined component files.
- Benchmark validation: strict — generated code must execute in the q interpreter without error AND its normalized output must match the task's `expected_output`. Pass@1 = correct / 100.

---

**Snapshot date**: 2026-05-14
