# case10

31 Claude Code (claude-sonnet-4-6) clinical benchmark-light cases with reference package-function hints. This is the Claude Code counterpart to the Codex reference-function run in case6.

Each case uses the same presentation template as the previous GitHub case batches. The generated code shown here is the evaluator candidate artifact (`generated_solution.R`) from the Claude Code run, not a regenerated sample.

Summary:

```text
Total cases: 31
Agent: Claude Code
Model: claude-code/claude-sonnet-4-6
Prompt setting: benchmark-light prompt plus reference package-function list; all 31 CLI tasks completed
Run directory: internal/coding_agent_outputs/case2_31/claude-code/claude-sonnet-4-6/case10_ref_functions_benchmark_light
Strict pass@1: 17 / 31 = 0.5484
Status counts: {'FAIL': 14, 'PASS': 17}
Invalid/internal package API pattern: 0 / 31
```

Fairness note:

- Normal exported APIs from installed packages are treated as supported.
- Missing functions, non-exported helper calls via `pkg::`, and `getFromNamespace()` access to internal helpers are not treated as environment-missing failures.

## Case10 Index

| Case | Package | Topic | Status | Pattern | Document |
| ---: | --- | --- | --- | --- | --- |
| 001 | `aNCA` | `PKNCA_impute_method_start_c1` | `FAIL` | `` | [case10_001_aNCA_PKNCA_impute_method_start_c1.md](./case10_001_aNCA_PKNCA_impute_method_start_c1.md) |
| 003 | `aNCA` | `add_exclusion_reasons` | `PASS` | `` | [case10_003_aNCA_add_exclusion_reasons.md](./case10_003_aNCA_add_exclusion_reasons.md) |
| 004 | `aNCA` | `add_impute_method` | `FAIL` | `` | [case10_004_aNCA_add_impute_method.md](./case10_004_aNCA_add_impute_method.md) |
| 016 | `aNCA` | `detect_study_types` | `FAIL` | `` | [case10_016_aNCA_detect_study_types.md](./case10_016_aNCA_detect_study_types.md) |
| 034 | `aNCA` | `interval_add_impute` | `FAIL` | `` | [case10_034_aNCA_interval_add_impute.md](./case10_034_aNCA_interval_add_impute.md) |
| 036 | `aNCA` | `keep_blq_timepoints` | `PASS` | `` | [case10_036_aNCA_keep_blq_timepoints.md](./case10_036_aNCA_keep_blq_timepoints.md) |
| 045 | `aNCA` | `remove_impute_method` | `PASS` | `` | [case10_045_aNCA_remove_impute_method.md](./case10_045_aNCA_remove_impute_method.md) |
| 050 | `aNCA` | `translate_terms` | `PASS` | `` | [case10_050_aNCA_translate_terms.md](./case10_050_aNCA_translate_terms.md) |
| 063 | `admiral` | `compute_egfr` | `FAIL` | `` | [case10_063_admiral_compute_egfr.md](./case10_063_admiral_compute_egfr.md) |
| 064 | `admiral` | `compute_framingham` | `PASS` | `` | [case10_064_admiral_compute_framingham.md](./case10_064_admiral_compute_framingham.md) |
| 065 | `admiral` | `compute_map` | `PASS` | `` | [case10_065_admiral_compute_map.md](./case10_065_admiral_compute_map.md) |
| 071 | `admiral` | `compute_tmf` | `PASS` | `` | [case10_071_admiral_compute_tmf.md](./case10_071_admiral_compute_tmf.md) |
| 086 | `admiral` | `convert_time_units` | `FAIL` | `` | [case10_086_admiral_convert_time_units.md](./case10_086_admiral_convert_time_units.md) |
| 087 | `admiral` | `convert_treatment_patterns` | `FAIL` | `` | [case10_087_admiral_convert_treatment_patterns.md](./case10_087_admiral_convert_treatment_patterns.md) |
| 088 | `admiral` | `convert_xxtpt_to_hours` | `PASS` | `` | [case10_088_admiral_convert_xxtpt_to_hours.md](./case10_088_admiral_convert_xxtpt_to_hours.md) |
| 094 | `admiral` | `derive_locf_records` | `FAIL` | `` | [case10_094_admiral_derive_locf_records.md](./case10_094_admiral_derive_locf_records.md) |
| 097 | `admiral` | `derive_param_map` | `PASS` | `` | [case10_097_admiral_derive_param_map.md](./case10_097_admiral_derive_param_map.md) |
| 098 | `admiral` | `derive_param_qtc` | `FAIL` | `` | [case10_098_admiral_derive_param_qtc.md](./case10_098_admiral_derive_param_qtc.md) |
| 100 | `admiral` | `derive_param_tte` | `FAIL` | `` | [case10_100_admiral_derive_param_tte.md](./case10_100_admiral_derive_param_tte.md) |
| 101 | `admiral` | `derive_var_atoxgr_dir` | `PASS` | `` | [case10_101_admiral_derive_var_atoxgr_dir.md](./case10_101_admiral_derive_var_atoxgr_dir.md) |
| 106 | `admiral` | `derive_var_trtemfl` | `FAIL` | `` | [case10_106_admiral_derive_var_trtemfl.md](./case10_106_admiral_derive_var_trtemfl.md) |
| 111 | `admiral` | `derive_vars_crit_flag` | `FAIL` | `` | [case10_111_admiral_derive_vars_crit_flag.md](./case10_111_admiral_derive_vars_crit_flag.md) |
| 118 | `admiral` | `derive_vars_extreme_event` | `PASS` | `` | [case10_118_admiral_derive_vars_extreme_event.md](./case10_118_admiral_derive_vars_extreme_event.md) |
| 123 | `admiral` | `derive_vars_query` | `PASS` | `` | [case10_123_admiral_derive_vars_query.md](./case10_123_admiral_derive_vars_query.md) |
| 142 | `admiral` | `get_flagged_records` | `FAIL` | `` | [case10_142_admiral_get_flagged_records.md](./case10_142_admiral_get_flagged_records.md) |
| 147 | `admiral` | `get_imputation_targets` | `PASS` | `` | [case10_147_admiral_get_imputation_targets.md](./case10_147_admiral_get_imputation_targets.md) |
| 148 | `admiral` | `get_joined_sub_data` | `PASS` | `` | [case10_148_admiral_get_joined_sub_data.md](./case10_148_admiral_get_joined_sub_data.md) |
| 160 | `admiral` | `propagate_na_values` | `PASS` | `` | [case10_160_admiral_propagate_na_values.md](./case10_160_admiral_propagate_na_values.md) |
| 164 | `admiral` | `slice_derivation` | `FAIL` | `` | [case10_164_admiral_slice_derivation.md](./case10_164_admiral_slice_derivation.md) |
| 179 | `ggsurvfit` | `scale_ggsurvfit` | `PASS` | `` | [case10_179_ggsurvfit_scale_ggsurvfit.md](./case10_179_ggsurvfit_scale_ggsurvfit.md) |
| 180 | `gridify` | `get_layouts` | `PASS` | `` | [case10_180_gridify_get_layouts.md](./case10_180_gridify_get_layouts.md) |
