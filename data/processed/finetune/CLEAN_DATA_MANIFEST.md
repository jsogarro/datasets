# Clean Training Data Manifest

Approved training files:

- `train.jsonl`
- `val.jsonl`
- `test.jsonl`
- `train_v2.jsonl`
- `val_v2.jsonl`
- `test_v2.jsonl`
- `clean_coding_examples.jsonl`
- `synthetic_mega_filtered.jsonl`
- `crosslingual_with_context.jsonl`
- `q_philosophy.jsonl`

Current canonical split counts:

- `train.jsonl`: 8,000 examples
- `val.jsonl`: 1,000 examples
- `test.jsonl`: 1,000 examples
- Total train/val/test examples: 10,000
- Unique target outputs in canonical corpus: 2,330

Quality gate applied:

- Valid JSONL.
- Schema is exactly `instruction`, `input`, `output`.
- No empty instructions or outputs.
- Outputs are q code only, with no prose answers.
- Outputs do not start with `query_`.
- Outputs do not contain inline `//` comments.
- Outputs pass deterministic bracket and quote balance checks.
- Keyword alignment checks pass for filtering, aggregation, joins, sorting, averages, sums, and time buckets.
- Train, validation, and test splits are grouped by `output`, so the same target code does not appear across split boundaries.
- KDB+/q idiom audit fixes applied for window-join time literals, keyed table unkeying via `0!`, duplicate-key quote joins, each-left/each-right wording, unsafe `.z.pg`/`.z.ps` examples, and inefficient `count select from ...` patterns.
- Expanded from Q for Mortals and KX reference/whitepaper patterns across basics/functions, qSQL/joins/temporal operations, and tick/IPC/HDB/performance examples.

Root-level compatibility aliases such as `synthetic_mega.jsonl`,
`synthetic_expanded.jsonl`, `synthetic_full.jsonl`, `augmented_examples.jsonl`,
`curated_examples.jsonl`, and `crosslingual_sql_to_qsql.jsonl` are clean
subsets/copies retained so older scripts and Colab upload workflows still find
the expected filenames.

Legacy raw JSONL files that did not pass the gate were moved to
`legacy_raw/*.raw`. Do not train on `legacy_raw`.
