# Six independent end-to-end runs (2026-10-23)

`run1`–`run5` were made with `SKIP_STATIC=1` (static prong silent); `run6_with_static/` is the
`run_case.sh … all` run with the CodeQL/Semgrep stage (its `static_verdict.auto.json` is included) and is the
source of `../expected/scan_summary.jsonl`. Each run is one complete `run_agent.py --case case_CVE-2026-44889` scan of all later commits of C1
(`2b9fbedafb`), with the fine-tuned Phase-1/Phase-2 adapters served by vLLM and GPT-5 as the Phase-3 judge.
Same inputs, different samples — kept so the variance of the LLM stages is visible.

| file | content |
|---|---|
| `summary.json` | one line per run: C2 verdict, patch type, per-round trace, any other commit flagged (none) |
| `agent_inputs_C2/` | exactly what the Phase-2 agent was given for C2: `cve_description.json`, `commits_data.jsonl` (C1/C2 messages + diffs), `slicing.jsonl` (intersecting slice snippets, `shared=44`, `core=2`) |
| `runN/phase23_loop.jsonl` | raw per-candidate records (Phase-1 classifier logprobs, agent retrieval rounds + final analysis incl. `<think>`, every judge text, dual-consistency prongs, final report) |
| `runN/scan_summary.jsonl` | one row per later commit (same shape as `../expected/scan_summary.jsonl`) |
| `runN/stdout.log` | console transcript of the run (Phase-1 refs/confidence, per-round judge outcome, tool calls) |
| `runN/C2_a6c1422/phase1_classifier.json` | Phase-1 reference extraction + intent classifier output for C2 |
| `runN/C2_a6c1422/agent_retrieval_rounds.json` | the agent's planning / information-sufficiency rounds and the MCP tool calls it issued |
| `runN/C2_a6c1422/agent_final_analysis.md` | the Phase-2 agent's final analysis text (last re-analysis) |
| `runN/C2_a6c1422/roundK_judge_gpt5.md` | GPT-5 judge output for round K (audit of claims, grounding, AGREE/DISAGREE) |
| `runN/C2_a6c1422/final.json` | final decision + ClassifyAndExplain report |
| `run6_with_static/agent_collector.json` | the agent's full working memory for that run: task info given to it, the generated plan, every MCP tool call with its raw output, retrieval rounds, final analysis (the five parallel runs shared this file, so only the last run's copy survives) |
| `run6_with_static/static_verdict.auto.json` | Phase-3 static prong: CodeQL/Semgrep states at the four snapshots (inconclusive, no CWE-601 finding) |

Result: **6/6 runs → C2 `a6c1422` = future-commit (`incomplete`)**, 0 other commits flagged in any run.
Rounds 2/3 differ between runs (the sampled agent twice flipped to NO / once returned no parsable decision,
and the judge in a few rounds answered with a bare `DECISION/PATCH_TYPE/CONFIDENCE` block that is recorded
as `grounding=?`); the final decision is always the judge-agreed round-1 prediction via the §3.4.3 fallback,
because the static prong stayed silent (these five runs were made with `SKIP_STATIC=1`, `static_vote = n/a`; the shipped `expected/` run includes the CodeQL/Semgrep stage).
