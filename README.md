# genie-agent-review-score-optimize (Genie Code skill, v0.5 pilot)

A Genie Code skill that reviews, scores and optimizes an existing Genie Agent (formerly Genie Space). The skill file is [`SKILL.md`](./SKILL.md).

In the Databricks workspace, Genie Code only loads a skill from `.assistant/skills/<skill-name>/SKILL.md` in your user folder. The install steps below save the file under that name.

No extra packages are needed. IQ scoring is built into the skill (ported from the Genie Workbench IQ Scan), and benchmarks run through the native Genie benchmark API, with a chat-mode fallback.

## Install

`SKILL.md` is at the root of this repo, so cloning the repo as a Git folder inside a skills folder installs the skill directly. Nothing to copy, and updating is just **Pull**. ([How Genie Code skills work](https://docs.databricks.com/aws/en/genie-code/skills/))

### Option 1: for the whole workspace (admin, recommended for teams)

1. As a workspace admin, go to **Workspace → Workspace** and open the `.assistant/skills` folder (create `.assistant` and `skills` if they don't exist; folders starting with a dot may be hidden).
2. In `skills`, choose **Create → Git folder**, paste `https://github.com/knwldg-hub/genie-agent-review-score-optimize.git`, and click **Create**. Keep the default folder name.
3. Make sure users can read the folder.

Everyone in the workspace gets the skill in their next Genie Code chat. To update everyone: open the Git folder and click **Pull**.

### Option 2: for yourself

1. In Genie Code, click **Settings → Open skills folder**. This opens `/Users/<you>/.assistant/skills/`, creating it if needed.
2. In that folder, choose **Create → Git folder**, paste `https://github.com/knwldg-hub/genie-agent-review-score-optimize.git`, and click **Create**. Keep the default folder name.
3. Start a new Genie Code chat.

To update: open the Git folder and click **Pull**, then start a new chat.

If a version of the skill already exists at `.assistant/skills/genie-agent-review-score-optimize`, delete that folder first.

### Option 3: copy the file (if you can't create a Git folder)

Use this only if Git folders aren't available to you. Updates mean copying again.

**In the workspace UI:**

1. Go to **Workspace → Users → your folder**.
2. Create the folder `.assistant/skills/genie-agent-review-score-optimize`. Folders that start with a dot may be hidden; turn on hidden files if you don't see them.
3. In that folder, choose **Create → File**, name it `SKILL.md`, and paste the content of [`SKILL.md`](./SKILL.md).

**With the Databricks CLI**, from a local clone of this repo:

```bash
databricks workspace mkdirs "/Workspace/Users/YOUR_EMAIL/.assistant/skills/genie-agent-review-score-optimize"

databricks workspace import \
  "/Workspace/Users/YOUR_EMAIL/.assistant/skills/genie-agent-review-score-optimize/SKILL.md" \
  --file ./SKILL.md \
  --format AUTO --overwrite
```

Replace `YOUR_EMAIL` with your workspace user name.

**Check the install:** in a new Genie Code chat, ask "Is the genie-agent-review-score-optimize skill installed, and which version?" It should say Version 0.5. If you just pulled an update and see old behavior, start a new chat or hard-refresh the browser tab.

## Run a review

Open Genie Code from a notebook, ideally a new empty one, and say one of:

- "Use the genie-agent-review-score-optimize skill to review Genie Agent <agent-id>."
- "Review, score and optimize my Genie Agent <agent-id>."

Other useful prompts during a session:

| You want to... | Say |
|---|---|
| Fix a recommendation | "Start with R3" (the skill applies one change at a time and re-runs the affected benchmarks) |
| See the current scorecard | "Show my scorecard" |
| Score again after a deploy | "It's deployed, re-score" |
| Undo changes | "Restore the backup" |
| Add more benchmark questions | "Draft more benchmarks" |
| Finish and get the report | "Finish the review" |

## Prerequisites

- Genie Code enabled in the workspace
- CAN EDIT on the Genie Agent being reviewed. CAN VIEW is enough for the quick scan and a review without changes.
- CAN USE on the agent's SQL warehouse
- A recent Databricks SDK with the Genie benchmark API (`genie_create_eval_run`). Tested on serverless with databricks-sdk 0.122.0. On older SDKs the skill falls back to a chat-mode harness.
- To apply Unity Catalog description fixes: MODIFY on those tables. Otherwise the skill writes a SQL script for the table owner.

## Testing tip

For the first runs, pick **Mode C (test on a copy)** at step 2. The skill clones the agent and makes all changes on the copy, so the original stays untouched. Benchmark runs show up in the copy's Benchmarks tab history, and the skill offers to trash the copy at the end.

## Feedback

Open an issue in this repo with the agent shape (number of tables and metric views), your cloud, and what happened.

## Version history

The skill file keeps only the current instructions; changes are listed here.

### v0.5

Adopts the useful parts of the field-kit skills `automated_iq_scorer` and `benchmark_eval_harness`. The headline score stays the Genie Workbench IQ score, so it matches what customers see in Workbench.

- Benchmark quality in the quick scan: expected SQL using `CURRENT_DATE()` (results drift daily) and question-type coverage (time, ranking, ratio, period comparison, segmentation, trailing window)
- Optional guardrail tests: 5 risky prompts (personal data, delete, prompt injection, out of scope, ambiguous); pass = no SQL generated
- Critical questions: a change that makes one fail is blocked
- Regression rules: alert on a drop of 5+ points or 3+ regressed questions; flaky questions ignored
- Readiness checklist: accuracy ≥ 85%, critical questions pass, guardrails pass
- Optional MLflow history of scores (no data)
- Field-kit checks (metric views, hidden columns, stricter description thresholds) shown as advice only
- Benchmark drafting with 2–4 phrasings per concept and dates anchored on the latest loaded date

Not adopted: the composite trust score (inputs undefined) and rules that fail valid agents (mandatory metric view, 5+ hidden columns, "GRAIN" in every comment, `GROUP BY ALL`).

### v0.4

- Quick scan first: a read-only scan in seconds (no warehouse compute) showing what looks good, what could be better and what's missing
- Benchmarks are offered after the quick scan, not run automatically; you can stop with a quick-scan report (works with CAN VIEW)
- Benchmark runs show live progress

### v0.3

- No package dependency: built-in IQ scoring using Genie Workbench rules
- Native benchmark runs through the API (same engine as the Benchmarks tab), with Genie reason codes
- Stability re-runs to separate real failures from flaky ones
- One change at a time, with targeted benchmark re-runs
- Guardrails against tuning to the benchmark and against reusing old scores
- Failure reason to fix mapping, and a tested helper library

### v0.2

- Benchmark sufficiency check, benchmark drafting with approval, low-confidence labels, like-for-like before/after comparison