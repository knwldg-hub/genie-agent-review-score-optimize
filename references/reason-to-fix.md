# Failure reason → likely fix

Reason codes come from native benchmark runs (Genie eval taxonomy) or from `compare_results`. Use them to pick the fix level, highest first. Confirm business definitions with the user where marked †.

| Reason code | What it usually means | Try first | Then |
|---|---|---|---|
| `RESULT_MISSING_COLUMNS`, `LLM_JUDGE_INCOMPLETE_OR_PARTIAL_OUTPUT` | Answer leaves out a requested measure or dimension | Measure or dimension in the metric view, or a SQL measure snippet; example SQL that returns the full shape | Check whether the question wording really asks for it |
| `RESULT_EXTRA_COLUMNS` | Extra context columns | Often harmless; look for instructions that force "context" columns (D13) | Remove the forcing instruction |
| `RESULT_MISSING_ROWS`, `RESULT_EXTRA_ROWS`, `LLM_JUDGE_MISSING_OR_INCORRECT_FILTER` | Wrong or missing filter, often a date window | Filter snippet with the confirmed definition †; entity matching on filter columns | Example SQL showing the filter |
| `EMPTY_RESULT` | Filter value doesn't match stored values | Entity matching or format assistance on that column; synonyms | Column description with valid values described in words |
| `SINGLE_CELL_DIFFERENCE`, `LLM_JUDGE_INCORRECT_METRIC_CALCULATION` | Metric computed differently | Measure snippet or metric-view measure (ratio of sums, distinct counts) † | Example SQL |
| `LLM_JUDGE_MISSING_OR_INCORRECT_AGGREGATION` | Wrong grain or grouping | Example SQL at the right grain; dimension expressions | Column descriptions for grain |
| `LLM_JUDGE_INCORRECT_TABLE_OR_FIELD_USAGE` | Picked the wrong column or table | Column descriptions and synonyms; hide trap columns (D9) | Narrow the data sources |
| `LLM_JUDGE_MISSING_OR_INCORRECT_JOIN` | Join missing or wrong | Join spec | Example SQL with the join |
| `LLM_JUDGE_INCORRECT_FUNCTION_USAGE` | Wrong SQL function or parameters | Trusted SQL function | Example SQL |
| `LLM_JUDGE_FORMATTING_ERROR` | Usually ordering or presentation | Check whether order matters to the question; often not a real error | Example SQL with explicit ORDER BY |
| `LLM_JUDGE_INSTRUCTION_COMPLIANCE_OR_MISSING_BUSINESS_LOGIC` | A rule in text isn't applied | Move the rule into a SQL snippet or example SQL † | Shorten competing instructions |
| `LLM_JUDGE_MISINTERPRETATION_OF_USER_REQUEST` | Question is ambiguous or misread | Sample questions and synonyms for the business term † | Benchmark correction, only with user confirmation |
| `EMPTY_GOOD_SQL` | The benchmark's expected SQL returns no rows | Benchmark correction with the user (excluded from accuracy until fixed) | |
| `COLUMN_TYPE_DIFFERENCE` | Same values, different type | Fix the data type (D7) or CAST in the measure | |
| `FLAKY` (from stability check) | Agent answers differently run to run | Remove ambiguity: definition †, competing example SQL, conflicting instructions | Benchmark wording, with user confirmation |
