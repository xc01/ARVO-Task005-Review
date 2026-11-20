# Zero-day / re-exploitation PoCs

Proof-of-concept exploits for **13 vulnerabilities** that PatchAudit surfaced by analyzing faulty security
patches: in each case a CVE's "fix" was a **bad patch** (incomplete or incorrect), leaving the original
vulnerability — or a sibling of it in the same code — still reachable. Several are documented as **still
present in the latest release** at the time of writing (a *lingering zero-day*). These PoCs substantiate the
paper's claim that faulty security patches leave lingering zero-days at scale.

> This set is **distinct** from `../reexploitation_cases.json` (the 12 re-exploitation *measurement* cases,
> a labeled metadata table). These 13 are **working PoC artifacts** (exploit code + reports); the two CVE
> sets do not overlap.

| CVE | Target | Vulnerability | PoC |
|---|---|---|---|
| CVE-2021-44878 | pac4j-oidc ≤ 6.3.2 | Case-insensitive `none`-algorithm bypass — incomplete OIDC unsigned-token guard (defense-in-depth failure) | `pac4j-oidc-none-bypass/` (Maven project), `description.md` |
| CVE-2023-23144 | GPAC | Integer / heap overflow in scene-graph parsing | `gpac_report/gpac_overflow_poc.c`, `*.bt` payloads, logs |
| CVE-2023-26145 | pydash 8.0.5 | Inconsistent security-check validation (`invoke` / `invoke_map`) | `poc.py` |
| CVE-2023-27586 | CairoSVG 2.8.2 | Server-side request forgery (SSRF) | `poc.py` |
| CVE-2023-31143 | Mage-AI ≥ 0.8.72 | Terminal authentication bypass | `poc.py` |
| CVE-2023-33977 | Kiwi TCMS v15.2 | Incomplete SVG-upload patch (stored XSS vector) | `malicious_*.svg` |
| CVE-2023-45897 | exfatprogs 1.3.1 (`fsck.exfat`) | Out-of-bounds heap read on UTF-16 filenames | `poc.sh` |
| CVE-2023-48795 | SSH "Terrapin" (ConnectBot) | Rekey / protocol downgrade | `ConceptualExploitDemo.java` |
| CVE-2023-50266 | Bazarr 1.5.3 | Unauthenticated SSRF | `poc.py` |
| CVE-2023-51441 | Apache Axis 1.8.0 | JNDI injection / SSRF via CORBALOC | Maven project (`src/`, `pom.xml`) |
| CVE-2024-35230 | GeoServer | Information disclosure (`AboutGeoServerPage`) | `discription.md` |
| CVE-2025-53010 | MaterialX ≤ 1.39.4 | Null-pointer dereference in `getShaderNodes()` — still in latest `main` | `description.md` |
| CVE-2025-57804 | h2 (Python HTTP/2) | HTTP request smuggling — CRLF unvalidated in `validate_outbound_headers()` | `description.md` |

Each subdirectory has its own `readme.md` / `description.md` with the root-cause analysis (which patch was
faulty and why), the affected version, and how to run the PoC.

## Responsible use

These PoCs are for **verification of the paper's findings and defensive research only**. All entries carry
public CVE identifiers; entries noted as *still present in the latest version* were handled under responsible
disclosure. Do not run them against systems you do not own or are not authorized to test.
