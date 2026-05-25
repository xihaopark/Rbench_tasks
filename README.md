# RBench Clinic Study Tasks

## Codex Agent Experiments

We tested Codex CLI with GPT-5.5 on the same 31 R clinical coding tasks. In each
task, the agent reads input files, writes an R solution script, and the output is
graded by an automatic evaluator.

The grouped comparison below is specifically about Codex as a coding agent:
`case4`, `case6`, and `case7` are three Codex runs on the same task set. They are
shown together because they test how different levels of prompt help affect the
same agent.

The three Codex runs differ only in how much help the prompt gives the agent:

| Folder | Agent | What the agent was given | Result |
| --- | --- | --- | ---: |
| [`case4`](./case4/) | Codex CLI / GPT-5.5 | Task description and input files only. | 19 / 31 = 61.29% |
| [`case6`](./case6/) | Codex CLI / GPT-5.5 | Same as case4, but the agent was explicitly allowed to look up public R package documentation such as CRAN, r-universe, and GitHub docs. | 19 / 31 = 61.29% |
| [`case7`](./case7/) | Codex CLI / GPT-5.5 | Same as case4, plus a short list of R package functions used by the reference solution. | 22 / 31 = 70.97% |

In short, simply allowing documentation lookup did not improve the overall
score, while giving the agent a concise package-function hint improved the
result on this 31-task subset.

## Claude Code Agent Experiments

We also tested Claude Code (claude-sonnet-4-6) on the same 31 tasks with equivalent
prompt settings, to enable a direct cross-agent comparison.

| Folder | Agent | What the agent was given | Result |
| --- | --- | --- | ---: |
| [`case8`](./case8/) | Claude Code / claude-sonnet-4-6 | Base prompt, no web access (equivalent to case4). | 15 / 31 = 48.39% |
| [`case9`](./case9/) | Claude Code / claude-sonnet-4-6 | Same as case8, but WebFetch and WebSearch tools enabled (equivalent to case6). | 13 / 31 = 41.94% |

Observation: unlike Codex, enabling web search in Claude Code did not maintain
or improve the score on this 31-task subset. The reference-function hint variant
(equivalent to case7) is pending.

---

This repository contains one Markdown document per clinical benchmark case.

Each case file includes:

- Metadata
- Pass/error
- Prompt
- Input
- Code: ground truth and LLM generated
- Output: ground truth and LLM generated

File naming convention:

```text
case1/case_<number>_<package>_<topic>.md
```

Case folders:

- `case1/`: all 197 clean clinical cases, GPT-5.1 results.
- `case2/`: selected 31 harder cases, GPT-5.1 rerun.
- `case3/`: the same 31 selected cases, GPT-5.5 rerun.
- `case4/`: the same 31 selected cases, Codex CLI with GPT-5.5.
- `case6/`: the same 31 selected cases, Codex CLI with GPT-5.5 and public package docs allowed.
- `case7/`: the same 31 selected cases, Codex CLI with GPT-5.5 and reference package-function list included.
- `case8/`: the same 31 selected cases, Claude Code with claude-sonnet-4-6, base prompt (no web).
- `case9/`: the same 31 selected cases, Claude Code with claude-sonnet-4-6, web access enabled.

Summary:

```text
Total cases: 197
Model: openai/gpt-5.1
Clean clinical strict pass@1: 62 / 197 = 0.3147
```

## Case Index

