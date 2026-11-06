## Support

This is a field-built pilot, not a supported Databricks product. Databricks does not offer official support for it. For any issue, open an issue in this GitHub repository; it will be looked at on a best-effort basis.

## Attribution

The IQ scoring in `SKILL.md` (Appendix A and the `iq_score` and `sql_in_prose_lines` helpers in Appendix H) is adapted from the Genie Workbench IQ Scan, `genie_space_optimizer/iq_scan/scoring.py` in https://github.com/databricks-solutions/databricks-genie-workbench, licensed under the DB license (see LICENSE.md).

Changes: the scoring logic was rewritten as standalone Python functions that read a Genie Agent's config and Unity Catalog metadata through the Databricks SDK, so no package install is needed. The 12 checks and their thresholds follow the Workbench rules.

Genie Workbench notice: "Databricks does not offer official support for Databricks Solutions and its repository. For any issue with this assets or the demos installed, please open an issue using github and the team will have a look on a best effort basis."
