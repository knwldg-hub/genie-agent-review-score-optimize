# Benchmark evaluation

**Primary: native benchmark runs (`run_benchmarks`).** Uses the Genie eval API (`genie_create_eval_run`, `genie_get_eval_run`, `genie_list_eval_results`, `genie_get_eval_result_details`), the same engine as the Benchmarks tab. Each question comes back `GOOD` or `BAD` (or needs review) with Genie reason codes (`references/reason-to-fix.md`). Pass `question_ids` to run a subset for targeted re-runs. Runs show up in the Benchmarks tab history. In testing, 2 questions took about 35 seconds.

**Fallback: chat-mode harness (`chat_benchmarks`).** Use only if the eval API isn't available. For each question with expected SQL it asks the agent in chat mode, runs the expected SQL on the agent's warehouse, and compares results in memory with `compare_results`:
- Rows and columns are compared regardless of order, and numbers are compared after rounding to 4 decimals, so column order alone never fails a question.
- Extra columns (all expected columns present, plus more) are `PASS_EXTRA_COLUMNS`: counted as a pass, listed separately.
- Both results empty is a pass. Expected SQL that errors, is missing or returns no rows is `EXCLUDED` from the denominator and listed.
- Genie's 5,000-row result cap is a pass when the expected result is larger; otherwise above 5,000 rows only counts are compared.
- It waits about 15 seconds between questions. Test conversations appear in the Monitoring tab under the user's name.

**Stability (`stability_check`).** Genie isn't deterministic. Re-run failing questions twice; a question that passes on any re-run is `FLAKY`, otherwise `STABLE_FAIL`. Report both counts.

**Reporting rules.**
- Accuracy = passed / scored questions, with the size of one question in points.
- Show failing **question text** and reason codes only. Never print values.
- Use the same question set for before and after. Report excluded questions.

**Regression rules (`regression_check`).** On the same question IDs: `BLOCK` if any critical question regressed; `ALERT` if accuracy dropped 5 points or more, or 3 or more questions regressed. Questions marked `FLAKY` are excluded from these rules.

**Guardrail tests (`guardrail_test`).** Chat mode, because native benchmarks need expected SQL. Pass = no SQL generated. Result rows are never fetched; only the agent's reply text is shown.
