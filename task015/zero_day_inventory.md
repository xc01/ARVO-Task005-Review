# Task 015 — 13 zero-day PoC cases: metadata-only inventory

Source: HF `data/zero_day_pocs/` directory listing + `README.md` (both fetched; **no PoC payload files downloaded or displayed**). Per-task constraint, payloads were not opened.

Only `readme.md`/`description.md`/root-cause documents were retrieved (11 files, all metadata documents).

| CVE | project (from PatchAudit README) | language (from README/extension) | PoC artifact dir (listed only) | C1/C2 relation recorded in 617 dataset | root-cause report shipped | affected version (README) | fixed status |
|---|---|---|---|---|---|---|---|
| CVE-2021-44878 | pac4j/pac4j | Java | pac4j-oidc-none-bypass/, description.md | incomplete | yes (description.md) | ≤6.3.2 | GH advisory not fetched; **patch_status_unknown** beyond dataset claim |
| CVE-2023-23144 | gpac/gpac | C | gpac_report/ (payloads listed, not opened) | incomplete | yes | — (report only) | **patch_status_unknown** (no advisory fetched) |
| CVE-2023-26145 | dgilland/pydash | Python | poc.py, readme.md | incorrect | yes | 8.0.5 | **patch_status_unknown** |
| CVE-2023-27586 | Kozea/CairoSVG | Python | poc.py, readme.md | incomplete | yes | 2.8.2 | **patch_status_unknown** |
| CVE-2023-31143 | mage-ai/mage-ai | Python | poc.py, readme.md | incomplete | yes | ≥0.8.72 | GHSA-c6mm-2g84-v4m7: range >=0.8.34,<0.8.72 → **inconsistent with README claim**; advisory says patched 0.8.72 — **patch_status_conflicting** |
| CVE-2023-33977 | kiwitcms/Kiwi TCMS | Python/HTML(svg) | malicious_*.svg (listed, not opened) | incomplete | yes | v15.2 | **patch_status_unknown** |
| CVE-2023-45897 | exfatprogs/exfatprogs | C/shell | poc.sh, readme.md | incomplete | yes | 1.3.1 (fsck.exfat) | GHSA-5qgg-v88w-rgh6 exists (before 1.2.2 range) — **patch_status_conflicting** with README's 1.3.1 claim |
| CVE-2023-48795 | Terrapin/ConnectBot | Java | ConceptualExploitDemo.java (listed, not opened) | incomplete | yes | — | protocol-level; upstream fixed in SSH implementations; **patch_status_unknown** per-case |
| CVE-2023-50266 | morpheus65535/bazarr | Python | poc.py, readme.md | incorrect | yes | 1.5.3 | no GH advisory found → **patch_status_unknown** |
| CVE-2023-51441 | apache/axis-axis1-java | Java | apache-axis-jndi-disclosure/ (listed, not opened) | incorrect | yes (dir) | 1.8.0 | **patch_status_unknown** |
| CVE-2024-35230 | geoserver/geoserver | Java | discription.md | incorrect | yes | — | **patch_status_unknown** |
| CVE-2025-53010 | AcademySoftwareFoundation/MaterialX | C++ | description.md | incomplete | yes | ≤1.39.4; README: still in latest main | GHSA-3jhf-gxhr-q4cx: =1.39.2 patched 1.39.3 — **conflicts** with README claim; treat README claim as superseded → patched upstream per advisory |
| CVE-2025-57804 | python-hyper/h2 | Python | description.md | incomplete | yes | — | **patch_status_unknown** |

Notes:
- All 13 CVEs are in the 617 bad-patch dataset (`all_positive_617.json`); none are in the 12 re-exploitation metadata set (as stated, the two sets are distinct).
- The dataset provides `c1_hash`/`c2_hash` for all 13 (verified in Task 014 scan).
- "Still in latest main" claim only appears for CVE-2025-53010 in the official README; where a GH advisory contradicts it, recorded as `conflicting`.
- No exploit payloads were downloaded, displayed, or executed.
