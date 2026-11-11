# IQ score rubric (12 checks)

Built in (`iq_score` in `scripts/genie_review_helpers.py`). Adapted from the Genie Workbench IQ Scan (`genie_space_optimizer/iq_scan/scoring.py` in https://github.com/databricks-solutions/databricks-genie-workbench, DB license; changed: rewritten as a standalone function over the agent config and Unity Catalog metadata), so no package install is needed and results match what the customer sees in Workbench. Descriptions count from the agent config or from Unity Catalog comments (Workbench fills blanks from Unity Catalog the same way).

| # | Check | Pass when | Warning (still passes) |
|---|---|---|---|
| 1 | Data sources exist | At least 1 table or metric view | |
| 2 | Table descriptions | ≥ 80% of tables described (metric-view-only agents pass) | < 100% |
| 3 | Column descriptions | ≥ 50% of configured columns described (metric-view-only agents pass) | < 80%; no synonyms anywhere |
| 4 | Text instructions | Total length > 50 characters | > 2,000 characters; SQL found in prose (`sql_in_prose_lines`) |
| 5 | Join specifications | At least 1 join spec. Workbench fails this even for a single source; report it, but treat as N/A for recommendations when there is 1 source | |
| 6 | Data source count | Between 1 and 12 | |
| 7 | Example SQLs | 8 or more | 8–9 (10–15 is the sweet spot); more than half lack `usage_guidance` |
| 8 | SQL snippets | At least 1 SQL function, measure, filter or expression | No filters, or no measures |
| 9 | Entity / format matching | Enabled on at least 1 column | More than 100 entity-matching columns (limit 120 per agent); tables with row filters or column masks silently disable entity matching |
| 10 | Benchmarks | 10 or more benchmark questions | |
| 11 | Benchmark run completed | A completed benchmark run covering the benchmark set (this session, or read with `recent_runs`) | |
| 12 | Accuracy | Accuracy of that run ≥ 85% | |

If an earlier review used the v0.1/v0.2 rubric, scores aren't comparable; say so when comparing.

**Tiers:**
- **Not Ready**: any of checks 1–10 fails
- **Ready to Optimize**: checks 1–10 pass, but 11 or 12 fails
- **Trusted**: all 12 pass
