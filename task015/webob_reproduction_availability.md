# Task 015 — WebOb Reproduction Material Availability

## Evidence matrix (per Planner standard)

| condition | status | evidence |
|---|---|---|
| A0+P1 vulnerable (CVE-2026-44889, tab bypass of CVE-2024-42353 fix) | supported_by_source (static) | GHSA-fh3h-vg37-cc95 advisory text; C1 message; C1 regression tests encode P1 payloads `/\t/…`, `\t//…`, `//\t…` |
| A1(C1)+P1 blocked | supported_by_source (static test, not dynamic) | C1 ships `test_location_no_open_redirect_tab_bypass` + `test__make_location_absolute_strips_url_whitespace` covering the same payloads with host-preserving expectations |
| A1(C1)+P2 vulnerable (C0/space bypass, CVE-2026-54770) | supported_by_source (static) | GHSA-6hx8-3wjj-gr8g; fork regression test `test_location_no_open_redirect_c0_control_or_space` parametrizes `" //evil…"` etc.; C2 commit message references the advisory |
| A2(C2) removes root cause | supported_by_source (static) | C2 removes `urlparse.urljoin` from all three call sites and adds `webob/util.py` urljoin with explicit no-whitespace-stripping tests |
| Our dynamic verification | not_established | Task 015 forbids running tests/PoC; nothing has been executed |

## Concrete P1 / P2 payload sets (from official tests; no new PoC authored)
- P1 (C1-era, tab bypass): `/\t/www.example.com/test`, `\t//www.example.com/test`, `//\twww.example.com/test`, `/\t\t/…` (C1 diff, `raw/c1.diff` lines 58–66)
- P2 (C2-era, C0/space bypass): `" //www.example.com/test"`, `"\x00//…"`, `"\x1f//…"`, `" \t//…"`, `" //… "` and `" https://www.example.com/test"` (fork `tests/test_response.py` at `54c1133c`; documented in raw/README.md and GHSA-6hx8-3wjj-gr8g)

## Executable materials present
- Official pytest regression tests ship in the repo at both C1 and C2 snapshots (public API level: `Response.location` + `Request.blank`). Pure Python, no network needed for the unit-level oracle.
- PatchAudit `examples/case_CVE-2026-44889/` provides the pipeline inputs (case.json, commits_data.jsonl, slicing.jsonl) and six recorded agent runs — not PoC inputs; its `expected/scan_summary.jsonl` is the pipeline oracle.

## Version / fix info
- GHSA-fh3h-vg37-cc95: webob ≤ 1.8.9 vulnerable, first patched 1.8.10.
- GHSA-6hx8-3wjj-gr8g: webob < 1.8.11 vulnerable, first patched 1.8.11.
- Latest WebOb release at fetch time is 1.8.11-line (post `c0d70f98`), so CVE-2026-44889/54770 are considered fixed upstream.
