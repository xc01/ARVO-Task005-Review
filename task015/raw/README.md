# Example (new, 2026): CVE-2026-44889 — WebOb `Location` open redirect, fixed twice

A **freshly disclosed** bad patch, **not part of the paper's dataset** — the Reusability input. Everything
here happened in 2026: the initial patch `C1 = 2b9fbeda` (2026-05-06, advisory GHSA-fh3h-vg37-cc95 /
CVE-2026-44889 published 2026-06-04), the follow-up `C2 = a6c14229` (2026-06-09), and the advisory that
documents why `C1` was insufficient (GHSA-6hx8-3wjj-gr8g / CVE-2026-54770, published 2026-08-27). Manual review
confirms exactly **one** later commit continues the fix, so `C1` is a bad patch (`N = 1` future commit). It is
pure Python, the repo is 7 MB, and the pipeline needs no `snapshot_paths` scoping.

## The bad patch

`Response._make_location_absolute()` (`src/webob/response.py`) turns a redirect target into an absolute URL. The
2024 fix (CVE-2024-42353) rewrote a leading `//` to `/%2f` so that `urllib.parse.urljoin()` would not treat the
value as protocol-relative. Since Python 3.10 `urlsplit()` strips tab/CR/LF before parsing, so `/\t/evil` still
became `//evil`. `C1` strips those three characters first:

```python
# C1 (2b9fbeda) — strip the characters urlsplit() would strip, then run the old checks
value = value.replace("\t", "").replace("\r", "").replace("\n", "")
if SCHEME_RE.search(value):          # ^[a-z]+:  -> already absolute
    return value
if value.startswith("//"):           # neutralise protocol-relative
    value = f"/%2f{value[2:]}"
new_location = urlparse.urljoin(_request_uri(environ), value)   # still the stdlib
```

But `urlsplit()` also strips **leading and trailing C0 control characters and spaces**. A value of
`" //evil.example/x"` passes both guards (it starts with a space, not `/`), and `urljoin()` then removes the space
and returns `http://evil.example/x` — the fix is **incomplete**. `C2` replaces the stdlib call in the same
function with WebOb's own RFC 3986 `webob.util.urljoin()`, which resolves the reference exactly as given, and
routes `Request.relative_url()` and the `HTTPFound`-style exceptions through the same code path.

## What the pipeline does here

`later_commits: "all"` scans the 19 commits after `C1` on `main`. Extraction keeps the 9 that change code
(the rest are merges, `CHANGES.txt`, `.mailmap`, CI). Only `C2` edits `_make_location_absolute`, so only `C2`
intersects `C1`'s slice (`shared=44`, `core=2`, in `response.py`/`request.py`); the other eight are
slice-filtered without an agent call. `C2`'s message links GHSA-6hx8-3wjj-gr8g and says it "makes the regression
tests added in the previous commit pass", so Phase 1 extracts the references, the `phase1` classifier scores it
**continues** (≈ 0.91 ≥ 0.85), and Phase 3 verifies that text prediction first. The GPT-5 judge agrees, but the
vulnerability-state prong is silent — CodeQL/Semgrep report no CWE-601 finding on any of the four snapshots (the
only in-scope hit is an unrelated `py/bad-tag-filter` in `exc.py`) — so the verdict is **unresolved** and the
Phase-2 agent re-analyses with the code evidence (function/variable retrieval over the repo), answering
**YES / incomplete**. The loop spends its three iterations and falls back to the judge-agreed prediction
(§3.4.3): **future-commit**. See [`expected/scan_summary.jsonl`](expected/scan_summary.jsonl):

| later commit | Phase 1 | Intent Scope | agent | Phase 3 | final |
|---|---|---|---|---|---|
| f2184c8b (regression tests) | refs, classifier NO (0.68, uncertain) → Phase 2 | `shared=0` | — | — | not-future-commit (filtered) |
| **a6c14229 (C2)** | refs → **continues 0.91** → Phase 3 | `shared=44`, `core=2` | **YES / incomplete** | unresolved × 3 (static blind) → fallback | **future-commit** |
| d3077b84 (util.py short-circuits) | NoReference → Phase 2 | `shared=0` | — | — | not-future-commit (filtered) |
| 79efe121, 6ba68859, 2d0505a9, bcc693a3 | refs → NO (0.999) → Phase 3 unresolved → Phase 2 | `shared=0` | — | — | not-future-commit (filtered) |
| 8e30eeaf, 7668e36a (utcnow) | NoReference → Phase 2 | `shared=0` | — | — | not-future-commit (filtered) |

The reproducible result is the primary decision: `C2` = future commit → `C1` = bad patch, and no other commit is
flagged. We ran the scan **six times** (five with the static stage skipped, one with it); all six returned
`future-commit / incomplete` for `C2` and nothing else — the full intermediate outputs (Phase-1 classifier,
agent retrieval rounds and analyses, every judge response) are kept in [`runs/`](runs/). What moves between
runs is the middle of the loop: the sampled agent occasionally answers NO or returns no parsable decision on a
re-analysis (the judge rejects it and the next round is YES again), and the judge sometimes answers with a bare
`DECISION/PATCH_TYPE/CONFIDENCE` block instead of an audit, which counts as a disagreement for that round.

## Run it

```bash
git clone https://github.com/Pylons/webob repos/webob
chmod +x tools/*.sh tools/agent/*.sh            # once (a HF/Zenodo download drops executable bits)
tools/vllm_server.sh start                      # Qwen3-14B + phase1/phase2 adapters (GPU)
export OPENAI_API_KEY=...                        # GPT-5 judge
docker run --rm --network host -e OPENAI_API_KEY -e VLLM_BASE_URL=http://localhost:8000/v1 \
  -v $PWD/examples:/cases -v $PWD/repos/webob:/repos/webob:ro \
  patchaudit-analysis:latest /cases/case_CVE-2026-44889 all
```

About 12 minutes on one GPU: extraction + Joern slicing of the 9 candidates ≈ 1.5 min, CodeQL + Semgrep on the
four snapshots ≈ 5 min (skip with `-e SKIP_STATIC=1`), Phase 1 on the candidates + the agent/judge loop for `C2`
≈ 6 min. Add `--stop-at-first-yes` to
stop at `C2`. Compare `result/phase23_loop.*.jsonl` (or the console `OUTPUT` lines) with
`expected/scan_summary.jsonl`.
