# case3

31 high-value GPT-5.5 clinical benchmark probe cases using the current clean simple prompt format.

Each case uses the same presentation template as the first 197-case batch.

Summary:

```text
Total cases: 31
Model: openai/gpt-5.5
Probe strict pass@1: 13 / 31 = 0.4194
Package API hallucination: 4 / 31 = 0.1290
```

## Case3 Index

| Case | Package | Topic | Status | Pattern | Document |
| ---: | --- | --- | --- | --- | --- |
| 001 | `aNCA` | `PKNCA_impute_method_start_c1` | `PASS` |  | [case3_001_aNCA_PKNCA_impute_method_start_c1.md](./case3_001_aNCA_PKNCA_impute_method_start_c1.md) |
| 003 | `aNCA` | `add_exclusion_reasons` | `PASS` |  | [case3_003_aNCA_add_exclusion_reasons.md](./case3_003_aNCA_add_exclusion_reasons.md) |
| 004 | `aNCA` | `add_impute_method` | `FAIL` |  | [case3_004_aNCA_add_impute_method.md](./case3_004_aNCA_add_impute_method.md) |
| 016 | `aNCA` | `detect_study_types` | `FAIL` |  | [case3_016_aNCA_detect_study_types.md](./case3_016_aNCA_detect_study_types.md) |
| 034 | `aNCA` | `interval_add_impute` | `FAIL` |  | [case3_034_aNCA_interval_add_impute.md](./case3_034_aNCA_interval_add_impute.md) |
| 036 | `aNCA` | `keep_blq_timepoints` | `PASS` |  | [case3_036_aNCA_keep_blq_timepoints.md](./case3_036_aNCA_keep_blq_timepoints.md) |
| 045 | `aNCA` | `remove_impute_method` | `NO_OUTPUT` |  | [case3_045_aNCA_remove_impute_method.md](./case3_045_aNCA_remove_impute_method.md) |
| 050 | `aNCA` | `translate_terms` | `PASS` |  | [case3_050_aNCA_translate_terms.md](./case3_050_aNCA_translate_terms.md) |
| 063 | `admiral` | `compute_egfr` | `FAIL` |  | [case3_063_admiral_compute_egfr.md](./case3_063_admiral_compute_egfr.md) |
| 064 | `admiral` | `compute_framingham` | `PASS` |  | [case3_064_admiral_compute_framingham.md](./case3_064_admiral_compute_framingham.md) |
| 065 | `admiral` | `compute_map` | `PASS` |  | [case3_065_admiral_compute_map.md](./case3_065_admiral_compute_map.md) |
| 071 | `admiral` | `compute_tmf` | `NO_OUTPUT` | `package_api_hallucination` | [case3_071_admiral_compute_tmf.md](./case3_071_admiral_compute_tmf.md) |
| 086 | `admiral` | `convert_time_units` | `FAIL` |  | [case3_086_admiral_convert_time_units.md](./case3_086_admiral_convert_time_units.md) |
| 087 | `admiral` | `convert_treatment_patterns` | `FAIL` |  | [case3_087_admiral_convert_treatment_patterns.md](./case3_087_admiral_convert_treatment_patterns.md) |
| 088 | `admiral` | `convert_xxtpt_to_hours` | `PASS` |  | [case3_088_admiral_convert_xxtpt_to_hours.md](./case3_088_admiral_convert_xxtpt_to_hours.md) |
| 094 | `admiral` | `derive_locf_records` | `FAIL` |  | [case3_094_admiral_derive_locf_records.md](./case3_094_admiral_derive_locf_records.md) |
| 097 | `admiral` | `derive_param_map` | `NO_OUTPUT` | `package_api_hallucination` | [case3_097_admiral_derive_param_map.md](./case3_097_admiral_derive_param_map.md) |
| 098 | `admiral` | `derive_param_qtc` | `NO_OUTPUT` | `package_api_hallucination` | [case3_098_admiral_derive_param_qtc.md](./case3_098_admiral_derive_param_qtc.md) |
| 100 | `admiral` | `derive_param_tte` | `NO_OUTPUT` |  | [case3_100_admiral_derive_param_tte.md](./case3_100_admiral_derive_param_tte.md) |
| 101 | `admiral` | `derive_var_atoxgr_dir` | `PASS` |  | [case3_101_admiral_derive_var_atoxgr_dir.md](./case3_101_admiral_derive_var_atoxgr_dir.md) |
| 106 | `admiral` | `derive_var_trtemfl` | `FAIL` |  | [case3_106_admiral_derive_var_trtemfl.md](./case3_106_admiral_derive_var_trtemfl.md) |
| 111 | `admiral` | `derive_vars_crit_flag` | `FAIL` |  | [case3_111_admiral_derive_vars_crit_flag.md](./case3_111_admiral_derive_vars_crit_flag.md) |
| 118 | `admiral` | `derive_vars_extreme_event` | `PASS` |  | [case3_118_admiral_derive_vars_extreme_event.md](./case3_118_admiral_derive_vars_extreme_event.md) |
| 123 | `admiral` | `derive_vars_query` | `PASS` |  | [case3_123_admiral_derive_vars_query.md](./case3_123_admiral_derive_vars_query.md) |
| 142 | `admiral` | `get_flagged_records` | `FAIL` |  | [case3_142_admiral_get_flagged_records.md](./case3_142_admiral_get_flagged_records.md) |
| 147 | `admiral` | `get_imputation_targets` | `NO_OUTPUT` | `package_api_hallucination` | [case3_147_admiral_get_imputation_targets.md](./case3_147_admiral_get_imputation_targets.md) |
| 148 | `admiral` | `get_joined_sub_data` | `PASS` |  | [case3_148_admiral_get_joined_sub_data.md](./case3_148_admiral_get_joined_sub_data.md) |
| 160 | `admiral` | `propagate_na_values` | `NO_OUTPUT` |  | [case3_160_admiral_propagate_na_values.md](./case3_160_admiral_propagate_na_values.md) |
| 164 | `admiral` | `slice_derivation` | `FAIL` |  | [case3_164_admiral_slice_derivation.md](./case3_164_admiral_slice_derivation.md) |
| 179 | `ggsurvfit` | `scale_ggsurvfit` | `PASS` |  | [case3_179_ggsurvfit_scale_ggsurvfit.md](./case3_179_ggsurvfit_scale_ggsurvfit.md) |
| 180 | `gridify` | `get_layouts` | `PASS` |  | [case3_180_gridify_get_layouts.md](./case3_180_gridify_get_layouts.md) |
