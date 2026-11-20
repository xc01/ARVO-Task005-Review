# Task 014 — Data Sources

All files saved under `results/task014/raw/` with SHA-256 (see also command log).

| file | source | size (B) | SHA-256 |
|---|---|---:|---|
| readme.md | https://raw.githubusercontent.com/SRestLabUB/PatchAudit/main/readme.md | 16,382 | 7517f81b9adc899f34066ee255f4a8549248076ceebe5bccdbb0ba011d3caa44 |
| all_positive.json | repo `measurement study/all_positive.json` (main branch, 2026-11-20 fetch) | 988,033 | 956320e1f65c8943e75ea8c5fa4167a816bb7895b394671a9658bd7b1bd0d2fe |
| badpatch_bench_final.csv | repo `badpatch-bench/badpatch_bench_final.csv` | 66,241 | 07256e91c5b498770467c43d91556eaf3a10ef9364245145f1e3dc2e7ecbcfc3 |
| hf_data_README.md | HF dataset zhcharyzhang/patchaudit-artifact `data/README.md` | 4,229 | 8307ae60cc35620143694386863cc95f83c60b2dc6ee36498aeee14f95479d85 |
| hf_reproduce_README.md | HF dataset `data/reproduce/README.md` | 3,054 | 9c36d20c052016bdc9be22a620ac4ea4f2f52be107279c7b6987d54c764c67e3 |
| reexploitation_cases.json | HF dataset `data/reproduce/reexploitation_cases.json` | 13,993 | a5e5a8800382ab76f397bbeffa56def0e03cf520a242969e73ee3c65653e3afe |
| all_positive_617.json | HF dataset `data/reproduce/all_positive_balance_v2_617.json` | 989,290 | 6980799cd88981e5108ce6953928c09757ed040b0b454ff1009f50107f9559fb |
| badpatch_bench_full.jsonl | HF dataset `data/badpatch_bench_full.jsonl` | 1,100,561 | 295892285414bf3d04a74415b22008aa607cce6a6e2c25084583ff752e9a120b |

Total downloaded: ~3.1 MiB (well under the 50 MiB limit).

Notes:
- Zenodo record 23654079 returned HTTP 403 on the API endpoint; no Zenodo files were downloaded.
- The GitHub `measurement study/all_positive.json` and HF `all_positive_balance_v2_617.json` differ in size/SHA. The HF file is the documented "617 positive" reproduction dataset per `hf_reproduce_README.md` and was used for §7 statistics; the repo file was retained as a source-variant record.
- No large archives, Docker images, models, PoC packages, or git repos were downloaded.
