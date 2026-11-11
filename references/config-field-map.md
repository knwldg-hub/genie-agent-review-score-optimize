# Config and API field map

Tested on databricks-sdk 0.122.0. String fields in the config are usually lists of strings; join them (`_txt`).

| Item | Location |
|---|---|
| Tables | `data_sources.tables[]`: `identifier`, `column_configs[]` |
| Metric views | `data_sources.metric_views[]`, and can also appear under `data_sources.tables[]`; detect with Unity Catalog `table_type = METRIC_VIEW` |
| Column config | `column_name`, `enable_entity_matching`, `enable_format_assistance`, `synonyms[]`, `exclude` (true = hidden from Genie), optional `description` |
| Text instructions | `instructions.text_instructions[]`: `id`, `content[]` |
| Example SQL | `instructions.example_question_sqls[]`: `id`, `question[]`, `sql[]`, optional `usage_guidance` |
| SQL functions, join specs | `instructions.sql_functions[]`, `instructions.join_specs[]` |
| SQL snippets | `instructions.sql_snippets.measures[]`, `.filters[]`, `.expressions[]` |
| Sample questions | `config.sample_questions[]`: `id`, `question[]` |
| Benchmarks | `benchmarks.questions[]`: `id`, `question[]`, `answer[]` with `{format: "SQL", content[]}` |
| Copy ID | `w.genie.create_space(...)` returns `.space_id` |
| Chat answer SQL | `start_conversation_and_wait(...)` → `attachments[i].query.query` |
| Chat answer rows | `get_message_attachment_query_result(space_id, conversation_id, message_id, attachment_id).statement_response.result.data_array` |
| Benchmark run | `genie_create_eval_run(space_id, benchmark_question_ids=None)` → `.eval_run_id`; status from `genie_get_eval_run` (`DONE` when finished) |
| Benchmark results | `genie_list_eval_results` → `result_id`, `benchmark_question_id`, `question`; `genie_get_eval_result_details` → `assessment` (`GOOD`/`BAD`), `assessment_reasons[]` |
