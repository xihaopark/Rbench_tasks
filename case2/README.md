# case2

Thirty high-value GPT-5.1 clinical benchmark probe cases using the current clean simple prompt format.

Each case uses the same presentation template as the first 197-case batch.

Summary:

```text
Total cases: 30
Model: openai/gpt-5.1
Probe strict pass@1: 4 / 30 = 0.1333
```

## Case2 Index

| Case | Package | Topic | Status | Document |
| ---: | --- | --- | --- | --- |
| 01 | `admiral` | `compute_egfr` | `NO_OUTPUT` | [case2_01_admiral_compute_egfr.md](./case2_01_admiral_compute_egfr.md) |
| 02 | `admiral` | `compute_framingham` | `NO_OUTPUT` | [case2_02_admiral_compute_framingham.md](./case2_02_admiral_compute_framingham.md) |
| 03 | `admiral` | `compute_map` | `PASS` | [case2_03_admiral_compute_map.md](./case2_03_admiral_compute_map.md) |
| 04 | `admiral` | `compute_tmf` | `NO_OUTPUT` | [case2_04_admiral_compute_tmf.md](./case2_04_admiral_compute_tmf.md) |
| 05 | `admiral` | `convert_treatment_patterns` | `NO_OUTPUT` | [case2_05_admiral_convert_treatment_patterns.md](./case2_05_admiral_convert_treatment_patterns.md) |
| 06 | `admiral` | `convert_xxtpt_to_hours` | `FAIL` | [case2_06_admiral_convert_xxtpt_to_hours.md](./case2_06_admiral_convert_xxtpt_to_hours.md) |
| 07 | `admiral` | `get_imputation_targets` | `NO_OUTPUT` | [case2_07_admiral_get_imputation_targets.md](./case2_07_admiral_get_imputation_targets.md) |
| 08 | `admiral` | `get_flagged_records` | `NO_OUTPUT` | [case2_08_admiral_get_flagged_records.md](./case2_08_admiral_get_flagged_records.md) |
| 09 | `admiral` | `get_joined_sub_data` | `PASS` | [case2_09_admiral_get_joined_sub_data.md](./case2_09_admiral_get_joined_sub_data.md) |
| 10 | `admiral` | `slice_derivation` | `NO_OUTPUT` | [case2_10_admiral_slice_derivation.md](./case2_10_admiral_slice_derivation.md) |
| 11 | `admiral` | `derive_param_map` | `NO_OUTPUT` | [case2_11_admiral_derive_param_map.md](./case2_11_admiral_derive_param_map.md) |
| 12 | `admiral` | `derive_param_qtc` | `NO_OUTPUT` | [case2_12_admiral_derive_param_qtc.md](./case2_12_admiral_derive_param_qtc.md) |
| 13 | `admiral` | `derive_param_tte` | `NO_OUTPUT` | [case2_13_admiral_derive_param_tte.md](./case2_13_admiral_derive_param_tte.md) |
| 14 | `admiral` | `derive_var_atoxgr_dir` | `NO_OUTPUT` | [case2_14_admiral_derive_var_atoxgr_dir.md](./case2_14_admiral_derive_var_atoxgr_dir.md) |
| 15 | `admiral` | `derive_var_trtemfl` | `NO_OUTPUT` | [case2_15_admiral_derive_var_trtemfl.md](./case2_15_admiral_derive_var_trtemfl.md) |
| 16 | `admiral` | `derive_vars_crit_flag` | `NO_OUTPUT` | [case2_16_admiral_derive_vars_crit_flag.md](./case2_16_admiral_derive_vars_crit_flag.md) |
| 17 | `admiral` | `derive_vars_extreme_event` | `NO_OUTPUT` | [case2_17_admiral_derive_vars_extreme_event.md](./case2_17_admiral_derive_vars_extreme_event.md) |
| 18 | `admiral` | `derive_vars_query` | `PASS` | [case2_18_admiral_derive_vars_query.md](./case2_18_admiral_derive_vars_query.md) |
| 19 | `admiral` | `derive_locf_records` | `NO_OUTPUT` | [case2_19_admiral_derive_locf_records.md](./case2_19_admiral_derive_locf_records.md) |
| 20 | `admiral` | `propagate_na_values` | `NO_OUTPUT` | [case2_20_admiral_propagate_na_values.md](./case2_20_admiral_propagate_na_values.md) |
| 21 | `aNCA` | `PKNCA_impute_method_start_c1` | `NO_OUTPUT` | [case2_21_aNCA_PKNCA_impute_method_start_c1.md](./case2_21_aNCA_PKNCA_impute_method_start_c1.md) |
| 22 | `aNCA` | `add_exclusion_reasons` | `NO_OUTPUT` | [case2_22_aNCA_add_exclusion_reasons.md](./case2_22_aNCA_add_exclusion_reasons.md) |
| 23 | `aNCA` | `add_impute_method` | `NO_OUTPUT` | [case2_23_aNCA_add_impute_method.md](./case2_23_aNCA_add_impute_method.md) |
| 24 | `aNCA` | `keep_blq_timepoints` | `NO_OUTPUT` | [case2_24_aNCA_keep_blq_timepoints.md](./case2_24_aNCA_keep_blq_timepoints.md) |
| 25 | `aNCA` | `detect_study_types` | `NO_OUTPUT` | [case2_25_aNCA_detect_study_types.md](./case2_25_aNCA_detect_study_types.md) |
| 26 | `aNCA` | `interval_add_impute` | `NO_OUTPUT` | [case2_26_aNCA_interval_add_impute.md](./case2_26_aNCA_interval_add_impute.md) |
| 27 | `aNCA` | `remove_impute_method` | `NO_OUTPUT` | [case2_27_aNCA_remove_impute_method.md](./case2_27_aNCA_remove_impute_method.md) |
| 28 | `aNCA` | `translate_terms` | `NO_OUTPUT` | [case2_28_aNCA_translate_terms.md](./case2_28_aNCA_translate_terms.md) |
| 29 | `ggsurvfit` | `scale_ggsurvfit` | `PASS` | [case2_29_ggsurvfit_scale_ggsurvfit.md](./case2_29_ggsurvfit_scale_ggsurvfit.md) |
| 30 | `gridify` | `get_layouts` | `FAIL` | [case2_30_gridify_get_layouts.md](./case2_30_gridify_get_layouts.md) |
