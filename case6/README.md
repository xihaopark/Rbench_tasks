# case6

31 Codex CLI GPT-5.5 clinical benchmark probe cases with the hidden reference solution package function list included in the prompt.

Each case uses the same presentation template as the previous GitHub case batches. The generated code shown here is the evaluator candidate artifact (`generated_solution.R`) from the Codex run, not a regenerated sample.

Summary:

```text
Total cases: 31
Agent: Codex CLI
Model: gpt-5.5
Prompt setting: sanitized metadata; reference package function list included
Run directory: internal/coding_agent_outputs/case2_31/codex/gpt-5.5/case7_ref_functions
Strict pass@1: 22 / 31 = 0.7097
Status counts: {'PASS': 22, 'FAIL': 9}
Invalid/internal package API pattern: 0 / 31
```

Fairness note:

- Normal exported APIs from installed packages are treated as supported.
- Missing functions, non-exported helper calls via `pkg::`, and `getFromNamespace()` access to internal helpers are not treated as environment-missing failures.

## Case6 Index

| Case | Package | Topic | Status | Pattern | Document |
| ---: | --- | --- | --- | --- | --- |
| 001 | `aNCA` | `PKNCA_impute_method_start_c1` | `PASS` | `` | [case6_001_aNCA_PKNCA_impute_method_start_c1.md](./case6_001_aNCA_PKNCA_impute_method_start_c1.md) |
| 003 | `aNCA` | `add_exclusion_reasons` | `PASS` | `` | [case6_003_aNCA_add_exclusion_reasons.md](./case6_003_aNCA_add_exclusion_reasons.md) |
| 004 | `aNCA` | `add_impute_method` | `FAIL` | `` | [case6_004_aNCA_add_impute_method.md](./case6_004_aNCA_add_impute_method.md) |
| 016 | `aNCA` | `detect_study_types` | `FAIL` | `` | [case6_016_aNCA_detect_study_types.md](./case6_016_aNCA_detect_study_types.md) |
| 034 | `aNCA` | `interval_add_impute` | `PASS` | `` | [case6_034_aNCA_interval_add_impute.md](./case6_034_aNCA_interval_add_impute.md) |
| 036 | `aNCA` | `keep_blq_timepoints` | `PASS` | `` | [case6_036_aNCA_keep_blq_timepoints.md](./case6_036_aNCA_keep_blq_timepoints.md) |
| 045 | `aNCA` | `remove_impute_method` | `PASS` | `` | [case6_045_aNCA_remove_impute_method.md](./case6_045_aNCA_remove_impute_method.md) |
| 050 | `aNCA` | `translate_terms` | `PASS` | `` | [case6_050_aNCA_translate_terms.md](./case6_050_aNCA_translate_terms.md) |
| 063 | `admiral` | `compute_egfr` | `FAIL` | `` | [case6_063_admiral_compute_egfr.md](./case6_063_admiral_compute_egfr.md) |
| 064 | `admiral` | `compute_framingham` | `PASS` | `` | [case6_064_admiral_compute_framingham.md](./case6_064_admiral_compute_framingham.md) |
| 065 | `admiral` | `compute_map` | `PASS` | `` | [case6_065_admiral_compute_map.md](./case6_065_admiral_compute_map.md) |
| 071 | `admiral` | `compute_tmf` | `PASS` | `` | [case6_071_admiral_compute_tmf.md](./case6_071_admiral_compute_tmf.md) |
| 086 | `admiral` | `convert_time_units` | `PASS` | `` | [case6_086_admiral_convert_time_units.md](./case6_086_admiral_convert_time_units.md) |
| 087 | `admiral` | `convert_treatment_patterns` | `FAIL` | `` | [case6_087_admiral_convert_treatment_patterns.md](./case6_087_admiral_convert_treatment_patterns.md) |
| 088 | `admiral` | `convert_xxtpt_to_hours` | `PASS` | `` | [case6_088_admiral_convert_xxtpt_to_hours.md](./case6_088_admiral_convert_xxtpt_to_hours.md) |
| 094 | `admiral` | `derive_locf_records` | `FAIL` | `` | [case6_094_admiral_derive_locf_records.md](./case6_094_admiral_derive_locf_records.md) |
| 097 | `admiral` | `derive_param_map` | `PASS` | `` | [case6_097_admiral_derive_param_map.md](./case6_097_admiral_derive_param_map.md) |
| 098 | `admiral` | `derive_param_qtc` | `FAIL` | `` | [case6_098_admiral_derive_param_qtc.md](./case6_098_admiral_derive_param_qtc.md) |
| 100 | `admiral` | `derive_param_tte` | `FAIL` | `` | [case6_100_admiral_derive_param_tte.md](./case6_100_admiral_derive_param_tte.md) |
| 101 | `admiral` | `derive_var_atoxgr_dir` | `PASS` | `` | [case6_101_admiral_derive_var_atoxgr_dir.md](./case6_101_admiral_derive_var_atoxgr_dir.md) |
| 106 | `admiral` | `derive_var_trtemfl` | `PASS` | `` | [case6_106_admiral_derive_var_trtemfl.md](./case6_106_admiral_derive_var_trtemfl.md) |
| 111 | `admiral` | `derive_vars_crit_flag` | `PASS` | `` | [case6_111_admiral_derive_vars_crit_flag.md](./case6_111_admiral_derive_vars_crit_flag.md) |
| 118 | `admiral` | `derive_vars_extreme_event` | `PASS` | `` | [case6_118_admiral_derive_vars_extreme_event.md](./case6_118_admiral_derive_vars_extreme_event.md) |
| 123 | `admiral` | `derive_vars_query` | `PASS` | `` | [case6_123_admiral_derive_vars_query.md](./case6_123_admiral_derive_vars_query.md) |
| 142 | `admiral` | `get_flagged_records` | `FAIL` | `` | [case6_142_admiral_get_flagged_records.md](./case6_142_admiral_get_flagged_records.md) |
| 147 | `admiral` | `get_imputation_targets` | `PASS` | `` | [case6_147_admiral_get_imputation_targets.md](./case6_147_admiral_get_imputation_targets.md) |
| 148 | `admiral` | `get_joined_sub_data` | `PASS` | `` | [case6_148_admiral_get_joined_sub_data.md](./case6_148_admiral_get_joined_sub_data.md) |
| 160 | `admiral` | `propagate_na_values` | `PASS` | `` | [case6_160_admiral_propagate_na_values.md](./case6_160_admiral_propagate_na_values.md) |
| 164 | `admiral` | `slice_derivation` | `FAIL` | `` | [case6_164_admiral_slice_derivation.md](./case6_164_admiral_slice_derivation.md) |
| 179 | `ggsurvfit` | `scale_ggsurvfit` | `PASS` | `` | [case6_179_ggsurvfit_scale_ggsurvfit.md](./case6_179_ggsurvfit_scale_ggsurvfit.md) |
| 180 | `gridify` | `get_layouts` | `PASS` | `` | [case6_180_gridify_get_layouts.md](./case6_180_gridify_get_layouts.md) |
