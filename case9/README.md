# case9

31 Claude Code (claude-sonnet-4-6) clinical benchmark-light cases. Base prompt in a lightweight benchmark-style Claude Code invocation, directly comparable to the Codex base run in case4.

Each case uses the same presentation template as the previous GitHub case batches. The generated code shown here is the evaluator candidate artifact (`generated_solution.R`) from the Claude Code run, not a regenerated sample.

Summary:

```text
Total cases: 31
Agent: Claude Code
Model: claude-code/claude-sonnet-4-6
Prompt setting: benchmark-light base prompt; Claude Code CLI with lightweight task prompt and restricted tool use
Run directory: internal/coding_agent_outputs/case2_31/claude-code/claude-sonnet-4-6/case9_benchmark_light
Strict pass@1: 14 / 31 = 0.4516
Status counts: {'FAIL': 10, 'PASS': 14, 'NO_OUTPUT': 7}
Invalid/internal package API pattern: 1 / 31
```

Fairness note:

- Normal exported APIs from installed packages are treated as supported.
- Missing functions, non-exported helper calls via `pkg::`, and `getFromNamespace()` access to internal helpers are not treated as environment-missing failures.

## Case9 Index

| Case | Package | Topic | Status | Pattern | Document |
| ---: | --- | --- | --- | --- | --- |
| 001 | `aNCA` | `PKNCA_impute_method_start_c1` | `FAIL` | `` | [case9_001_aNCA_PKNCA_impute_method_start_c1.md](./case9_001_aNCA_PKNCA_impute_method_start_c1.md) |
| 003 | `aNCA` | `add_exclusion_reasons` | `PASS` | `` | [case9_003_aNCA_add_exclusion_reasons.md](./case9_003_aNCA_add_exclusion_reasons.md) |
| 004 | `aNCA` | `add_impute_method` | `FAIL` | `` | [case9_004_aNCA_add_impute_method.md](./case9_004_aNCA_add_impute_method.md) |
| 016 | `aNCA` | `detect_study_types` | `FAIL` | `` | [case9_016_aNCA_detect_study_types.md](./case9_016_aNCA_detect_study_types.md) |
| 034 | `aNCA` | `interval_add_impute` | `FAIL` | `` | [case9_034_aNCA_interval_add_impute.md](./case9_034_aNCA_interval_add_impute.md) |
| 036 | `aNCA` | `keep_blq_timepoints` | `PASS` | `` | [case9_036_aNCA_keep_blq_timepoints.md](./case9_036_aNCA_keep_blq_timepoints.md) |
| 045 | `aNCA` | `remove_impute_method` | `PASS` | `` | [case9_045_aNCA_remove_impute_method.md](./case9_045_aNCA_remove_impute_method.md) |
| 050 | `aNCA` | `translate_terms` | `NO_OUTPUT` | `` | [case9_050_aNCA_translate_terms.md](./case9_050_aNCA_translate_terms.md) |
| 063 | `admiral` | `compute_egfr` | `FAIL` | `` | [case9_063_admiral_compute_egfr.md](./case9_063_admiral_compute_egfr.md) |
| 064 | `admiral` | `compute_framingham` | `PASS` | `` | [case9_064_admiral_compute_framingham.md](./case9_064_admiral_compute_framingham.md) |
| 065 | `admiral` | `compute_map` | `PASS` | `` | [case9_065_admiral_compute_map.md](./case9_065_admiral_compute_map.md) |
| 071 | `admiral` | `compute_tmf` | `NO_OUTPUT` | `` | [case9_071_admiral_compute_tmf.md](./case9_071_admiral_compute_tmf.md) |
| 086 | `admiral` | `convert_time_units` | `NO_OUTPUT` | `` | [case9_086_admiral_convert_time_units.md](./case9_086_admiral_convert_time_units.md) |
| 087 | `admiral` | `convert_treatment_patterns` | `NO_OUTPUT` | `` | [case9_087_admiral_convert_treatment_patterns.md](./case9_087_admiral_convert_treatment_patterns.md) |
| 088 | `admiral` | `convert_xxtpt_to_hours` | `NO_OUTPUT` | `` | [case9_088_admiral_convert_xxtpt_to_hours.md](./case9_088_admiral_convert_xxtpt_to_hours.md) |
| 094 | `admiral` | `derive_locf_records` | `FAIL` | `` | [case9_094_admiral_derive_locf_records.md](./case9_094_admiral_derive_locf_records.md) |
| 097 | `admiral` | `derive_param_map` | `PASS` | `` | [case9_097_admiral_derive_param_map.md](./case9_097_admiral_derive_param_map.md) |
| 098 | `admiral` | `derive_param_qtc` | `FAIL` | `` | [case9_098_admiral_derive_param_qtc.md](./case9_098_admiral_derive_param_qtc.md) |
| 100 | `admiral` | `derive_param_tte` | `FAIL` | `` | [case9_100_admiral_derive_param_tte.md](./case9_100_admiral_derive_param_tte.md) |
| 101 | `admiral` | `derive_var_atoxgr_dir` | `PASS` | `` | [case9_101_admiral_derive_var_atoxgr_dir.md](./case9_101_admiral_derive_var_atoxgr_dir.md) |
| 106 | `admiral` | `derive_var_trtemfl` | `PASS` | `` | [case9_106_admiral_derive_var_trtemfl.md](./case9_106_admiral_derive_var_trtemfl.md) |
| 111 | `admiral` | `derive_vars_crit_flag` | `PASS` | `` | [case9_111_admiral_derive_vars_crit_flag.md](./case9_111_admiral_derive_vars_crit_flag.md) |
| 118 | `admiral` | `derive_vars_extreme_event` | `PASS` | `` | [case9_118_admiral_derive_vars_extreme_event.md](./case9_118_admiral_derive_vars_extreme_event.md) |
| 123 | `admiral` | `derive_vars_query` | `NO_OUTPUT` | `` | [case9_123_admiral_derive_vars_query.md](./case9_123_admiral_derive_vars_query.md) |
| 142 | `admiral` | `get_flagged_records` | `FAIL` | `` | [case9_142_admiral_get_flagged_records.md](./case9_142_admiral_get_flagged_records.md) |
| 147 | `admiral` | `get_imputation_targets` | `NO_OUTPUT` | `invalid_or_internal_package_api` | [case9_147_admiral_get_imputation_targets.md](./case9_147_admiral_get_imputation_targets.md) |
| 148 | `admiral` | `get_joined_sub_data` | `PASS` | `` | [case9_148_admiral_get_joined_sub_data.md](./case9_148_admiral_get_joined_sub_data.md) |
| 160 | `admiral` | `propagate_na_values` | `PASS` | `` | [case9_160_admiral_propagate_na_values.md](./case9_160_admiral_propagate_na_values.md) |
| 164 | `admiral` | `slice_derivation` | `FAIL` | `` | [case9_164_admiral_slice_derivation.md](./case9_164_admiral_slice_derivation.md) |
| 179 | `ggsurvfit` | `scale_ggsurvfit` | `PASS` | `` | [case9_179_ggsurvfit_scale_ggsurvfit.md](./case9_179_ggsurvfit_scale_ggsurvfit.md) |
| 180 | `gridify` | `get_layouts` | `PASS` | `` | [case9_180_gridify_get_layouts.md](./case9_180_gridify_get_layouts.md) |
