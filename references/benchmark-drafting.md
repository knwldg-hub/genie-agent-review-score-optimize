# Drafting benchmark questions

Goal: a benchmark set of 15–20 questions that reflects how real users ask, and covers the edge cases the agent is supposed to handle.

**1. Gather sources (config and metadata only, never data values)**

- Sample questions in the agent description and the agent's suggested questions
- Rules and edge cases stated in the text instructions (default date windows, exclusions, definitions, title or name mappings)
- Join specs, SQL expressions (measures, filters, dimensions) and trusted SQL functions
- Table and column descriptions in Unity Catalog and in the config
- Failing questions from chat-mode or Agent-mode runs, if any
- Real user questions from the Monitoring tab. Ask the user first: *"May I use recent questions from the Monitoring tab as benchmark candidates? I'll only use the question text."* If the user declines or the API isn't available, ask them to paste a few questions instead.

**2. Draft candidates**

Draft enough candidates to reach 15–20 in total (existing plus new). Aim for this mix:

| Category | Share | Example of what it tests |
|---|---|---|
| Core questions | about 40% | The main metrics and breakdowns the agent is for |
| Edge cases | about 25% | Default date windows, exclusions, NULL handling, distinct counts, rates computed from sums |
| Phrasing variants | about 15% | 2–4 wordings of the same concept (synonyms, abbreviations, business terms), sharing one expected SQL |
| Multi-table | about 10% | Questions that need a join spec or metric view |
| Out of scope | about 10% | Questions the agent should decline or redirect. Don't add them as benchmarks (they have no expected SQL); test them with `guardrail_test` instead |

Also cover the question types that `benchmark_quality` reports below target (time aggregation ≥ 20%, ranking ≥ 10%, ratios ≥ 10%, period comparison ≥ 15%, segmentation ≥ 15%, trailing windows ≥ 5%). Mark which drafts are critical candidates.

For each candidate write: the question as a user would ask it, the category, the question type, the source it came from, and the expected SQL.

**3. Avoid overfitting**

- Don't copy an existing example SQL question word for word. Rephrase it, or change a filter or grain, so the benchmark tests generalization.
- Don't duplicate an existing benchmark.
- Don't put hard-coded dates or literal data values in the question or SQL.
- Anchor date windows in expected SQL on the latest loaded date, for example `(SELECT MAX(<date column>) FROM <source>)`, not `CURRENT_DATE()`, so the expected answer doesn't drift. If the user's confirmed definition is calendar-relative, keep it and note that results will change daily.

**4. Validate the expected SQL without showing data**

- Run `EXPLAIN` on each expected SQL. Drop or fix any that fail.
- Optionally run it once and record only the row count and column count. Flag queries that return 0 rows (probably a wrong filter) or more than 5,000 rows (too broad for a reliable comparison).
- Check the SQL against D4 (window partitions, sums vs averages, DISTINCT on user counts, date windows).

**5. Get approval** ⏸

Show the candidates as a table (no data values):

| # | Question | Category | Source | SQL valid | Rows |
|---|---|---|---|---|---|

Ask: *"Which of these should I add? You can say 'all', list numbers (for example 1–8, 11), or ask me to edit any of them."* Let the user edit wording and expected SQL. Only approved questions are added.

**6. Add them**

- **Mode A or C:** add the approved questions to the benchmarks section of the config, following golden rules 2 and 7 (backup exists, new 32-character hex IDs, copy the structure of existing benchmark entries; if there are none, use the built-in genie-space-authoring knowledge for the schema). Apply with `update_space`, then re-fetch and confirm that the benchmark count went up by exactly the approved number and nothing else changed.
- **Mode B:** add them to `{DIR}/proposed/serialized_space_{TS}.json` and list them in `CHANGES_{TS}.md`.

Tell the user how many were added and the new total. Suggest they review the out-of-scope questions in the UI Benchmarks tab, since those can't be auto-checked.
