---
name: genie-agent-review-score-optimize
description: Review, score and optimize an existing Genie Agent (formerly Genie Space). Use when the user asks to review, audit, score, IQ-score, health-check, benchmark, tune, improve the accuracy of, or optimize a Genie Agent or Genie Space. Runs a guided loop - back up config, check benchmark coverage (and draft new benchmark questions with approval when there are too few), baseline IQ score (built in, no package install) plus deep review plus native Genie benchmark runs with stability checks, prioritized recommendations, implement approved fixes one at a time with targeted re-runs, then re-score with a like-for-like before/after comparison.
license: See NOTICE.md in this skill folder
compatibility: Databricks Genie Code, working in a notebook on serverless or a cluster. Needs a databricks-sdk with the Genie benchmark API (tested on 0.122.0).
metadata:
  version: "0.6"
---

# Genie Agent Review, Score & Optimize

Version 0.6 (pilot). Works with Genie Agents on any Databricks workspace. Field-built pilot, not a supported Databricks product; see the repo's NOTICE.md.

You are guiding the user through an end-to-end review of one Genie Agent. Work in a notebook as the current user. Go one step at a time. At every ⏸ checkpoint, stop and wait for the user's answer.

Version history is in the README.

## Files in this skill

Paths are relative to this skill folder. Read each file only at the step named, not up front.

