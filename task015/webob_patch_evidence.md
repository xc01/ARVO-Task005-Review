# Task 015 — WebOb Patch Evidence (C1 incomplete-fix mechanism)

## Vulnerable function
`Response._make_location_absolute()` in `src/webob/response.py` (CWE-601 open redirect).

## C1 content (verified from `raw/c1.diff`)
C1 strips ASCII TAB, CR, LF **before** the existing checks, then applies the CVE-2024-42353 logic:
```python
value = value.replace("\t", "").replace("\r", "").replace("\n", "")
if SCHEME_RE.search(value):
    return value
if value.startswith("//"):
    value = f"/%2f{value[2:]}"
new_location = urlparse.urljoin(_request_uri(environ), value)
```

## Why C1 is incomplete (evidence chain)
- GHSA-6hx8-3wjj-gr8g / CVE-2026-54770 (published 2026-08-27): "Open redirect in Location header normalization via leading C0 control / space characters". Affected `< 1.8.11`, first patched 1.8.11.
- The regression test added in the fork branch (`test_location_no_open_redirect_c0_control_or_space`, in `tests/test_response.py` at `54c1133c`) parametrizes payloads including `" //www.example.com/test"`, `"\x00//www.example.com/test"`, `"\x1f//www.example.com/test"`, `" https://www.example.com/test"` with expected results keeping the redirect on the original host. These payloads pass C1's guards (they do not start with `//`, TAB/CR/LF stripping does not affect leading space/NUL/C0), and Python ≥3.10 `urlsplit()` strips leading/trailing C0 control + space before parsing, promoting them to protocol-relative/absolute URLs.
- C2 replaces `urlparse.urljoin` with WebOb's own RFC 3986 `webob.util.urljoin()` in `response.py`, `request.py` (`relative_url`), and `exc.py` (redirect exceptions), plus a 172-line pure-Python implementation and tests asserting that whitespace/NUL are treated literally (e.g. `(" //www.example.com/test", "http://a/b/c/ //www.example.com/test")`).

## C1 vs C2 same code path
- Both modify `Response._make_location_absolute` in `src/webob/response.py`; C2 also routes `Request.relative_url()` and redirect exceptions through the same WebOb urljoin. The PatchAudit slicing confirms `shared=44, core=2` between C1 and C2 (raw/scan_summary.jsonl).

## Oracle
- The C2 test `test_location_no_open_redirect_c0_control_or_space` expects `req.get_response(res).location` to remain on the request host for C0/space-prefixed payloads — usable as the pass/fail oracle for A1+P2.

## C1 does block P1 (tab bypass)
C1's own regression tests (`test_location_no_open_redirect_tab_bypass`, `test__make_location_absolute_strips_url_whitespace` in `raw/c1.diff`) cover TAB/CR/LF payloads and assert host preservation. These tests ship inside C1 itself, so A0+P1 (tab bypass) is asserted fixed by C1's test suite. Per-task standard this is **static/test evidence**, not our dynamic verification.
