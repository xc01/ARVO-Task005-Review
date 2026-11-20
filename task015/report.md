# Task 015 Report — WebOb / PatchAudit reproduction-material audit

## A. Commit relations confirmed
- C1 `2b9fbedafb31180c910cf8526e9ea72b4603d0bc` (2026-05-06) — parent `bb1515b5`.
- C2 exists as two parallel copies of the same fix: main-line `a6c1422948131b9d679b341f72dab3c146bc8600` (parent `f2184c8b`) and advisory/fork `ff89560643fb252751b4db8806a283b5377f1f07` (parent `54c1133c`). Same author/date/message; not ancestor-related; merged to main via `3a5fd961`.
- Later commit `d3077b84` (parent of C2 on main) is a functional follow-up (degenerate-input short-circuits), not a security advisory. No further Location security fixes through 2026-08-03.

## B. P1/P2 and test material
- P1 (tab bypass) payloads ship inside C1's own regression tests.
- P2 (leading C0/space bypass) payloads ship in the fork regression tests at `54c1133c` (`test_location_no_open_redirect_c0_control_or_space`) and are documented by GHSA-6hx8-3wjj-gr8g.
- Oracle: public-API location host preservation; pure pytest, executable material available but NOT run in this task.

## C. C1 incomplete-fix evidence
Static evidence sufficient: C1's own diff shows the exact guard set that P2 bypasses; the official advisory and the C2 diff confirm the missing normalization classes (leading C0/space). `patch_type=incomplete` in PatchAudit; pipeline verdict `future-commit/incomplete` (6 runs consistent).

## D. 13 zero-day PoC cases
Directory listing + READMEs only (payloads untouched). All 13 CVEs present in the 617 dataset with C1/C2 and root-cause docs. Fix status: 2 conflicting with GH advisories (CVE-2023-31143, CVE-2023-45897, CVE-2025-53010), rest `patch_status_unknown` from this task's allowed metadata-only checks.

## E. Mechanism comparison
Common abstract rule: validate-before-normalize/dereference ordering. WebOb = string normalization vs URL validation; HarfBuzz = offset dereference vs bounds check. Facts recorded; no transferability claim.

## Downloads
~170 KiB total (raw/): 24 files, SHA-256 in command_log. Well within 30 MiB. No PoC payloads, no models, no Docker.
