# Reproduce: measurement-study tables & figures

Scripts + data + rendered outputs for the paper's large-scale measurement-study figures and tables.
Run each `.py` from **this directory** (Python 3.12, `matplotlib` + `numpy` + `pandas`); every script reads
its inputs from alongside itself and writes its output here.

## Scripts → outputs

| script | paper artifact | output |
|---|---|---|
| `generate_fig4.py`  | Fig. 4 — bad-patch year distribution | `fig4_year_dist.tex` (pgfplots) |
| `generate_fig5.py`  | Fig. 5 — CWE distribution of bad-patch CVEs | `fig5_cwe_dist.tex` (pgfplots) |
| `generate_fig6.py`  | Fig. 6 — C1-metric violins, 3 patch types × 6 params | `fig6_3types_6params.pdf` |
| `generate_fig12.py` | Fig. 12 — re-exploitation timeline | `fig12_reexploitation_timeline.tex` |
| `generate_fig13.py` | Fig. 13 — C2-metric violins, 2 types × 3 params | `fig13_2types_3params.pdf` |
| `generate_table4.py`| Table 4 — bad-patch characteristics before vs. since 2020 | `table4_before_after_2020.tex` |
| `generate_table5.py`| Table 5 — failure-cause taxonomy | `table5_failure_cause_taxonomy.tex` |
| `generate_table6.py`| Table 6 — severity of defects in incorrect patches | `table6_consequence_severity.tex` |

The `.tex`/`.pdf` files already here are the **rendered artifacts used in the paper**; re-running a script
reproduces its own output. The scripts are the paper's originals, shipped unmodified; each one reads its input
from this directory (`BASE = Path(__file__).parent`), so run them from here.

## Data

| file | contents |
|---|---|
| `all_positive_balance_v2_617.json` | the 617 positive (bad-patch) entries — CVE/C1/C2, dates, CWE, `c1_metrics`, `with_ref` |
| `all_combined_v2.json` | all 6535 entries (617 positive + 5918 negative), with `patch_type` and, for the positives, `c1_metrics` — the single combined dataset; input to Fig. 4/6/13 and Table 4 |
| `all_negative_c1_metrics_progress.json` | per-CVE C1 metrics of the negatives (good patches: lines/hunks/files changed, developers, `c1_date`), keyed by CVE id — the Good-patch series of Fig. 6 is read from here |
| `reexploitation_cases.json` | the re-exploitation cases (Fig. 12) — cause, `evidence_source`, `evidence_id` |

`enrich_c1_metrics_local_or_api.py` / `enrich_negative_c1_metrics.py` are the metric-enrichment helpers that
built the `c1_metrics` fields and the negatives' progress file from the GitHub API; the enriched JSON above is
shipped, so the figure/table scripts run without re-enriching.

## Run

```bash
pip install matplotlib numpy pandas          # into your Python 3.12 env
cd data/reproduce
python3 generate_fig4.py  && python3 generate_fig5.py
python3 generate_fig6.py  && python3 generate_fig13.py     # -> fig6/fig13 PDFs
python3 generate_fig12.py                                   # -> fig12 .tex (re-exploitation)
python3 generate_table4.py                                  # -> table4 .tex (before vs. since 2020)
python3 generate_table5.py && python3 generate_table6.py    # -> table5/table6 .tex
```
