# Task 014 — Overlap with Assimp / HarfBuzz

Checked: all records in `all_positive_617.json`, `all_positive.json`, `badpatch_bench_full.jsonl`, `badpatch_bench_final.csv`, and `reexploitation_cases.json` against:
- project names containing `assimp` or `harfbuzz`
- the exact commits: `6a3ac623b960905b3450b78e7614453dae0540ed`, `67449c39331babb88f7d29d737895d786cd5da33`, `1909b3e8d27deed5acc93e7f9a0bf48b397e1788`, `ff8fa1052cac4bea0a8a97d28856b422c5bfbe0b`

## Result
- **Assimp**: no records in any file. No direct overlap.
- **HarfBuzz**: project-level match found — CVE-2023-25193 (`harfbuzz/harfbuzz`, C1 `85be877925ddbf34f74a1229f3ca1716bb6170dc`) appears in the 617 dataset, the old repo `all_positive.json`, `badpatch_bench_full.jsonl`, and `badpatch_bench_final.csv`. Also CVE-2015-9274 (`harfbuzz/harfbuzz`, C1 `c917965b9e6fe2b21ed6c51559673288fa3af4b7`) in the two JSON datasets.
- **Exact commit matches to our Task 011/012 commits**: **none**. The HarfBuzz C1 `85be8779…` is a different commit from our `67449c39…`; this is a project-level overlap only, not the same vulnerability.
- Similar vulnerability types (buffer overflow etc.) were NOT treated as same-vulnerability evidence.
