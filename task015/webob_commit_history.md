# Task 015 — WebOb Commit History (verified from GitHub API)

## C1 — initial patch (CVE-2026-44889 / GHSA-fh3h-vg37-cc95)
- Full SHA: `2b9fbedafb31180c910cf8526e9ea72b4603d0bc`
- Date: 2026-05-06T06:38:51Z
- Parent: `bb1515b5f1a480958ec7b6849153b2a6efb28028`
- Message: "Fix open redirect issue due to changes made in cPython >=3.10"
- Files: CHANGES.txt (+9), src/webob/response.py (+8/-3), tests/test_response.py (+52)
- Full diff: `raw/c1.diff`

## Pre-C2 test commit
- `f2184c8b475ade2f6761f72d467cc316a3483b34` — 2026-06-09T20:10:43Z, parent of C2, "Add regression tests for open redirect via Location normalization"; touches only tests (`tests/test_exc.py` +58, `tests/test_response.py` +48). Its tests were added in the fork branch that produced the advisory fix; see fork relationship below.

## C2 — follow-up fix (CVE-2026-54770 / GHSA-6hx8-3wjj-gr8g)
Two related commits exist with the same message; they are on different parent chains:

### C2a (main branch, used by PatchAudit example)
- Full SHA: `a6c1422948131b9d679b341f72dab3c146bc8600`
- Date: 2026-06-09T20:12:20Z
- Parent: `f2184c8b475ade2f6761f72d467cc316a3483b34` (the regression-test commit)
- Files: CHANGES.txt (+19), src/webob/exc.py (+9/-2), src/webob/request.py (+6/-2), src/webob/response.py (+12/-6), src/webob/util.py (+172), tests/test_util.py (+161/-1)
- Full diff: `raw/c2.diff`

### C2b (security fork merge base; advisory-referenced commit)
- Full SHA: `ff89560643fb252751b4db8806a283b5377f1f07`
- Date: 2026-06-09T20:12:20Z (same timestamp, same message)
- Parent: `54c1133c79e48940776a272f4500530e1cc3232a` (fork-side test commit, same tests, different SHA from `f2184c8b`)
- Files: CHANGES.txt (+25), src/webob/exc.py (+8/-2), src/webob/request.py (+6/-1), src/webob/response.py (+18/-7), src/webob/util.py (+184), tests/test_util.py (+165)
- Full diff: `raw/ff895606.diff`
- Note: `ff895606` ≠ `a6c14229`. Same author/date/message, different parent chains; the code changes are near-identical but not byte-identical (diff of diffs ≈ 215 lines, mostly CHANGES.txt hunk offsets and small exc.py/response.py deltas).
- The fork branch was later merged to main as `3a5fd961942c053a56fb60396059d4f2a80f3a40` ("Merge commit from fork", 2026-08-02, merge parents `98770ea9` + `d3077b84`).

## Relationship determination
- `a6c14229` and `ff895606` are **parallel copies of the same fix** on two lines of history (main vs. security fork). Neither is the Git ancestor of the other; both descend from different copies of the same test commit.
- The PatchAudit example uses the main-line `a6c14229`; the CVE-2026-54770 advisory (`GHSA-6hx8-3wjj-gr8g`) corresponds to the fork commit `ff895606` merged via `3a5fd961`.

## C3 / later security fixes
- `d3077b8444f95fddefa6cbdf29b579b0fddcc62c` (2026-06-09T20:33:34Z, parent `a6c14229`) — "Mirror urllib.parse.urljoin()'s short-circuits for degenerate input"; adds None/empty-string handling in `webob/util.py` for API compatibility. Functional follow-up, not a security advisory.
- Post-merge history through 2026-08-03 (`c0d70f98` HEAD at fetch time) contains no further Location/URL security commits.
- Conclusion: the C1→C2 chain is the complete security fix chain for CVE-2026-44889/CVE-2026-54770 as of 2026-08-03.
