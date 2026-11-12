# Deep-review checks

These come from real Genie Agent reviews.

| # | Check | Flag when |
|---|---|---|
| D1 | Logic in text | `sql_in_prose_lines` finds SQL in the instructions, or the text defines metric formulas, filters, joins or name mappings that could be SQL snippets, join specs or synonyms |
| D2 | Instruction size | ⚠️ over 2,000 characters; ❌ over about 8,000 characters or 15 sections; or persona and tone text mixed with data rules |
| D3 | Example SQL and snippet validity | Any example SQL fails `EXPLAIN`, or `snippets_ok` reports a snippet that fails on every data source. Read the error first: a snippet written with a table alias (for example `t.col`) can fail on its own but still work in Genie; report that as ⚠️, not ❌. In a quick scan (no warehouse), D3 is ❓ Not measured yet. |
| D4 | Example SQL logic | Window functions partitioned on the wrong column; averaging rates or averages instead of dividing sums; comments describe a filter the SQL doesn't apply; a date window that doesn't match the stated default (for example `BETWEEN DATE_SUB(CURRENT_DATE(), 8) AND DATE_SUB(CURRENT_DATE(), 1)` is 8 days); `COUNT(*)` used as a user count; a missing `DISTINCT` on user metrics |
| D5 | Stale facts | Instructions contain hard-coded dates, row counts or measured values that will go out of date |
| D6 | Date windows | Defaults are relative to `CURRENT_DATE()` instead of the latest loaded date in each source |
| D7 | Data types | Numeric measures stored as STRING (they need CAST in SQL) |
| D8 | Metadata contradictions | Instructions say column comments are wrong or not to be trusted. Fix them in Unity Catalog instead |
| D9 | Trap columns | Columns the instructions warn against (misleading flags, always-NULL, deprecated) are still visible |
| D10 | Scope and overlap | Sources span several domains or grains; or the description overlaps other agents. Pass only when both are checked: the scope is one domain or grain, and the user either confirmed there are no related agents or gave their IDs and the descriptions don't overlap. Adding out-of-scope text alone doesn't pass D10 |
| D11 | Benchmark coverage | Fewer than 15 questions; or edge cases documented in the instructions have no benchmark |
| D12 | Description for routing | Pass needs all three: sample questions, a statement of what's out of scope, and no technical detail that doesn't help routing |
| D13 | Response verbosity | Instructions force a long plan or many metadata fields on every answer |
| D14 | Feedback signal (manual check) | Few or no thumbs up or down, or review comments. Ask the user to check the Monitoring tab. Not scored |
| D15 | Production readiness (manual check) | Name contains [STAGE] or [DEV], or the prod agent isn't certified. Not scored |
