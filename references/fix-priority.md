# Fix priority and default impact/effort

The order follows the Databricks best practices: metadata first, text instructions last.

| Order | Fix type | Typical impact | Typical effort |
|---|---|---|---|
| 1 | Unity Catalog metadata: table and column descriptions, correct data types, hide noisy columns | High | Low–Medium |
| 2 | Entity matching, format assistance, column synonyms | High | Low |
| 3 | SQL expressions (measures, filters, dimensions), join specs, metric views | High | Medium |
| 4 | Fix or add example SQL and trusted SQL functions | High | Low–Medium |
| 5 | Shorten text instructions (keep only behavior, scope and tone) | Medium | Medium |
| 6 | Expand benchmarks to 15–20, including edge cases (`references/benchmark-drafting.md`). Move to first and set impact to High when fewer than 10 have expected SQL | Medium | Medium |
| 7 | Description, sample questions, redirects to other agents | Medium | Low |
| 8 | Remove [STAGE] from the name and certify | Low (routing boost) | Low |

Text instructions (row 5) come after rows 1–4 for a reason: in Genie, structured config (metadata, synonyms, snippets, example SQL) is applied more reliably than prose, and long instructions push out higher-value context. Adding a text rule to fix one benchmark usually trades one failure for another.
