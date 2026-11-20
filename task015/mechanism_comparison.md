# Task 015 — Mechanism comparison: WebOb vs HarfBuzz (facts only)

## WebOb (CVE-2026-44889 / CVE-2026-54770)
- Mechanism: URL parser **normalization before validation**. Python ≥3.10 `urllib.parse.urlsplit()` removes TAB/CR/LF anywhere and strips leading/trailing C0+space **before** C1's checks (`SCHEME_RE` match, `startswith("//")`) run, so a value like `" //evil"` bypasses both guards and becomes a protocol-relative URL after normalization. (Evidence: C1 diff comments, GHSA-6hx8-3wjj-gr8g, fork regression tests.)
- Fix direction: remove normalization-before-check entirely by replacing stdlib `urljoin` with a strictly RFC 3986 implementation that does not mutate the input (C2).
- Character class: **validation-vs-normalization ordering** on string inputs; no memory bounds, no offsets.

## HarfBuzz (ARVO 42470379, verified in Task 012)
- Mechanism: `ResourceMap::sanitize` computed `&(this+typeList)` **before** `check_struct(this)` validated the structure bounds; the offset dereference happened on an unvalidated object, producing heap-buffer-overflow READ 1 in `BEInt<unsigned short,2>::operator unsigned short` via `OffsetTo::operator() → Offset::is_null`. (Evidence: Task 012 vul_stderr.txt; source diff.)
- Fix direction: move the offset evaluation **after** the structural check (C2 diff, Task 012 source_comparison.diff).
- Character class: **check-order vs offset dereference** on binary structures; memory-safety oracle (ASan).

## Common security rule (fact-level)
Both are instances of the abstract ordering rule: **validate the input/object before applying transformations that assume validity** (WebOb: parse/normalize before trust checks; HarfBuzz: offset dereference before bounds check). In both cases C1 kept the *ordering* wrong (or wrong normalizer semantics) and C2 fixed the ordering itself — i.e. the incomplete patch was not a missing branch but a misplaced/mis-scoped validation step.

## Differences (do not conflate)
- WebOb: pure string semantics, deterministic unit-test oracle, no sanitizer.
- HarfBuzz: binary memory safety, ASan oracle, struct offsets.
- Shared abstract rule exists; no claim of transferability is made here.
