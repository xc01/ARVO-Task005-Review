# Task 014 — 617 Bad-Patch Records Statistics

Source: `results/task014/raw/all_positive_617.json` (HF artifact `data/reproduce/all_positive_balance_v2_617.json`, documented as "the 617 positive (bad-patch) entries" in `data/reproduce/README.md`).

## Basic
- Actual records: **617**
- Unique CVEs: **617** (1 CVE per record)
- Unique projects (where field present): **239** (21 records lack the `project` field entirely — all with sparse schema, see below)

## Failure cause taxonomy (as provided)
`cause_l1`:
| value | count |
|---|---:|
| edge_case_missed | 196 |
| all_scenarios_not_considered | 195 |
| incorrect_implementation | 117 |
| incorrect_conditioning | 83 |
| data_path_inconsistency | 26 |

`cause_l2` (top): missing_parallel_locations 109, missing_boundary_values 88, logic_inconsistency 71, missing_execution_paths 61, uncovered_input_variants 56, overlooked_exceptional_states 52, overly_strict_or_broad 43, wrong_variable_pointer_usage 25, check_wrong_position_order 23, api_misuse 21, wrong_value_threshold_constant 14, resource_lifecycle_mismatch 11, platform_architecture_considerations 10, resource_lifecycle_handling 9, inconsistent_state_across_contexts 9, protocol_semantic_variants 6, toctou_timing 5, off_by_one 3, incorrect_pointer_reference 1.

## Patch type (as provided)
- incomplete: **457**
- incorrect: **160**

## Relationship type (as provided, 172 missing/empty)
incomplete_fix_completion 174, incomplete 68, edge_case_fix 57, regression_fix 48, regressive + incomplete 43, incorrect 36, compatibility_fix 11, false_positive_reduction 8, empty/missing 172.

## Vulnerability-type grouping (as provided)
`normalized_issue` top: Use-After-Free & Pointer Issues 109, Buffer Errors 95, Improper Access Control 85, Uncontrolled Resource Consumption 55, Logic & Type Error 51, Integer Overflow & Underflow 44, Information Disclosure 41, Path Traversal 38, Injection 36, Request Forgery & Redirect 26.

CWE layer-1 top: CWE-118 (118), missing (82), CWE-666 (45), CWE-668 (40), CWE-706 (38), CWE-610 (31), CWE-74 (30), CWE-20 (30), CWE-476 (27), CWE-190 (25).

## Key-field availability
- Both C1 hash and C2 hash present: **617 / 617**
- `c1_metrics` present: 596 (missing 21 — the same sparse-schema subset)
- PoC / exploit / reproducer / execution-evidence fields: **none exist** in the dataset (verified by field scan)
- Re-exploitation evidence in this file: **none** (only the separate 12-case file cites advisories/exploit-db)
- Missing/empty: `c2_latest_date` 615, CVSS fields 589–615, `severity` 457, `consequence` 458, `c1_subject`/`c2_subject` 596, `relationship_type` 172, `cwe_layer1` 82, `ground_truth` 73, `cve_description` 65, `project` 21.

## Static candidate records (max 20)
Selection rule applied exactly as specified: only records explicitly marked with re-exploitation evidence or incomplete-fix status; using `patch_type == incomplete` (explicit dataset label, 457 available), sorted by CVE string ascending then C1 hash ascending, first 20. Output: `candidate_records.csv`.

No case-selection judgment was applied beyond this deterministic rule.
