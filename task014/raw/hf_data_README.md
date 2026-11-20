# BadPatch-Bench dataset

Also here: `reproduce/` — scripts + data to reproduce the measurement-study tables & figures (see
`reproduce/README.md`); `zero_day_pocs/` — proof-of-concept artifacts for the discovered lingering zero-days.

Four JSONL files, one `(C1, Ci)` pair per line. Fields: `cve_id`, `project` (`owner/repo`), `c1_hash`
(initial patch under audit), `ci_hash` (later candidate commit), `reference` (`yes`/`no` — whether `Ci`'s
commit message carries an explicit reference to `C1`, i.e. the Phase-1 with-/without-reference split),
`ground_truth` (`yes` = `Ci` is a future commit ⇒ `C1` is a bad patch; `no` = not), `scenario`, `source`,
`split`.

## `badpatch_bench_full.jsonl` — the full dataset (this is what Table 1 reports)

4118 pairs. Its composition is exactly Table 1 of the paper:

| slice | projects | CVEs | positive CVEs | positive pairs | negative pairs |
|-------|---------:|-----:|--------------:|---------------:|---------------:|
| with-reference    | 221 | 905 | 147 | 283 | 2002 |
| without-reference | 143 | 242 |  80 |  80 | 1753 |
| **all**           | **265** | **1147** | **227** | **363** | **3755** |

Verify with `python3 generate_table1.py` (reproduces the counts above from `badpatch_bench_full.jsonl`).

## The splits

A project lives entirely in one split — the test projects appear in neither training split, so the held-out
evaluation is project-disjoint.

- **`badpatch_bench_phase1_train.jsonl`** — **1920 pairs**, the **with-reference** training pairs. Phase 1
  (Explicit Intent Identification) is trained on these: from `(C1 patch message, C2 candidate message)` it
  learns `true`/`false` (does `C2` genuinely state it repairs `C1`?). Direct binary SFT, no reasoning trace.
- **`badpatch_bench_phase2_train.jsonl`** — **1453 pairs**, the **without-reference** training pairs (no
  textual anchor). Phase 2 (Implicit Intent Analysis) is trained on the slice-intersecting subset of these,
  reasoning over the InterSlice.
- **`badpatch_bench_test.jsonl`** — the **565-pair** held-out evaluation set: **110 initial patches
  (45 bad, 65 good) across 62 projects**, 58 positive + 507 negative pairs (a bad patch can have more than one
  genuine follow-up, so 45 bad patches yield 58 positive pairs). Used to report end-to-end P/R/F1.

The remaining **180 pairs** of `badpatch_bench_full.jsonl` carry `split: "unused"`: they belong to the 62
test projects but are not part of the 565-pair evaluation set, so they are withheld from training to keep
the split project-disjoint. Table 1 counts the full dataset and is unaffected by how it is split.

## `llm_responses/` — the shipped end-to-end run on the test set, and how to score it

One JSON per `(C1, Ci)` test pair on which the LLM pipeline actually ran (178 of the 565; the other 387 pairs
were rejected deterministically before any LLM call — extraction found no code change for `Ci`, or the Joern
slice intersection with `C1` was empty). Each file keeps the complete trace: `phase1` (reference extraction +
classifier verdict/confidence), `rounds` (per round: the agent's decision and patch type, the Phase-3 GPT-5
judge's full reply, its AGREE/DISAGREE verdict and grounding score, the dual-consistency prongs),
`static_vote` (CodeQL/Semgrep prong), `final` (verdict + acceptance rule), `output` (the rendered report), plus
`ground_truth`, `prediction` and `status` for scoring.

```bash
python3 score_llm_responses.py        # pair-level and patch-level TP/FP/FN/TN, Accuracy/Precision/Recall/F1
python3 score_llm_responses.py -v     # + the patch-level FN/FP CVEs and the per-status breakdown
```

*Pair level* scores each `(C1, Ci)` decision (pairs without a file count as predicted `no`). *Patch level* is
what the paper reports: an initial patch `C1` is predicted bad iff any of its candidate pairs is predicted
`yes`, and is truly bad iff any pair has `ground_truth: yes`. The last two lines give, for Phase 2 (intent
scoping or the agent) and Phase 3 (verification), the number of pairs each decided (rejected + accepted), how
many positives it wrongly output as `no`, and its accuracy over all its decisions — attributed from the stored
rounds; Phase 1 only routes and never rejects a pair.
