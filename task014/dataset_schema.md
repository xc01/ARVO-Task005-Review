# Task 014 — Actual Dataset Structure and Field Semantics

## reexploitation_cases.json (12 records, list)
Official description (`data/reproduce/README.md`): "the re-exploitation cases (Fig. 12) — cause, `evidence_source`, `evidence_id`".

Fields (verified from data): `cve_id` (str), `project` (str, `owner/repo`), `cve_description` (str), `c1_hash` (short str), `c1_date` (ISO), `c2_hash` (short str), `c2_earliest_date` (ISO), `cause_l1`, `cause_l2`, `normalized_issue`, `evidence_source` (e.g. `github_advisory`, `nvd`, `exploit_db`), `evidence_id`, `evidence_date`, `evidence_description`, `evidence_content` (null in all 12), `confidence` (null in all 12), `reason` (str).

Semantics of C1/C2 (from READMEs and `cause_reason` text in the 617 dataset): C1 = initial security patch for the CVE; C2 = the earliest later commit that completed/fixed the same issue (i.e. C1 was a bad patch). `cause_l1`/`cause_l2` classify why C1 failed; `evidence_*` points to external evidence of re-exploitation. No fields for PoC bytes, reproducer inputs, repo URL beyond `project`, language, or full commit SHAs.

## all_positive_617.json (617 records, list)
Fields: `cve_id`, `project`, `c1_hash`, `c1_date`, `c2_hash`, `c2_earliest_date`, `cve_description`, `published_date`, `cause_l1`, `cause_l2`, `cause_reason` (natural-language failure description), `normalized_issue` (paper's vulnerability-type grouping), `relationship_type` (e.g. `incomplete_fix_completion`, `edge_case_fix`, `regression_fix`), `source`, `ground_truth` (`yes`/null), `patch_type` (`incomplete`/`incorrect`), `with_ref`, `c1_metrics` (object: sha, lines, hunks, files, developers, commits), `cwe_layer1`. A subset of records additionally has `c2_latest_date`, `c1_subject`, `c2_subject`, `cvss_*`, `severity`, `severity_reason`, `consequence`.

- `patch_type` distinguishes **incomplete** (457) vs **incorrect** (160) — confirmed by counts and by `cause_reason` text.
- All 617 records have both `c1_hash` and `c2_hash`.
- **No PoC / exploit / reproducer / execution-evidence fields exist** in this file (verified field scan).

## badpatch_bench_full.jsonl (4,118 pair records)
Per `data/README.md`: one `(C1, Ci)` pair per line; fields `cve_id`, `project`, `c1_hash`, `ci_hash`, `reference` (`yes`/`no`: whether Ci's message explicitly references C1), `ground_truth` (`yes` = C1 is a bad patch confirmed by later commit), `scenario`, `source`, `split`.

## badpatch_bench_final.csv (565 rows)
Fields: `cve_id`, `project`, `c1_hash`, `c2_hash`, `reference`, `ground_truth`. This is the 565-pair held-out test set per README.

## Semantic caution
- `c2_hash`/`ci_hash` presence does NOT prove P1-blocking or bypass existence; it only marks a later commit whose relationship to C1 was classified.
- The 12 re-exploitation records cite external advisories/exploit-db entries as re-exploitation evidence but ship no PoC bytes (`evidence_content` is null in all 12).