| Case | Package | Topic | Document |
| ---: | --- | --- | --- |
| 001 | `aNCA` | `PKNCA_start_c1` | [case1/case_001_aNCA_PKNCA_start_c1.md](case1/case_001_aNCA_PKNCA_start_c1.md) |
| 002 | `aNCA` | `PKNCA_start_logslope` | [case1/case_002_aNCA_PKNCA_start_logslope.md](case1/case_002_aNCA_PKNCA_start_logslope.md) |
| 003 | `aNCA` | `add_exclusion_reasons` | [case1/case_003_aNCA_add_exclusion_reasons.md](case1/case_003_aNCA_add_exclusion_reasons.md) |
| 004 | `aNCA` | `add_impute_method` | [case1/case_004_aNCA_add_impute_method.md](case1/case_004_aNCA_add_impute_method.md) |
| 005 | `aNCA` | `add_label_attribute` | [case1/case_005_aNCA_add_label_attribute.md](case1/case_005_aNCA_add_label_attribute.md) |
| 006 | `aNCA` | `adjust_class_and_length` | [case1/case_006_aNCA_adjust_class_and_length.md](case1/case_006_aNCA_adjust_class_and_length.md) |
| 007 | `aNCA` | `apply_filters` | [case1/case_007_aNCA_apply_filters.md](case1/case_007_aNCA_apply_filters.md) |
| 008 | `aNCA` | `apply_labels` | [case1/case_008_aNCA_apply_labels.md](case1/case_008_aNCA_apply_labels.md) |
| 009 | `aNCA` | `apply_mapping` | [case1/case_009_aNCA_apply_mapping.md](case1/case_009_aNCA_apply_mapping.md) |
| 010 | `aNCA` | `clean_deparse` | [case1/case_010_aNCA_clean_deparse.md](case1/case_010_aNCA_clean_deparse.md) |
| 011 | `aNCA` | `convert_to_iso8601_duration` | [case1/case_011_aNCA_convert_to_iso8601_duration.md](case1/case_011_aNCA_convert_to_iso8601_duration.md) |
| 012 | `aNCA` | `convert_volume_units` | [case1/case_012_aNCA_convert_volume_units.md](case1/case_012_aNCA_convert_volume_units.md) |
| 013 | `aNCA` | `create_metabfl` | [case1/case_013_aNCA_create_metabfl.md](case1/case_013_aNCA_create_metabfl.md) |
| 014 | `aNCA` | `create_start_impute` | [case1/case_014_aNCA_create_start_impute.md](case1/case_014_aNCA_create_start_impute.md) |
| 015 | `aNCA` | `derive_last_dose_time` | [case1/case_015_aNCA_derive_last_dose_time.md](case1/case_015_aNCA_derive_last_dose_time.md) |
| 016 | `aNCA` | `detect_study_types` | [case1/case_016_aNCA_detect_study_types.md](case1/case_016_aNCA_detect_study_types.md) |
| 017 | `aNCA` | `exploration_individualplot` | [case1/case_017_aNCA_exploration_individualplot.md](case1/case_017_aNCA_exploration_individualplot.md) |
| 018 | `aNCA` | `exploration_meanplot` | [case1/case_018_aNCA_exploration_meanplot.md](case1/case_018_aNCA_exploration_meanplot.md) |
| 019 | `aNCA` | `filter_by_list` | [case1/case_019_aNCA_filter_by_list.md](case1/case_019_aNCA_filter_by_list.md) |
| 020 | `aNCA` | `finalize_meanplot` | [case1/case_020_aNCA_finalize_meanplot.md](case1/case_020_aNCA_finalize_meanplot.md) |
| 021 | `aNCA` | `find_common_prefix` | [case1/case_021_aNCA_find_common_prefix.md](case1/case_021_aNCA_find_common_prefix.md) |
| 022 | `aNCA` | `format_pkncadata_intervals` | [case1/case_022_aNCA_format_pkncadata_intervals.md](case1/case_022_aNCA_format_pkncadata_intervals.md) |
| 023 | `aNCA` | `format_unit_string` | [case1/case_023_aNCA_format_unit_string.md](case1/case_023_aNCA_format_unit_string.md) |
| 024 | `aNCA` | `g_lineplot` | [case1/case_024_aNCA_g_lineplot.md](case1/case_024_aNCA_g_lineplot.md) |
| 025 | `aNCA` | `generate_pre_specs` | [case1/case_025_aNCA_generate_pre_specs.md](case1/case_025_aNCA_generate_pre_specs.md) |
| 026 | `aNCA` | `generate_tooltip_text` | [case1/case_026_aNCA_generate_tooltip_text.md](case1/case_026_aNCA_generate_tooltip_text.md) |
| 027 | `aNCA` | `get_code` | [case1/case_027_aNCA_get_code.md](case1/case_027_aNCA_get_code.md) |
| 028 | `aNCA` | `get_conversion_factor` | [case1/case_028_aNCA_get_conversion_factor.md](case1/case_028_aNCA_get_conversion_factor.md) |
| 029 | `aNCA` | `get_halflife_plots_single` | [case1/case_029_aNCA_get_halflife_plots_single.md](case1/case_029_aNCA_get_halflife_plots_single.md) |
| 030 | `aNCA` | `get_label` | [case1/case_030_aNCA_get_label.md](case1/case_030_aNCA_get_label.md) |
| 031 | `aNCA` | `get_session_code` | [case1/case_031_aNCA_get_session_code.md](case1/case_031_aNCA_get_session_code.md) |
| 032 | `aNCA` | `get_subjid` | [case1/case_032_aNCA_get_subjid.md](case1/case_032_aNCA_get_subjid.md) |
| 033 | `aNCA` | `identify_target_rows` | [case1/case_033_aNCA_identify_target_rows.md](case1/case_033_aNCA_identify_target_rows.md) |
| 034 | `aNCA` | `interval_add_impute` | [case1/case_034_aNCA_interval_add_impute.md](case1/case_034_aNCA_interval_add_impute.md) |
| 035 | `aNCA` | `interval_remove_impute` | [case1/case_035_aNCA_interval_remove_impute.md](case1/case_035_aNCA_interval_remove_impute.md) |
| 036 | `aNCA` | `keep_blq_timepoints` | [case1/case_036_aNCA_keep_blq_timepoints.md](case1/case_036_aNCA_keep_blq_timepoints.md) |
| 037 | `aNCA` | `l_pkcl01` | [case1/case_037_aNCA_l_pkcl01.md](case1/case_037_aNCA_l_pkcl01.md) |
| 038 | `aNCA` | `log_conversion` | [case1/case_038_aNCA_log_conversion.md](case1/case_038_aNCA_log_conversion.md) |
| 039 | `aNCA` | `multiple_matrix_ratios` | [case1/case_039_aNCA_multiple_matrix_ratios.md](case1/case_039_aNCA_multiple_matrix_ratios.md) |
| 040 | `aNCA` | `parse_annotation` | [case1/case_040_aNCA_parse_annotation.md](case1/case_040_aNCA_parse_annotation.md) |
| 041 | `aNCA` | `pk.calc.volpk` | [case1/case_041_aNCA_pk.calc.volpk.md](case1/case_041_aNCA_pk.calc.volpk.md) |
| 042 | `aNCA` | `prepare_plot_data` | [case1/case_042_aNCA_prepare_plot_data.md](case1/case_042_aNCA_prepare_plot_data.md) |
| 043 | `aNCA` | `process_data_individual` | [case1/case_043_aNCA_process_data_individual.md](case1/case_043_aNCA_process_data_individual.md) |
| 044 | `aNCA` | `process_data_mean` | [case1/case_044_aNCA_process_data_mean.md](case1/case_044_aNCA_process_data_mean.md) |
| 045 | `aNCA` | `remove_impute_method` | [case1/case_045_aNCA_remove_impute_method.md](case1/case_045_aNCA_remove_impute_method.md) |
| 046 | `aNCA` | `remove_pp_not_requested` | [case1/case_046_aNCA_remove_pp_not_requested.md](case1/case_046_aNCA_remove_pp_not_requested.md) |
| 047 | `aNCA` | `rm_impute_obs_params` | [case1/case_047_aNCA_rm_impute_obs_params.md](case1/case_047_aNCA_rm_impute_obs_params.md) |
| 048 | `aNCA` | `select_minimal_grouping_cols` | [case1/case_048_aNCA_select_minimal_grouping_cols.md](case1/case_048_aNCA_select_minimal_grouping_cols.md) |
| 049 | `aNCA` | `simplify_unit` | [case1/case_049_aNCA_simplify_unit.md](case1/case_049_aNCA_simplify_unit.md) |
| 050 | `aNCA` | `translate_terms` | [case1/case_050_aNCA_translate_terms.md](case1/case_050_aNCA_translate_terms.md) |
| 051 | `aNCA` | `update_main_intervals` | [case1/case_051_aNCA_update_main_intervals.md](case1/case_051_aNCA_update_main_intervals.md) |
| 052 | `aNCA` | `validate_pk` | [case1/case_052_aNCA_validate_pk.md](case1/case_052_aNCA_validate_pk.md) |
| 053 | `admiral` | `adjust_last_day_imputation` | [case1/case_053_admiral_adjust_last_day_imputation.md](case1/case_053_admiral_adjust_last_day_imputation.md) |
| 054 | `admiral` | `calculate_range_value` | [case1/case_054_admiral_calculate_range_value.md](case1/case_054_admiral_calculate_range_value.md) |
| 055 | `admiral` | `call_derivation` | [case1/case_055_admiral_call_derivation.md](case1/case_055_admiral_call_derivation.md) |
| 056 | `admiral` | `censor_source` | [case1/case_056_admiral_censor_source.md](case1/case_056_admiral_censor_source.md) |
| 057 | `admiral` | `chr2vars` | [case1/case_057_admiral_chr2vars.md](case1/case_057_admiral_chr2vars.md) |
| 058 | `admiral` | `compute_age_years` | [case1/case_058_admiral_compute_age_years.md](case1/case_058_admiral_compute_age_years.md) |
| 059 | `admiral` | `compute_bmi` | [case1/case_059_admiral_compute_bmi.md](case1/case_059_admiral_compute_bmi.md) |
| 060 | `admiral` | `compute_bsa` | [case1/case_060_admiral_compute_bsa.md](case1/case_060_admiral_compute_bsa.md) |
| 061 | `admiral` | `compute_dtf` | [case1/case_061_admiral_compute_dtf.md](case1/case_061_admiral_compute_dtf.md) |
| 062 | `admiral` | `compute_duration` | [case1/case_062_admiral_compute_duration.md](case1/case_062_admiral_compute_duration.md) |
| 063 | `admiral` | `compute_egfr` | [case1/case_063_admiral_compute_egfr.md](case1/case_063_admiral_compute_egfr.md) |
| 064 | `admiral` | `compute_framingham` | [case1/case_064_admiral_compute_framingham.md](case1/case_064_admiral_compute_framingham.md) |
| 065 | `admiral` | `compute_map` | [case1/case_065_admiral_compute_map.md](case1/case_065_admiral_compute_map.md) |
| 066 | `admiral` | `compute_qtc` | [case1/case_066_admiral_compute_qtc.md](case1/case_066_admiral_compute_qtc.md) |
| 067 | `admiral` | `compute_qual_imputation` | [case1/case_067_admiral_compute_qual_imputation.md](case1/case_067_admiral_compute_qual_imputation.md) |
| 068 | `admiral` | `compute_qual_imputation_dec` | [case1/case_068_admiral_compute_qual_imputation_dec.md](case1/case_068_admiral_compute_qual_imputation_dec.md) |
| 069 | `admiral` | `compute_rr` | [case1/case_069_admiral_compute_rr.md](case1/case_069_admiral_compute_rr.md) |
| 070 | `admiral` | `compute_scale` | [case1/case_070_admiral_compute_scale.md](case1/case_070_admiral_compute_scale.md) |
| 071 | `admiral` | `compute_tmf` | [case1/case_071_admiral_compute_tmf.md](case1/case_071_admiral_compute_tmf.md) |
| 072 | `admiral` | `convert_blanks_to_na` | [case1/case_072_admiral_convert_blanks_to_na.md](case1/case_072_admiral_convert_blanks_to_na.md) |
| 073 | `admiral` | `convert_date_to_dtm` | [case1/case_073_admiral_convert_date_to_dtm.md](case1/case_073_admiral_convert_date_to_dtm.md) |
| 074 | `admiral` | `convert_dtc_to_dt` | [case1/case_074_admiral_convert_dtc_to_dt.md](case1/case_074_admiral_convert_dtc_to_dt.md) |
| 075 | `admiral` | `convert_dtc_to_dtm` | [case1/case_075_admiral_convert_dtc_to_dtm.md](case1/case_075_admiral_convert_dtc_to_dtm.md) |
| 076 | `admiral` | `convert_min_after_start` | [case1/case_076_admiral_convert_min_after_start.md](case1/case_076_admiral_convert_min_after_start.md) |
| 077 | `admiral` | `convert_min_pre_eot` | [case1/case_077_admiral_convert_min_pre_eot.md](case1/case_077_admiral_convert_min_pre_eot.md) |
| 078 | `admiral` | `convert_na_to_blanks` | [case1/case_078_admiral_convert_na_to_blanks.md](case1/case_078_admiral_convert_na_to_blanks.md) |
| 079 | `admiral` | `convert_post_end_patterns` | [case1/case_079_admiral_convert_post_end_patterns.md](case1/case_079_admiral_convert_post_end_patterns.md) |
| 080 | `admiral` | `convert_predose_patterns` | [case1/case_080_admiral_convert_predose_patterns.md](case1/case_080_admiral_convert_predose_patterns.md) |
| 081 | `admiral` | `convert_ranges` | [case1/case_081_admiral_convert_ranges.md](case1/case_081_admiral_convert_ranges.md) |
| 082 | `admiral` | `convert_ranges_eot` | [case1/case_082_admiral_convert_ranges_eot.md](case1/case_082_admiral_convert_ranges_eot.md) |
| 083 | `admiral` | `convert_simple_units` | [case1/case_083_admiral_convert_simple_units.md](case1/case_083_admiral_convert_simple_units.md) |
| 084 | `admiral` | `convert_special_cases` | [case1/case_084_admiral_convert_special_cases.md](case1/case_084_admiral_convert_special_cases.md) |
| 085 | `admiral` | `convert_start_patterns` | [case1/case_085_admiral_convert_start_patterns.md](case1/case_085_admiral_convert_start_patterns.md) |
| 086 | `admiral` | `convert_time_units` | [case1/case_086_admiral_convert_time_units.md](case1/case_086_admiral_convert_time_units.md) |
| 087 | `admiral` | `convert_treatment_patterns` | [case1/case_087_admiral_convert_treatment_patterns.md](case1/case_087_admiral_convert_treatment_patterns.md) |
| 088 | `admiral` | `convert_xxtpt_to_hours` | [case1/case_088_admiral_convert_xxtpt_to_hours.md](case1/case_088_admiral_convert_xxtpt_to_hours.md) |
| 089 | `admiral` | `count_vals` | [case1/case_089_admiral_count_vals.md](case1/case_089_admiral_count_vals.md) |
| 090 | `admiral` | `create_period_dataset` | [case1/case_090_admiral_create_period_dataset.md](case1/case_090_admiral_create_period_dataset.md) |
| 091 | `admiral` | `date_source` | [case1/case_091_admiral_date_source.md](case1/case_091_admiral_date_source.md) |
| 092 | `admiral` | `default_qtc_paramcd` | [case1/case_092_admiral_default_qtc_paramcd.md](case1/case_092_admiral_default_qtc_paramcd.md) |
| 093 | `admiral` | `derive_basetype_records` | [case1/case_093_admiral_derive_basetype_records.md](case1/case_093_admiral_derive_basetype_records.md) |
| 094 | `admiral` | `derive_locf_records` | [case1/case_094_admiral_derive_locf_records.md](case1/case_094_admiral_derive_locf_records.md) |
| 095 | `admiral` | `derive_param_bmi` | [case1/case_095_admiral_derive_param_bmi.md](case1/case_095_admiral_derive_param_bmi.md) |
| 096 | `admiral` | `derive_param_bsa` | [case1/case_096_admiral_derive_param_bsa.md](case1/case_096_admiral_derive_param_bsa.md) |
| 097 | `admiral` | `derive_param_map` | [case1/case_097_admiral_derive_param_map.md](case1/case_097_admiral_derive_param_map.md) |
| 098 | `admiral` | `derive_param_qtc` | [case1/case_098_admiral_derive_param_qtc.md](case1/case_098_admiral_derive_param_qtc.md) |
| 099 | `admiral` | `derive_param_rr` | [case1/case_099_admiral_derive_param_rr.md](case1/case_099_admiral_derive_param_rr.md) |
| 100 | `admiral` | `derive_param_tte` | [case1/case_100_admiral_derive_param_tte.md](case1/case_100_admiral_derive_param_tte.md) |
| 101 | `admiral` | `derive_var_atoxgr_dir` | [case1/case_101_admiral_derive_var_atoxgr_dir.md](case1/case_101_admiral_derive_var_atoxgr_dir.md) |
| 102 | `admiral` | `derive_var_base` | [case1/case_102_admiral_derive_var_base.md](case1/case_102_admiral_derive_var_base.md) |
| 103 | `admiral` | `derive_var_ontrtfl` | [case1/case_103_admiral_derive_var_ontrtfl.md](case1/case_103_admiral_derive_var_ontrtfl.md) |
| 104 | `admiral` | `derive_var_pchg` | [case1/case_104_admiral_derive_var_pchg.md](case1/case_104_admiral_derive_var_pchg.md) |
| 105 | `admiral` | `derive_var_trtdurd` | [case1/case_105_admiral_derive_var_trtdurd.md](case1/case_105_admiral_derive_var_trtdurd.md) |
| 106 | `admiral` | `derive_var_trtemfl` | [case1/case_106_admiral_derive_var_trtemfl.md](case1/case_106_admiral_derive_var_trtemfl.md) |
| 107 | `admiral` | `derive_vars_aage` | [case1/case_107_admiral_derive_vars_aage.md](case1/case_107_admiral_derive_vars_aage.md) |
| 108 | `admiral` | `derive_vars_atc` | [case1/case_108_admiral_derive_vars_atc.md](case1/case_108_admiral_derive_vars_atc.md) |
| 109 | `admiral` | `derive_vars_cat` | [case1/case_109_admiral_derive_vars_cat.md](case1/case_109_admiral_derive_vars_cat.md) |
| 110 | `admiral` | `derive_vars_computed` | [case1/case_110_admiral_derive_vars_computed.md](case1/case_110_admiral_derive_vars_computed.md) |
| 111 | `admiral` | `derive_vars_crit_flag` | [case1/case_111_admiral_derive_vars_crit_flag.md](case1/case_111_admiral_derive_vars_crit_flag.md) |
| 112 | `admiral` | `derive_vars_dt` | [case1/case_112_admiral_derive_vars_dt.md](case1/case_112_admiral_derive_vars_dt.md) |
| 113 | `admiral` | `derive_vars_dtm` | [case1/case_113_admiral_derive_vars_dtm.md](case1/case_113_admiral_derive_vars_dtm.md) |
| 114 | `admiral` | `derive_vars_dtm_to_dt` | [case1/case_114_admiral_derive_vars_dtm_to_dt.md](case1/case_114_admiral_derive_vars_dtm_to_dt.md) |
| 115 | `admiral` | `derive_vars_dtm_to_tm` | [case1/case_115_admiral_derive_vars_dtm_to_tm.md](case1/case_115_admiral_derive_vars_dtm_to_tm.md) |
| 116 | `admiral` | `derive_vars_duration` | [case1/case_116_admiral_derive_vars_duration.md](case1/case_116_admiral_derive_vars_duration.md) |
| 117 | `admiral` | `derive_vars_dy` | [case1/case_117_admiral_derive_vars_dy.md](case1/case_117_admiral_derive_vars_dy.md) |
| 118 | `admiral` | `derive_vars_extreme_event` | [case1/case_118_admiral_derive_vars_extreme_event.md](case1/case_118_admiral_derive_vars_extreme_event.md) |
| 119 | `admiral` | `derive_vars_merged` | [case1/case_119_admiral_derive_vars_merged.md](case1/case_119_admiral_derive_vars_merged.md) |
| 120 | `admiral` | `derive_vars_merged_lookup` | [case1/case_120_admiral_derive_vars_merged_lookup.md](case1/case_120_admiral_derive_vars_merged_lookup.md) |
| 121 | `admiral` | `derive_vars_merged_summary` | [case1/case_121_admiral_derive_vars_merged_summary.md](case1/case_121_admiral_derive_vars_merged_summary.md) |
| 122 | `admiral` | `derive_vars_period` | [case1/case_122_admiral_derive_vars_period.md](case1/case_122_admiral_derive_vars_period.md) |
| 123 | `admiral` | `derive_vars_query` | [case1/case_123_admiral_derive_vars_query.md](case1/case_123_admiral_derive_vars_query.md) |
| 124 | `admiral` | `derive_vars_transposed` | [case1/case_124_admiral_derive_vars_transposed.md](case1/case_124_admiral_derive_vars_transposed.md) |
| 125 | `admiral` | `dt_level` | [case1/case_125_admiral_dt_level.md](case1/case_125_admiral_dt_level.md) |
| 126 | `admiral` | `dthcaus_source` | [case1/case_126_admiral_dthcaus_source.md](case1/case_126_admiral_dthcaus_source.md) |
| 127 | `admiral` | `dtm_level` | [case1/case_127_admiral_dtm_level.md](case1/case_127_admiral_dtm_level.md) |
| 128 | `admiral` | `event` | [case1/case_128_admiral_event.md](case1/case_128_admiral_event.md) |
| 129 | `admiral` | `event_joined` | [case1/case_129_admiral_event_joined.md](case1/case_129_admiral_event_joined.md) |
| 130 | `admiral` | `event_source` | [case1/case_130_admiral_event_source.md](case1/case_130_admiral_event_source.md) |
| 131 | `admiral` | `extend_source_datasets` | [case1/case_131_admiral_extend_source_datasets.md](case1/case_131_admiral_extend_source_datasets.md) |
| 132 | `admiral` | `extract_duplicate_records` | [case1/case_132_admiral_extract_duplicate_records.md](case1/case_132_admiral_extract_duplicate_records.md) |
| 133 | `admiral` | `extract_unit` | [case1/case_133_admiral_extract_unit.md](case1/case_133_admiral_extract_unit.md) |
| 134 | `admiral` | `filter_exist` | [case1/case_134_admiral_filter_exist.md](case1/case_134_admiral_filter_exist.md) |
| 135 | `admiral` | `filter_extreme` | [case1/case_135_admiral_filter_extreme.md](case1/case_135_admiral_filter_extreme.md) |
| 136 | `admiral` | `filter_not_exist` | [case1/case_136_admiral_filter_not_exist.md](case1/case_136_admiral_filter_not_exist.md) |
| 137 | `admiral` | `filter_relative` | [case1/case_137_admiral_filter_relative.md](case1/case_137_admiral_filter_relative.md) |
| 138 | `admiral` | `flag_event` | [case1/case_138_admiral_flag_event.md](case1/case_138_admiral_flag_event.md) |
| 139 | `admiral` | `format_imputed_dtc` | [case1/case_139_admiral_format_imputed_dtc.md](case1/case_139_admiral_format_imputed_dtc.md) |
| 140 | `admiral` | `get_admiral_option` | [case1/case_140_admiral_get_admiral_option.md](case1/case_140_admiral_get_admiral_option.md) |
| 141 | `admiral` | `get_dt_dtm_range` | [case1/case_141_admiral_get_dt_dtm_range.md](case1/case_141_admiral_get_dt_dtm_range.md) |
| 142 | `admiral` | `get_flagged_records` | [case1/case_142_admiral_get_flagged_records.md](case1/case_142_admiral_get_flagged_records.md) |
| 143 | `admiral` | `highest_impute_level` | [case1/case_143_admiral_highest_impute_level.md](case1/case_143_admiral_highest_impute_level.md) |
| 144 | `admiral` | `get_hori_data` | [case1/case_144_admiral_get_hori_data.md](case1/case_144_admiral_get_hori_data.md) |
| 145 | `admiral` | `impute_target_date` | [case1/case_145_admiral_impute_target_date.md](case1/case_145_admiral_impute_target_date.md) |
| 146 | `admiral` | `impute_target_time` | [case1/case_146_admiral_impute_target_time.md](case1/case_146_admiral_impute_target_time.md) |
| 147 | `admiral` | `get_imputation_targets` | [case1/case_147_admiral_get_imputation_targets.md](case1/case_147_admiral_get_imputation_targets.md) |
| 148 | `admiral` | `get_joined_sub_data` | [case1/case_148_admiral_get_joined_sub_data.md](case1/case_148_admiral_get_joined_sub_data.md) |
| 149 | `admiral` | `get_partialdatetime` | [case1/case_149_admiral_get_partialdatetime.md](case1/case_149_admiral_get_partialdatetime.md) |
| 150 | `admiral` | `get_summary_records` | [case1/case_150_admiral_get_summary_records.md](case1/case_150_admiral_get_summary_records.md) |
| 151 | `admiral` | `get_unified_time_unit` | [case1/case_151_admiral_get_unified_time_unit.md](case1/case_151_admiral_get_unified_time_unit.md) |
| 152 | `admiral` | `get_vars_query` | [case1/case_152_admiral_get_vars_query.md](case1/case_152_admiral_get_vars_query.md) |
| 153 | `admiral` | `impute_date_time` | [case1/case_153_admiral_impute_date_time.md](case1/case_153_admiral_impute_date_time.md) |
| 154 | `admiral` | `impute_dtc_dt` | [case1/case_154_admiral_impute_dtc_dt.md](case1/case_154_admiral_impute_dtc_dt.md) |
| 155 | `admiral` | `impute_dtc_dtm` | [case1/case_155_admiral_impute_dtc_dtm.md](case1/case_155_admiral_impute_dtc_dtm.md) |
| 156 | `admiral` | `is_partial_datetime` | [case1/case_156_admiral_is_partial_datetime.md](case1/case_156_admiral_is_partial_datetime.md) |
| 157 | `admiral` | `list_all_templates` | [case1/case_157_admiral_list_all_templates.md](case1/case_157_admiral_list_all_templates.md) |
| 158 | `admiral` | `list_tte_source_objects` | [case1/case_158_admiral_list_tte_source_objects.md](case1/case_158_admiral_list_tte_source_objects.md) |
| 159 | `admiral` | `print_named_list` | [case1/case_159_admiral_print_named_list.md](case1/case_159_admiral_print_named_list.md) |
| 160 | `admiral` | `propagate_na_values` | [case1/case_160_admiral_propagate_na_values.md](case1/case_160_admiral_propagate_na_values.md) |
| 161 | `admiral` | `restrict_dtc_dt` | [case1/case_161_admiral_restrict_dtc_dt.md](case1/case_161_admiral_restrict_dtc_dt.md) |
| 162 | `admiral` | `restrict_dtc_dtm` | [case1/case_162_admiral_restrict_dtc_dtm.md](case1/case_162_admiral_restrict_dtc_dtm.md) |
| 163 | `admiral` | `set_admiral_options` | [case1/case_163_admiral_set_admiral_options.md](case1/case_163_admiral_set_admiral_options.md) |
| 164 | `admiral` | `slice_derivation` | [case1/case_164_admiral_slice_derivation.md](case1/case_164_admiral_slice_derivation.md) |
| 165 | `admiral` | `yn_to_numeric` | [case1/case_165_admiral_yn_to_numeric.md](case1/case_165_admiral_yn_to_numeric.md) |
| 166 | `admiraldev` | `backquote` | [case1/case_166_admiraldev_backquote.md](case1/case_166_admiraldev_backquote.md) |
| 167 | `admiraldev` | `dquote` | [case1/case_167_admiraldev_dquote.md](case1/case_167_admiraldev_dquote.md) |
| 168 | `admiraldev` | `get_duplicates` | [case1/case_168_admiraldev_get_duplicates.md](case1/case_168_admiraldev_get_duplicates.md) |
| 169 | `admiraldev` | `get_source_vars` | [case1/case_169_admiraldev_get_source_vars.md](case1/case_169_admiraldev_get_source_vars.md) |
| 170 | `admiraldev` | `is_order_vars` | [case1/case_170_admiraldev_is_order_vars.md](case1/case_170_admiraldev_is_order_vars.md) |
| 171 | `admiraldev` | `is_valid_dtc` | [case1/case_171_admiraldev_is_valid_dtc.md](case1/case_171_admiraldev_is_valid_dtc.md) |
| 172 | `admiraldev` | `parse_code` | [case1/case_172_admiraldev_parse_code.md](case1/case_172_admiraldev_parse_code.md) |
| 173 | `admiraldev` | `process_set_values_to` | [case1/case_173_admiraldev_process_set_values_to.md](case1/case_173_admiraldev_process_set_values_to.md) |
| 174 | `admiraldev` | `replace_values_by_names` | [case1/case_174_admiraldev_replace_values_by_names.md](case1/case_174_admiraldev_replace_values_by_names.md) |
| 175 | `admiraldev` | `squote` | [case1/case_175_admiraldev_squote.md](case1/case_175_admiraldev_squote.md) |
| 176 | `admiraldev` | `suppress_warning` | [case1/case_176_admiraldev_suppress_warning.md](case1/case_176_admiraldev_suppress_warning.md) |
| 177 | `admiraldiscovery` | `admiral_pkg_versions` | [case1/case_177_admiraldiscovery_admiral_pkg_versions.md](case1/case_177_admiraldiscovery_admiral_pkg_versions.md) |
| 178 | `envsetup` | `detach_autos` | [case1/case_178_envsetup_detach_autos.md](case1/case_178_envsetup_detach_autos.md) |
| 179 | `ggsurvfit` | `scale_ggsurvfit` | [case1/case_179_ggsurvfit_scale_ggsurvfit.md](case1/case_179_ggsurvfit_scale_ggsurvfit.md) |
| 180 | `gridify` | `get_layouts` | [case1/case_180_gridify_get_layouts.md](case1/case_180_gridify_get_layouts.md) |
| 181 | `logrx` | `log_config` | [case1/case_181_logrx_log_config.md](case1/case_181_logrx_log_config.md) |
| 182 | `logrx` | `log_init` | [case1/case_182_logrx_log_init.md](case1/case_182_logrx_log_init.md) |
| 183 | `logrx` | `parse_log` | [case1/case_183_logrx_parse_log.md](case1/case_183_logrx_parse_log.md) |
| 184 | `logrx` | `reformat_subsections` | [case1/case_184_logrx_reformat_subsections.md](case1/case_184_logrx_reformat_subsections.md) |
| 185 | `metatools` | `add_labels` | [case1/case_185_metatools_add_labels.md](case1/case_185_metatools_add_labels.md) |
| 186 | `metatools` | `create_subgrps` | [case1/case_186_metatools_create_subgrps.md](case1/case_186_metatools_create_subgrps.md) |
| 187 | `metatools` | `dash_to_eq` | [case1/case_187_metatools_dash_to_eq.md](case1/case_187_metatools_dash_to_eq.md) |
| 188 | `metatools` | `remove_labels` | [case1/case_188_metatools_remove_labels.md](case1/case_188_metatools_remove_labels.md) |
| 189 | `metatools` | `validate_verbose` | [case1/case_189_metatools_validate_verbose.md](case1/case_189_metatools_validate_verbose.md) |
| 190 | `sdtm.oak` | `iso8601_sec` | [case1/case_190_sdtm.oak_iso8601_sec.md](case1/case_190_sdtm.oak_iso8601_sec.md) |
| 191 | `sdtm.oak` | `sbj_vars` | [case1/case_191_sdtm.oak_sbj_vars.md](case1/case_191_sdtm.oak_sbj_vars.md) |
| 192 | `sdtm.oak` | `str_to_anycase` | [case1/case_192_sdtm.oak_str_to_anycase.md](case1/case_192_sdtm.oak_str_to_anycase.md) |
| 193 | `sdtmchecks` | `fail` | [case1/case_193_sdtmchecks_fail.md](case1/case_193_sdtmchecks_fail.md) |
| 194 | `sdtmchecks` | `pass` | [case1/case_194_sdtmchecks_pass.md](case1/case_194_sdtmchecks_pass.md) |
| 195 | `tidytlg` | `add_indent` | [case1/case_195_tidytlg_add_indent.md](case1/case_195_tidytlg_add_indent.md) |
| 196 | `tidytlg` | `col_borders` | [case1/case_196_tidytlg_col_borders.md](case1/case_196_tidytlg_col_borders.md) |
| 197 | `tidytlg` | `replace_na_with_blank` | [case1/case_197_tidytlg_replace_na_with_blank.md](case1/case_197_tidytlg_replace_na_with_blank.md) |
