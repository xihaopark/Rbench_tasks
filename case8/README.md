# case8

31 Claude Code (claude-sonnet-4-6) clinical benchmark cases. Base prompt, no web access. Same 31-task set as case4/case6/case7 (Codex), directly comparable to case4.

Each case uses the same presentation template as the previous GitHub case batches. The generated code shown here is the evaluator candidate artifact (`generated_solution.R`) from the Claude Code run, not a regenerated sample.

Summary:

```text
Total cases: 31
Agent: Claude Code
Model: claude-code/claude-sonnet-4-6
Prompt setting: base prompt, no WebFetch/WebSearch (equivalent to case4)
Run directory: internal/coding_agent_outputs/case2_31/claude-code/claude-sonnet-4-6/case8_base
Strict pass@1: 15 / 31 = 0.4839
Status counts: {'PASS': 15, 'NO_OUTPUT': 6, 'FAIL': 10}
Invalid/internal package API pattern: 1 / 31
```

Fairness note:

- Normal exported APIs from installed packages are treated as supported.
- Missing functions, non-exported helper calls via `pkg::`, and `getFromNamespace()` access to internal helpers are not treated as environment-missing failures.

## Case8 Index

| Case | Package | Topic | Status | Pattern | Document |
| ---: | --- | --- | --- | --- | --- |
| 001 | `aNCA` | `PKNCA_impute_method_start_c1` | `PASS` | `` | [case8_001_aNCA_PKNCA_impute_method_start_c1.md](./case8_001_aNCA_PKNCA_impute_method_start_c1.md) |
| 003 | `aNCA` | `add_exclusion_reasons` | `PASS` | `` | [case8_003_aNCA_add_exclusion_reasons.md](./case8_003_aNCA_add_exclusion_reasons.md) |
| 004 | `aNCA` | `add_impute_method` | `NO_OUTPUT` | `` | [case8_004_aNCA_add_impute_method.md](./case8_004_aNCA_add_impute_method.md) |
| 016 | `aNCA` | `detect_study_types` | `FAIL` | `` | [case8_016_aNCA_detect_study_types.md](./case8_016_aNCA_detect_study_types.md) |
| 034 | `aNCA` | `interval_add_impute` | `PASS` | `` | [case8_034_aNCA_interval_add_impute.md](./case8_034_aNCA_interval_add_impute.md) |
| 036 | `aNCA` | `keep_blq_timepoints` | `NO_OUTPUT` | `invalid_or_internal_package_api` | [case8_036_aNCA_keep_blq_timepoints.md](./case8_036_aNCA_keep_blq_timepoints.md) |
| 045 | `aNCA` | `remove_impute_method` | `PASS` | `` | [case8_045_aNCA_remove_impute_method.md](./case8_045_aNCA_remove_impute_method.md) |
| 050 | `aNCA` | `translate_terms` | `FAIL` | `` | [case8_050_aNCA_translate_terms.md](./case8_050_aNCA_translate_terms.md) |
| 063 | `admiral` | `compute_egfr` | `FAIL` | `` | [case8_063_admiral_compute_egfr.md](./case8_063_admiral_compute_egfr.md) |
| 064 | `admiral` | `compute_framingham` | `PASS` | `` | [case8_064_admiral_compute_framingham.md](./case8_064_admiral_compute_framingham.md) |
| 065 | `admiral` | `compute_map` | `PASS` | `` | [case8_065_admiral_compute_map.md](./case8_065_admiral_compute_map.md) |
| 071 | `admiral` | `compute_tmf` | `NO_OUTPUT` | `` | [case8_071_admiral_compute_tmf.md](./case8_071_admiral_compute_tmf.md) |
| 086 | `admiral` | `convert_time_units` | `NO_OUTPUT` | `` | [case8_086_admiral_convert_time_units.md](./case8_086_admiral_convert_time_units.md) |
| 087 | `admiral` | `convert_treatment_patterns` | `FAIL` | `` | [case8_087_admiral_convert_treatment_patterns.md](./case8_087_admiral_convert_treatment_patterns.md) |
| 088 | `admiral` | `convert_xxtpt_to_hours` | `NO_OUTPUT` | `` | [case8_088_admiral_convert_xxtpt_to_hours.md](./case8_088_admiral_convert_xxtpt_to_hours.md) |
| 094 | `admiral` | `derive_locf_records` | `FAIL` | `` | [case8_094_admiral_derive_locf_records.md](./case8_094_admiral_derive_locf_records.md) |
| 097 | `admiral` | `derive_param_map` | `PASS` | `` | [case8_097_admiral_derive_param_map.md](./case8_097_admiral_derive_param_map.md) |
| 098 | `admiral` | `derive_param_qtc` | `FAIL` | `` | [case8_098_admiral_derive_param_qtc.md](./case8_098_admiral_derive_param_qtc.md) |
| 100 | `admiral` | `derive_param_tte` | `FAIL` | `` | [case8_100_admiral_derive_param_tte.md](./case8_100_admiral_derive_param_tte.md) |
| 101 | `admiral` | `derive_var_atoxgr_dir` | `PASS` | `` | [case8_101_admiral_derive_var_atoxgr_dir.md](./case8_101_admiral_derive_var_atoxgr_dir.md) |
| 106 | `admiral` | `derive_var_trtemfl` | `FAIL` | `` | [case8_106_admiral_derive_var_trtemfl.md](./case8_106_admiral_derive_var_trtemfl.md) |
| 111 | `admiral` | `derive_vars_crit_flag` | `PASS` | `` | [case8_111_admiral_derive_vars_crit_flag.md](./case8_111_admiral_derive_vars_crit_flag.md) |
| 118 | `admiral` | `derive_vars_extreme_event` | `PASS` | `` | [case8_118_admiral_derive_vars_extreme_event.md](./case8_118_admiral_derive_vars_extreme_event.md) |
| 123 | `admiral` | `derive_vars_query` | `PASS` | `` | [case8_123_admiral_derive_vars_query.md](./case8_123_admiral_derive_vars_query.md) |
| 142 | `admiral` | `get_flagged_records` | `FAIL` | `` | [case8_142_admiral_get_flagged_records.md](./case8_142_admiral_get_flagged_records.md) |
| 147 | `admiral` | `get_imputation_targets` | `NO_OUTPUT` | `` | [case8_147_admiral_get_imputation_targets.md](./case8_147_admiral_get_imputation_targets.md) |
| 148 | `admiral` | `get_joined_sub_data` | `PASS` | `` | [case8_148_admiral_get_joined_sub_data.md](./case8_148_admiral_get_joined_sub_data.md) |
| 160 | `admiral` | `propagate_na_values` | `PASS` | `` | [case8_160_admiral_propagate_na_values.md](./case8_160_admiral_propagate_na_values.md) |
| 164 | `admiral` | `slice_derivation` | `FAIL` | `` | [case8_164_admiral_slice_derivation.md](./case8_164_admiral_slice_derivation.md) |
| 179 | `ggsurvfit` | `scale_ggsurvfit` | `PASS` | `` | [case8_179_ggsurvfit_scale_ggsurvfit.md](./case8_179_ggsurvfit_scale_ggsurvfit.md) |
| 180 | `gridify` | `get_layouts` | `PASS` | `` | [case8_180_gridify_get_layouts.md](./case8_180_gridify_get_layouts.md) |
