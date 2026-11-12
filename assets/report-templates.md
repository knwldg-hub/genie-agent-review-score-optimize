# Report templates

Use these layouts exactly. Never put data values in them (golden rule 3).

## Quick scan (Step 1.5)

**Quick scan: <title>** · config checks x/10 passed · provisional tier <tier>

Take the tier from `qs["provisional_tier"]`; never work it out yourself. If check 5 is the only failure (single-source agent), say the tier is Not Ready because of the Workbench join-spec rule, not because of a real gap.

| What it has | |
|---|---|
| Data sources | n tables · n metric views |
| Instructions | n characters · n example SQLs · n SQL snippets · n join specs |
| Columns | n configured · n with synonyms · n with entity matching |
| Benchmarks | n questions (n with expected SQL) |
| Last full benchmark run | x% on <date> (n questions), or "none yet" |
| Benchmark quality | n expected SQLs use `CURRENT_DATE()`-style dates · question types below target: … (heuristic, from question text and SQL) |

- ✅ **Looks good**: passed checks, for example "All 15 configured columns are described"
- ⚠️ **Configured, could be better**: passed with a warning, for example "Instructions are 3,795 characters; about 2,000 is recommended"
- ❌ **Missing**: failed checks, for example "No SQL snippets (measures, filters, expressions)". Only items with evidence from a helper result, `EXPLAIN` or a benchmark run (golden rule 12); suspicions go under ❓.
- ➖ **Not applicable**: for example join specs on a single-source agent
- ❓ **Not measured yet**: accuracy (IQ 11–12), example SQL and snippet validity (D3), SQL logic (D4, D6), guardrail behavior, plus anything you suspect but haven't checked, worded as a question

Then one line **Advisory checks (don't change the score)** with each item as ✅ or ⚠️, and up to 3 likely quick wins.

## Baseline scorecard (Step 4)

| Area | Baseline |
|---|---|
| IQ score | x/12 (tier: Not Ready / Ready to Optimize / Trusted) |
| Deep review (D1–D13) | n ✅ · n ⚠️ · n ❌ |
| Benchmarks (native run) | x% (passed/n) · 1 question = y pts |
| Failing questions | n stable · n flaky |
| Other benchmark runs (step 5) | x% (mode as reported), or "none" |
| Benchmark coverage | n questions (n with expected SQL) · ✅ / ⚠️ / ❌ from step 3.5 |
| Critical questions | n marked · n passing |
| Guardrails | x/5 passed, or "not run" |
| Readiness | ready / not ready (which items fail) |

Below the table, list every failed IQ check, every ❌ or ⚠️ deep-review check, and the manual checks, one line each.

## Recommendations (Step 6)

| ID | Recommendation | Impact | Effort | Moves | Fix level | Type |
|---|---|---|---|---|---|---|

## Before/after (Step 9)

| Area | Before | After | Change |
|---|---|---|---|
| IQ score | | | |
| Tier | | | |
| Deep review (✅ / ⚠️ / ❌) | | | |
| Benchmarks (native run) | | | |
| Failing questions (stable / flaky) | | | |
| Other benchmark runs | | | |
| Benchmark coverage (count) | | | |
| Benchmark corrections | | | |
| Critical questions passing | | | |
| Guardrails | | | |
| Readiness | | | |

Then list each check that changed status, and any regression flags with a rollback offer.

## Final review report (Step 10)

Save as `{DIR}/review_report_{date}.md` with:
- Agent details, apply mode, and the backup path
- Baseline and final scorecards
- Changes made and their status (applied or PR-ready), benchmark corrections listed separately
- Remaining recommendations, ordered
- Readiness checklist, regressions (and any overrides), guardrail results, still-flaky questions
- Advisory check items
- How to roll back