| File | Read it when |
|---|---|
| `scripts/genie_review_helpers.py` | Step 0: import it (don't paste it into context unless the import fails) |
| `references/iq-rubric.md` | Step 4b, or when explaining an IQ check or tier |
| `references/deep-review.md` | Step 1.5 (config-only checks) and Step 4c |
| `references/benchmark-evaluation.md` | Steps 3.5 and 4a, and whenever you interpret a benchmark run |
| `references/benchmark-drafting.md` | Drafting benchmark questions (Step 3.5, or a benchmark recommendation) |
| `references/reason-to-fix.md` | Step 6, mapping failure reason codes to fixes |
| `references/fix-priority.md` | Step 6 ranking, and before any text-instruction change (golden rule 10) |
| `references/config-field-map.md` | Before building a config change (Step 7), or when a helper fails |
| `assets/report-templates.md` | Steps 1.5, 4, 6, 9 and 10, before showing a report or scorecard |

## Golden rules (always follow)

1. **No change without approval.** Show the exact change (diff or SQL) and get an explicit "yes" for each change before applying it.
2. **Back up first, verify after.** Save the agent's full config before the first change. After every change, re-fetch the config and check nothing else was lost. If anything was removed unexpectedly, restore the backup immediately and tell the user.
3. **No data values in output.** The data may contain PII. Never print result rows, sample values or example values. Show only counts, column names, data types, SQL text and pass/fail.
4. **Show code before running it**, and run it in notebook cells.
5. **Don't guess.** If an API call fails or a field is missing, explain what happened and offer a manual alternative.
6. **Use GCP documentation links** (docs.databricks.com/gcp/...).
7. **Preserve IDs.** When editing the config, keep existing item IDs and all fields you aren't changing. New items get a new 32-character lowercase hex ID (`new_id()`). Copy the structure of existing entries of the same type. Use the field map in `references/config-field-map.md`.
8. **Don't tune to the test.** Never change a benchmark, or write an instruction, just so the agent's output matches a benchmark's expected SQL. When a failure depends on a business definition (for example what "last year" or "active player" means), ask the user for the definition first. A benchmark may be edited only when the user confirms it is wrong; label that a "benchmark correction" and report it separately from accuracy changes.
9. **Never carry a score forward.** Every number in a scorecard must come from a run in this session or a run the user reports. If a score was not re-run, show "not re-run", not the old value.
10. **Text instructions are the last resort.** Before proposing a text-instruction change, say why a higher fix in `references/fix-priority.md` (Unity Catalog metadata, synonyms, entity matching, SQL snippets, example SQL) can't solve it.
11. **Use the helper library.** Import `scripts/genie_review_helpers.py` once in Step 0 and use its functions. Don't rediscover API field names by trial and error.

## Gotchas

- **Metric views can sit under `data_sources.tables`.** `summarize` detects them from Unity Catalog (`table_type = METRIC_VIEW`). Never call a metric view a table.
- **`exclude: true` on a column config means hidden from Genie**, not deleted. Count it as a hidden column.
- **IQ check 5 (join specs) fails on single-source agents** because that's the Workbench rule. Report it as scored, but treat it as N/A for recommendations.
- **`CURRENT_DATE()` in expected SQL drifts.** The expected answer changes daily, so a benchmark can start failing with no agent change. Anchor windows on the latest loaded date.
- **Column order never decides pass or fail**, and extra columns count as a pass (`PASS_EXTRA_COLUMNS`). Neither is a reason to edit a benchmark.
- **One question is noise.** A move of one question or less is never a real improvement or regression; Genie isn't deterministic, so always check stability before crediting a fix.
- **Restarting Python clears the helpers.** After `%pip install` and `restartPython()`, re-run the Step 0 setup cell.
- **Import the helpers; don't `exec()` file content.** Genie Code may refuse to run code read from a file with `exec`/`open`. The Step 0 import is the supported path. If the skill folder is updated mid-session, re-run the setup cell (it reloads the module).
- **Benchmark runs and guardrail chats are visible.** They show in the agent's Benchmarks tab history and Monitoring tab under the user's name. Tell the user before running them.
- **Trash only on an explicit yes.** `w.genie.trash_space` is called only in Step 10, only for a Mode C copy, and only after the user says yes.

## Workflow

Step 0 Setup → 1 Pick agent → 1.5 Quick scan (read-only; can stop here) → 2 Apply mode → 3 Backup → 3.5 Benchmark sufficiency check → 4 Baseline score (+ optional guardrail tests) → 5 Other benchmark runs → 6 Recommendations → 7 Draft one change → 8 Apply and targeted re-run → (repeat 7–8) → 9 Full re-score → 10 Next or finish

### Step 0: Setup

Make sure a notebook is attached to serverless compute or a cluster. No package installs are needed unless the SDK check below says so.

Run this setup cell. It imports the helper library from this skill folder (per-user or workspace install) and defines `w`, `ME`, `REVIEW_ROOT` and the helper functions used in every step. Tell the user it's a read-only setup cell.

```python
import os, sys, importlib
from databricks.sdk import WorkspaceClient
_me = WorkspaceClient().current_user.me().user_name
_cands = [f"/Workspace/Users/{_me}/.assistant/skills/genie-agent-review-score-optimize/scripts",
          "/Workspace/.assistant/skills/genie-agent-review-score-optimize/scripts"]
SKILL_SCRIPTS = next((p for p in _cands if os.path.exists(f"{p}/genie_review_helpers.py")), None)
assert SKILL_SCRIPTS, "helper file not found in either skills folder"
if SKILL_SCRIPTS not in sys.path:
    sys.path.insert(0, SKILL_SCRIPTS)
import genie_review_helpers as grh
importlib.reload(grh)
from genie_review_helpers import *
print("helpers", grh.HELPERS_VERSION, "from", SKILL_SCRIPTS)
```

The printed version should be 0.6 (this skill's `metadata.version`); if it differs, tell the user the skill folder is out of date. If the file isn't found or the import fails, read `scripts/genie_review_helpers.py` with the skill file reader, paste it into one cell exactly as written, and run it. If a helper fails, show the error and use `references/config-field-map.md` to work out the field, rather than guessing.

Then run `sdk = sdk_check()`. If `sdk["eval_api"]` is False, tell the user the installed databricks-sdk is too old for the native benchmark API (tested on 0.122.0), and that without it benchmarks fall back to the less reliable chat-mode harness. Offer to run, in a new cell:

```python
%pip install -U databricks-sdk
dbutils.library.restartPython()
```

The restart clears all variables, so afterwards re-run the setup cell and `sdk_check()`. If the user declines, or the install fails (for example no access to PyPI), continue and label every benchmark result "chat-mode harness (approximation)".

### Step 1: Pick the agent ⏸

Ask: *"Which Genie Agent should I review? Paste the agent ID (the 32-character ID in the agent's URL), or tell me its name and I'll look it up."*

If they give a name, list the agents they can access (`w.genie.list_spaces()`) and match on title.

```python
SPACE_ID = "<agent-id>"
space, cfg = get_cfg(SPACE_ID)
uc = uc_info(cfg)          # Unity Catalog metadata only
summary = summarize(cfg, uc)
```

Show the title, agent ID and warehouse ID, and ask: *"Is this the right agent?"* Metric views can appear under `data_sources.tables`; `summarize` detects them from Unity Catalog, so never call a metric view a table.

### Step 1.5: Quick scan ⏸

Tell the user first: read-only, config and Unity Catalog metadata only, no warehouse compute, no changes.

```python
qs = quick_scan(space, cfg, uc)
bq = benchmark_quality(cfg)        # relative dates in expected SQL, question-type coverage (heuristic)
adv = advisory_checks(cfg, uc)     # advice only; never changes the IQ score
```

Add the config-only deep-review checks (D1, D2, D5, D8, D9, D10, D12, D13 in `references/deep-review.md`) to the same groups; the rest wait for the full score. Show the result in plain language, with an evidence count per line, using the **Quick scan** template in `assets/report-templates.md`. Put each `adv` item on the advisory line, and list relative-date benchmarks under ⚠️ (their expected answers drift daily). Then ask:

*"Next step? (a) run benchmarks for a full score: n questions, about a few minutes, shows in the Benchmarks tab history. It uses warehouse compute: one run of all n questions, plus 2 re-runs of each failing question, so up to about 3× one run if many fail; (b) see recommendations from the config alone; (c) stop here and save this quick-scan report."*

Default: recommend (a) unless the user only has CAN VIEW (then (c)).

- **(a)**: continue to step 2. Recommend running on a copy (Mode C) if the user doesn't own the agent.
- **(b)**: go to step 6 using the quick-scan evidence only. Label it "config-only, accuracy not measured". Before the first change, do steps 2 and 3 (apply mode and backup) and offer a baseline benchmark run, so the change can be measured.
- **(c)**: save `{REVIEW_ROOT}/{SPACE_ID}/quick_scan_{date}.md` and finish. This works with CAN VIEW only.

An earlier `last_full_run` is context only, shown with its date (golden rule 9).

### Step 2: Choose how changes are applied ⏸

Ask: *"How should I apply approved changes?"*

- **Mode A: edit the agent directly.** Best for agents managed in the UI.
- **Mode B: produce PR-ready config.** Best for agents deployed through CI/CD (DABs, a pipeline, or a deployment service principal). A UI edit would be overwritten by the next deploy. I'll write the updated config file and a change summary for your repo, then re-score after you deploy.
- **Mode C: test on a copy.** I clone the agent (same tables, instructions and benchmarks) as "[REVIEW COPY] <title>" and change only the copy. Good for trying the skill out.

If the agent is owned by a service principal, or the user says it's deployed by CI/CD, recommend Mode B. Otherwise the default is Mode C for a first review of an agent. The user decides. Record the choice. The user can switch modes for any single recommendation.

For Mode C, create the copy:

```python
copy = w.genie.create_space(
    warehouse_id=space.warehouse_id,
    serialized_space=space.serialized_space,
    title=f"[REVIEW COPY] {space.title}",
    description=space.description,
)
SOURCE_ID, SPACE_ID = SPACE_ID, copy.space_id
space, cfg = get_cfg(SPACE_ID)
```

Continue with the copy's ID. Tell the user the copy's ID, and that you'll offer to trash it at the end.

### Step 3: Backup

```python
TS = datetime.datetime.utcnow().strftime("%Y%m%dT%H%M%SZ")
DIR = f"{REVIEW_ROOT}/{SPACE_ID}"
os.makedirs(DIR, exist_ok=True)
with open(f"{DIR}/backup_{TS}.json", "w") as f:
    json.dump({"title": space.title, "description": space.description,
               "warehouse_id": space.warehouse_id, "serialized_space": cfg}, f, indent=2)
```

Tell the user where the backup is saved.

### Step 3.5: Benchmark sufficiency check ⏸

Benchmarks are how every other fix is measured, so check them before scoring. Use `summary["benchmarks"]` and `summary["benchmarks_with_sql"]` (only questions with expected SQL can be auto-checked; see `references/benchmark-evaluation.md`).

Classify coverage:

| Benchmarks with expected SQL | Status | What to tell the user |
|---|---|---|
| 0 | ❌ None | "There are no benchmarks to run, so accuracy can't be measured and IQ checks 10, 11 and 12 will fail." |
| 1–9 | ⚠️ Low confidence | "With only n questions, each one moves accuracy by about 100/n points. The accuracy score is indicative only." |
| 10–14 | ⚠️ Thin | "Enough to score, but edge cases are probably not covered." |
| 15 or more | ✅ OK | Continue. Still check edge-case coverage (D11) in step 4c. |

With fewer than 10, label every accuracy figure "low confidence (n questions)"; with none, report "not measurable", never 0%.

If the status is ❌ or ⚠️, ask: *"Do you want me to draft additional benchmark questions now, before the baseline score? (recommended / later / no)"*

- **Recommended (default):** follow `references/benchmark-drafting.md`, add the approved questions (Mode A/C) or write them to the proposed config (Mode B), then continue to step 4 on the expanded set. Record original and added counts.
- **Later / No:** continue to step 4 with the label above. The benchmark recommendation still appears in step 6 (ranked first if later).

Record the decision in the scorecard.

**Benchmark date anchoring.** If `bq["relative_date_sql"]` isn't empty, list those questions and offer a benchmark correction (golden rule 8) that anchors the window on the latest loaded date, as in `references/benchmark-drafting.md`.

**Critical questions** ⏸

Show the benchmark questions as a numbered list and ask: *"Which of these are business-critical, meaning they must always be answered correctly? (numbers, 'none', or 'skip')"* Save the answer:

```python
CRITICAL = [ids of the chosen questions]
json.dump(CRITICAL, open(f"{DIR}/critical_questions.json", "w"))
```

If a later change makes a critical question fail stably, the change is blocked (step 8). If the user picks none, say that regressions are judged on the overall rules only.

### Step 4: Baseline score

Benchmarks first (the IQ score needs the accuracy); reuse the quick-scan results and add only what it couldn't measure. Show one scorecard at the end.

**4a. Benchmarks (read `references/benchmark-evaluation.md`)**

Tell the user the question count, that it shows in the Benchmarks tab history, and the cost: one run of all questions plus 2 stability re-runs of each failing question, so up to about 3× one run. Each approved change later adds a targeted re-run, and step 9 adds one full run.

```python
base = run_benchmarks(SPACE_ID)                 # native run, same engine as the Benchmarks tab
failing = [r["benchmark_question_id"] for r in base["results"] if r["assessment"] != "GOOD"]
stab = stability_check(SPACE_ID, failing, reps=2) if failing else {}
```

If the eval API isn't available (older SDK, 404, permission), say so and use `chat_benchmarks(SPACE_ID, cfg, space.warehouse_id)`, labelled "chat-mode harness (approximation)".

Report *"x% (passed/n). One question = 100/n points."* For each failing question show its text, reason codes and `STABLE_FAIL` / `FLAKY`. Only `STABLE_FAIL` drives fix recommendations; `FLAKY` gets one ambiguity recommendation (wording, missing definition, competing example SQL). A change of one question or less is never a real improvement or regression.

**4b. IQ score (built in; rubric in `references/iq-rubric.md`)**

```python
iq = iq_score(cfg, uc, best_accuracy=base["accuracy"])
```

Show score, tier and every check with detail and severity, exactly as the Workbench rules give it (for example check 5 fails on a single-source agent; mark that "N/A for recommendations").

**4c. Deep review (`references/deep-review.md`)**

Give D1–D13 a status (✅ / ⚠️ / ❌) with evidence meeting the criteria in that file; evidence may quote config text or SQL. Use `sql_in_prose_lines(text)` for D1 and `explain_ok(space.warehouse_id, sql)` for D3. D14 and D15 go under **Manual checks**, unscored.

**4d. Guardrail tests (optional)** ⏸

Ask: *"Should I test how the agent handles 5 risky prompts (personal data, delete requests, prompt injection, out of scope, ambiguous)? About 1–2 minutes; the conversations appear in the Monitoring tab."* Default: recommend yes. Offer to adapt the prompts to the domain first (for example name person-level fields in the PII prompt).

```python
guard = guardrail_test(SPACE_ID)           # or guardrail_test(SPACE_ID, adapted_prompts)
```

Pass = no SQL generated. An ambiguous prompt that gets SQL is `REVIEW`: show the reply and let the user judge. Any other prompt that gets SQL fails and becomes a recommendation (out-of-scope statement, hiding sensitive columns, or column masks).

**Readiness**

```python
ready = readiness(base, CRITICAL, guard)    # guard stays None if 4d was skipped
```

Show its 3 items as ✅ / ❌ with detail.

**Scorecard**

Show it using the **Baseline scorecard** template in `assets/report-templates.md`.

Save the scorecard to `{DIR}/scorecard_baseline_{TS}.json`.

**History (optional).** Ask once: *"Should I log scores to MLflow so you can track this agent over time?"* Default: no. If yes, call `log_review_mlflow(SPACE_ID, "baseline", iq, base, guard)` now, and with `"after_R<n>"` and `"final"` later.

### Step 5: Other benchmark runs (optional) ⏸

The API run in 4a uses the platform's default benchmark mode. If the user also wants a run in another mode (for example Agent mode in the UI), ask: *"Do you want to run benchmarks in another mode from the Benchmarks tab? I can read the result directly once it finishes."*

If yes:
1. Open the Genie Agent (the copy, in Mode C) and go to the **Benchmarks** tab.
2. Choose the mode, click **Run all**, and wait for it to finish.
3. Tell me "done" and which mode you used.

Read it with `recent_runs(SPACE_ID)` and `run_results(SPACE_ID, eval_run_id)` instead of asking for pasted numbers, and record the mode the user states. If no, mark it "none".

### Step 6: Recommendations ⏸

Build one recommendation per failed or warned item, and one per group of `STABLE_FAIL` questions that share a reason code. Also include guardrail prompts that failed, benchmarks with relative dates, and question types below target (as a benchmark recommendation, `references/benchmark-drafting.md`). Use `references/reason-to-fix.md` to map reason codes to fixes. Combine related items. Don't make recommendations from IQ check 5 when it is N/A, or from manual checks. Each recommendation has:

- **ID** (R1, R2, …) and a short title
- **Why**: the evidence from step 4
- **Fix**: what will change
- **Impact** (High / Medium / Low) and **Effort** (Low / Medium / High). Use `references/fix-priority.md` defaults, adjusted to the evidence.
- **Moves**: which IQ checks, deep-review checks and benchmark questions it should improve
- **Change type**: agent config, Unity Catalog metadata, or both
- **Fix level**: its row in `references/fix-priority.md`. If it's a text-instruction change, say why a higher level can't fix it (golden rule 10).
- **Needs a definition?**: yes if it depends on a business definition the user must confirm (golden rule 8)

Sort by impact, then by lowest effort, then by the best-practice order in `references/fix-priority.md`. Exception: if benchmark coverage from step 3.5 is ❌ or ⚠️ Low confidence (fewer than 10 with expected SQL), put the benchmark recommendation first and set its impact to High, because other fixes can't be measured reliably without it. Show the top 10 using the **Recommendations** template in `assets/report-templates.md`.

Below it, list **Manual checks** (D14, D15, and anything the API can't see) as a checklist for the user, not as recommendations.

Ask: *"Which one should we start with? Pick one (for example R2), or ask me to explain any."* Default: R1. If the user picks several, explain that one at a time shows which change helped; if they still want them together, warn that the effects can't be separated, then follow their choice.

### Step 7: Draft the change ⏸

Draft one change at a time. For the chosen recommendation:

- **Business definitions first** (golden rule 8): for date windows, "active", "churned", currency, fiscal vs calendar, offer the common options and let the user choose. Default windows anchor on the latest loaded data (D6) unless the user says otherwise.
- **Agent config changes**: build the updated config in memory. Show a before/after diff of only the affected section.
- **Text instruction changes** (last resort): keep them short and general, not tied to one benchmark's wording. Keep total instructions under about 2,000 characters where possible; when adding a line, propose removing or moving an equivalent one.
- **Benchmark corrections** (golden rule 8): show the diff and record it separately. Column order or extra columns are never a reason to edit a benchmark.
- **Unity Catalog changes**: write the SQL, for example `COMMENT ON TABLE ...` and `ALTER TABLE ... ALTER COLUMN ... COMMENT '...'`. Descriptions must explain meaning, units and valid ranges in words. They must not contain real data values.
- **New or changed SQL** (example SQL, SQL expressions, functions): validate with `EXPLAIN` before showing it.
- **New benchmark questions**: follow `references/benchmark-drafting.md`.

Ask: *"Apply this change? (yes / edit / skip)"*

### Step 8: Apply

**Mode A or C:**

```python
before = cfg
w.genie.update_space(SPACE_ID, serialized_space=json.dumps(new_cfg))
space, cfg = get_cfg(SPACE_ID)
check = verify_update(before, cfg, expected_counts_delta={})   # e.g. {"synonym_cols": 2}
```

- **Verify the agent config:** `check["ok"]` must be true: counts moved only as intended and no existing IDs were lost. If not, restore the backup and report it.
- **Targeted re-run:** re-run only the questions this change should affect plus the questions that failed before, twice: `run_benchmarks(SPACE_ID, qids)` then `stability_check`. Report each question's flip (fail→pass, pass→fail, flaky). Then compare with `rc = regression_check(prev, new, CRITICAL, stab)` on the same question IDs:
  - `BLOCK` (a critical question regressed): roll back this change, unless the user overrides with a reason; record the override in the report.
  - `ALERT` (accuracy down 5 points or more, or 3 or more questions regressed): show the regressed questions and offer to roll back.
  - Flaky questions are listed separately, not counted.

  Then go back to step 7 for the next recommendation, or to step 9.
- **Unity Catalog SQL:** run it only if the user approved it and has `MODIFY` on the tables. Otherwise save it to `{DIR}/proposed/uc_metadata_{TS}.sql` for the table owner.

**Mode B:**

Write these files and tell the user the paths:
- `{DIR}/proposed/serialized_space_{TS}.json`: the full updated config
- `{DIR}/proposed/uc_metadata_{TS}.sql`: if there are Unity Catalog changes
- `{DIR}/proposed/CHANGES_{TS}.md`: what changed, why, and which checks it should improve

Say: *"Commit these to your repo and deploy through your normal process. Tell me 'deployed' when it's live and I'll re-score."* ⏸

### Step 9: Re-score

Re-fetch the config and rerun step 4 in full (native benchmarks with stability check, IQ score, deep review). Ask whether they want to rerun other-mode benchmarks using step 5. Anything not re-run is shown as "not re-run" (golden rule 9).

Credit only stable changes larger than one question (step 4a). Run `regression_check(base, final, CRITICAL, stab)` and `readiness(final, CRITICAL, guard)`, and re-run guardrail tests if they ran at baseline (instruction changes can affect refusals). Compare like for like: if benchmarks were added after the baseline, report accuracy on the original set (the true before/after) and on the expanded set (the new baseline). A change caused only by adding or removing questions is never an improvement.

Show the result using the **Before/after** template in `assets/report-templates.md`.

Save the scorecard to `{DIR}/scorecard_after_{TS}.json`.

### Step 10: Next or finish ⏸

Ask: *"Want to continue with the next recommendation (R?), pick another, or finish?"*

When they finish, save the report described under **Final review report** in `assets/report-templates.md`.

Share the path. For Mode C, remind them to apply the proven changes to the real agent (Mode A or B), then ask: *"Should I move the review copy to the trash?"* Only on yes, run `w.genie.trash_space(SPACE_ID)`.

## Rollback

```python
b = json.load(open("<backup path>"))
w.genie.update_space(SPACE_ID, serialized_space=json.dumps(b["serialized_space"]))
```

For Mode B, revert the commit in their repo.

## References (GCP)

- Curate an effective Genie Agent: https://docs.databricks.com/gcp/en/genie-agents/best-practices
- Tune Genie Agent quality: https://docs.databricks.com/gcp/en/genie-agents/tune-quality
- Benchmarks: https://docs.databricks.com/gcp/en/genie-agents/benchmarks
- Genie Agents API: https://docs.databricks.com/gcp/en/genie-agents/conversation-api
- Metric views: https://docs.databricks.com/gcp/en/uc-semantics/metric-views
