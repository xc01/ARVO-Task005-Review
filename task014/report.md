# Task 014 — PatchAudit Data Audit Report

## What was obtained
8 data files (≈3.1 MiB total) from the official GitHub repo (SRestLabUB/PatchAudit) and the official HF mirror (zhcharyzhang/patchaudit-artifact). Zenodo returned HTTP 403 on the API; no Zenodo data obtained (URL documented in `sources.md`). No large archives, images, models, PoC packages, or git repos downloaded. All within the 50 MiB budget.

## Key facts
1. **12 re-exploitation cases** obtained in full (`reexploitation_cases.json`). Each records CVE, project, short C1/C2 hashes with dates, failure-cause taxonomy, external evidence reference (github_advisory / nvd / exploit_db), and a reason string. **None ship PoC bytes, bypass PoC, reproducer inputs, execution evidence, full 40-char SHAs, language, or explicit repo URL** (`evidence_content` is null in all 12).
2. **617 bad-patch records** obtained in full. All have C1+C2 hashes. `patch_type` distinguishes incomplete (457) vs incorrect (160). `cause_l1`/`cause_l2` + `cause_reason` give failure descriptions. **No PoC or re-exploitation-evidence fields exist** in this file.
3. **4,118-pair benchmark** (`badpatch_bench_full.jsonl`) + 565-row test CSV, with README-documented field semantics (C1=initial patch, Ci=later candidate, ground_truth=yes ⇒ C1 bad).
4. **Overlap**: HarfBuzz appears as project (CVE-2023-25193, CVE-2015-9274) but with commits different from ours; Assimp absent. No direct commit/CVE overlap with Task 011/012 targets.
5. 20 static candidates generated deterministically from `patch_type == incomplete`, sorted per spec → `candidate_records.csv`.

## Unverified / blockers
- Zenodo record 23654079: HTTP 403 on API; contents unknown. The HF mirror is treated as the official artifact mirror per task instructions.
- Whether the GitHub `all_positive.json` (older format, same count 617, different schema/SHA) is an earlier version of the HF 617 file: both have 617 records; the HF file matches the documented reproduction dataset; the relationship between them is **not verified** — recorded as two source variants.
- `evidence_content`/`confidence` in re-exploitation file: null in all 12 — the actual advisory text is **not shipped**; only IDs/dates/descriptions.
- No dynamic or PoC-level verification of any claim is included in the data; all P1/P2/execution conditions are `not_established` per the Planner's labeling standard.
