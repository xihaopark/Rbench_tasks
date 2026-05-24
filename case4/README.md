# case4

31 Codex CLI GPT-5.5 clinical benchmark probe cases using the same task set as case2/case3.

Each case uses the same presentation template as the previous GitHub case batches. The generated code shown here is the evaluator candidate artifact (`generated_solution.R`) from the Codex run, not a regenerated sample.

Summary:

```text
Total cases: 31
Agent: Codex CLI
Model: gpt-5.5
Strict pass@1: 19 / 31 = 0.6129
Status counts: {'PASS': 19, 'FAIL': 8, 'NO_OUTPUT': 4}
Invalid/internal package API pattern: 0 / 31
```

Fairness note:

- Normal exported APIs from installed packages are treated as supported.
- Missing functions, non-exported helper calls via `pkg::`, and `getFromNamespace()` access to internal helpers are not treated as environment-missing failures.

## Case4 Index

| Case | Package | Topic | Status | Pattern | Document |
| ---: | --- | --- | --- | --- | --- |
| 001 | `aNCA` | `PKNCA_impute_method_start_c1` | `PASS` | `` | [case4_001_aNCA_PKNCA_impute_method_start_c1.md](./case4_001_aNCA_PKNCA_impute_method_start_c1.md) |
| 003 | `aNCA` | `add_exclusion_reasons` | `PASS` | `` | [case4_003_aNCA_add_exclusion_reasons.md](./case4_003_aNCA_add_exclusion_reasons.md) |
| 004 | `aNCA` | `add_impute_method` | `FAIL` | `` | [case4_004_aNCA_add_impute_method.md](./case4_004_aNCA_add_impute_method.md) |
| 016 | `aNCA` | `detect_study_types` | `FAIL` | `` | [case4_016_aNCA_detect_study_types.md](./case4_016_aNCA_detect_study_types.md) |
| 034 | `aNCA` | `interval_add_impute` | `PASS` | `` | [case4_034_aNCA_interval_add_impute.md](./case4_034_aNCA_interval_add_impute.md) |
| 036 | `aNCA` | `keep_blq_timepoints` | `PASS` | `` | [case4_036_aNCA_keep_blq_timepoints.md](./case4_036_aNCA_keep_blq_timepoints.md) |
| 045 | `aNCA` | `remove_impute_method` | `PASS` | `` | [case4_045_aNCA_remove_impute_method.md](./case4_045_aNCA_remove_impute_method.md) |
| 050 | `aNCA` | `translate_terms` | `PASS` | `` | [case4_050_aNCA_translate_terms.md](./case4_050_aNCA_translate_terms.md) |
| 063 | `admiral` | `compute_egfr` | `FAIL` | `` | [case4_063_admiral_compute_egfr.md](./case4_063_admiral_compute_egfr.md) |
| 064 | `admiral` | `compute_framingham` | `PASS` | `` | [case4_064_admiral_compute_framingham.md](./case4_064_admiral_compute_framingham.md) |
| 065 | `admiral` | `compute_map` | `PASS` | `` | [case4_065_admiral_compute_map.md](./case4_065_admiral_compute_map.md) |
| 071 | `admiral` | `compute_tmf` | `NO_OUTPUT` | `` | [case4_071_admiral_compute_tmf.md](./case4_071_admiral_compute_tmf.md) |
| 086 | `admiral` | `convert_time_units` | `NO_OUTPUT` | `` | [case4_086_admiral_convert_time_units.md](./case4_086_admiral_convert_time_units.md) |
| 087 | `admiral` | `convert_treatment_patterns` | `FAIL` | `` | [case4_087_admiral_convert_treatment_patterns.md](./case4_087_admiral_convert_treatment_patterns.md) |
| 088 | `admiral` | `convert_xxtpt_to_hours` | `PASS` | `` | [case4_088_admiral_convert_xxtpt_to_hours.md](./case4_088_admiral_convert_xxtpt_to_hours.md) |
| 094 | `admiral` | `derive_locf_records` | `FAIL` | `` | [case4_094_admiral_derive_locf_records.md](./case4_094_admiral_derive_locf_records.md) |
| 097 | `admiral` | `derive_param_map` | `NO_OUTPUT` | `` | [case4_097_admiral_derive_param_map.md](./case4_097_admiral_derive_param_map.md) |
| 098 | `admiral` | `derive_param_qtc` | `FAIL` | `` | [case4_098_admiral_derive_param_qtc.md](./case4_098_admiral_derive_param_qtc.md) |
| 100 | `admiral` | `derive_param_tte` | `NO_OUTPUT` | `` | [case4_100_admiral_derive_param_tte.md](./case4_100_admiral_derive_param_tte.md) |
| 101 | `admiral` | `derive_var_atoxgr_dir` | `PASS` | `` | [case4_101_admiral_derive_var_atoxgr_dir.md](./case4_101_admiral_derive_var_atoxgr_dir.md) |
| 106 | `admiral` | `derive_var_trtemfl` | `PASS` | `` | [case4_106_admiral_derive_var_trtemfl.md](./case4_106_admiral_derive_var_trtemfl.md) |
| 111 | `admiral` | `derive_vars_crit_flag` | `PASS` | `` | [case4_111_admiral_derive_vars_crit_flag.md](./case4_111_admiral_derive_vars_crit_flag.md) |
| 118 | `admiral` | `derive_vars_extreme_event` | `PASS` | `` | [case4_118_admiral_derive_vars_extreme_event.md](./case4_118_admiral_derive_vars_extreme_event.md) |
| 123 | `admiral` | `derive_vars_query` | `PASS` | `` | [case4_123_admiral_derive_vars_query.md](./case4_123_admiral_derive_vars_query.md) |
| 142 | `admiral` | `get_flagged_records` | `FAIL` | `` | [case4_142_admiral_get_flagged_records.md](./case4_142_admiral_get_flagged_records.md) |
| 147 | `admiral` | `get_imputation_targets` | `PASS` | `` | [case4_147_admiral_get_imputation_targets.md](./case4_147_admiral_get_imputation_targets.md) |
| 148 | `admiral` | `get_joined_sub_data` | `PASS` | `` | [case4_148_admiral_get_joined_sub_data.md](./case4_148_admiral_get_joined_sub_data.md) |
| 160 | `admiral` | `propagate_na_values` | `PASS` | `` | [case4_160_admiral_propagate_na_values.md](./case4_160_admiral_propagate_na_values.md) |
| 164 | `admiral` | `slice_derivation` | `FAIL` | `` | [case4_164_admiral_slice_derivation.md](./case4_164_admiral_slice_derivation.md) |
| 179 | `ggsurvfit` | `scale_ggsurvfit` | `PASS` | `` | [case4_179_ggsurvfit_scale_ggsurvfit.md](./case4_179_ggsurvfit_scale_ggsurvfit.md) |
| 180 | `gridify` | `get_layouts` | `PASS` | `` | [case4_180_gridify_get_layouts.md](./case4_180_gridify_get_layouts.md) |
