# RBench Clinic Study Tasks

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
cases/case_<number>_<package>_<topic>.md
```

Case folders:

- `cases/`: all 197 clean clinical cases, GPT-5.1 results.
- `case2/`: selected 31 harder cases, GPT-5.1 rerun.
- `case3/`: the same 31 selected cases, GPT-5.5 rerun.

Summary:

```text
Total cases: 197
Model: openai/gpt-5.1
Clean clinical strict pass@1: 62 / 197 = 0.3147
```

## Case Index

| Case | Package | Topic | Document |
| ---: | --- | --- | --- |
| 001 | `aNCA` | `PKNCA_start_c1` | [cases/case_001_aNCA_PKNCA_start_c1.md](cases/case_001_aNCA_PKNCA_start_c1.md) |
| 002 | `aNCA` | `PKNCA_start_logslope` | [cases/case_002_aNCA_PKNCA_start_logslope.md](cases/case_002_aNCA_PKNCA_start_logslope.md) |
| 003 | `aNCA` | `add_exclusion_reasons` | [cases/case_003_aNCA_add_exclusion_reasons.md](cases/case_003_aNCA_add_exclusion_reasons.md) |
| 004 | `aNCA` | `add_impute_method` | [cases/case_004_aNCA_add_impute_method.md](cases/case_004_aNCA_add_impute_method.md) |
| 005 | `aNCA` | `add_label_attribute` | [cases/case_005_aNCA_add_label_attribute.md](cases/case_005_aNCA_add_label_attribute.md) |
| 006 | `aNCA` | `adjust_class_and_length` | [cases/case_006_aNCA_adjust_class_and_length.md](cases/case_006_aNCA_adjust_class_and_length.md) |
| 007 | `aNCA` | `apply_filters` | [cases/case_007_aNCA_apply_filters.md](cases/case_007_aNCA_apply_filters.md) |
| 008 | `aNCA` | `apply_labels` | [cases/case_008_aNCA_apply_labels.md](cases/case_008_aNCA_apply_labels.md) |
| 009 | `aNCA` | `apply_mapping` | [cases/case_009_aNCA_apply_mapping.md](cases/case_009_aNCA_apply_mapping.md) |
| 010 | `aNCA` | `clean_deparse` | [cases/case_010_aNCA_clean_deparse.md](cases/case_010_aNCA_clean_deparse.md) |
| 011 | `aNCA` | `convert_to_iso8601_duration` | [cases/case_011_aNCA_convert_to_iso8601_duration.md](cases/case_011_aNCA_convert_to_iso8601_duration.md) |
| 012 | `aNCA` | `convert_volume_units` | [cases/case_012_aNCA_convert_volume_units.md](cases/case_012_aNCA_convert_volume_units.md) |
| 013 | `aNCA` | `create_metabfl` | [cases/case_013_aNCA_create_metabfl.md](cases/case_013_aNCA_create_metabfl.md) |
| 014 | `aNCA` | `create_start_impute` | [cases/case_014_aNCA_create_start_impute.md](cases/case_014_aNCA_create_start_impute.md) |
| 015 | `aNCA` | `derive_last_dose_time` | [cases/case_015_aNCA_derive_last_dose_time.md](cases/case_015_aNCA_derive_last_dose_time.md) |
| 016 | `aNCA` | `detect_study_types` | [cases/case_016_aNCA_detect_study_types.md](cases/case_016_aNCA_detect_study_types.md) |
| 017 | `aNCA` | `exploration_individualplot` | [cases/case_017_aNCA_exploration_individualplot.md](cases/case_017_aNCA_exploration_individualplot.md) |
| 018 | `aNCA` | `exploration_meanplot` | [cases/case_018_aNCA_exploration_meanplot.md](cases/case_018_aNCA_exploration_meanplot.md) |
| 019 | `aNCA` | `filter_by_list` | [cases/case_019_aNCA_filter_by_list.md](cases/case_019_aNCA_filter_by_list.md) |
| 020 | `aNCA` | `finalize_meanplot` | [cases/case_020_aNCA_finalize_meanplot.md](cases/case_020_aNCA_finalize_meanplot.md) |
| 021 | `aNCA` | `find_common_prefix` | [cases/case_021_aNCA_find_common_prefix.md](cases/case_021_aNCA_find_common_prefix.md) |
| 022 | `aNCA` | `format_pkncadata_intervals` | [cases/case_022_aNCA_format_pkncadata_intervals.md](cases/case_022_aNCA_format_pkncadata_intervals.md) |
| 023 | `aNCA` | `format_unit_string` | [cases/case_023_aNCA_format_unit_string.md](cases/case_023_aNCA_format_unit_string.md) |
| 024 | `aNCA` | `g_lineplot` | [cases/case_024_aNCA_g_lineplot.md](cases/case_024_aNCA_g_lineplot.md) |
| 025 | `aNCA` | `generate_pre_specs` | [cases/case_025_aNCA_generate_pre_specs.md](cases/case_025_aNCA_generate_pre_specs.md) |
| 026 | `aNCA` | `generate_tooltip_text` | [cases/case_026_aNCA_generate_tooltip_text.md](cases/case_026_aNCA_generate_tooltip_text.md) |
| 027 | `aNCA` | `get_code` | [cases/case_027_aNCA_get_code.md](cases/case_027_aNCA_get_code.md) |
| 028 | `aNCA` | `get_conversion_factor` | [cases/case_028_aNCA_get_conversion_factor.md](cases/case_028_aNCA_get_conversion_factor.md) |
| 029 | `aNCA` | `get_halflife_plots_single` | [cases/case_029_aNCA_get_halflife_plots_single.md](cases/case_029_aNCA_get_halflife_plots_single.md) |
| 030 | `aNCA` | `get_label` | [cases/case_030_aNCA_get_label.md](cases/case_030_aNCA_get_label.md) |
| 031 | `aNCA` | `get_session_code` | [cases/case_031_aNCA_get_session_code.md](cases/case_031_aNCA_get_session_code.md) |
| 032 | `aNCA` | `get_subjid` | [cases/case_032_aNCA_get_subjid.md](cases/case_032_aNCA_get_subjid.md) |
| 033 | `aNCA` | `identify_target_rows` | [cases/case_033_aNCA_identify_target_rows.md](cases/case_033_aNCA_identify_target_rows.md) |
| 034 | `aNCA` | `interval_add_impute` | [cases/case_034_aNCA_interval_add_impute.md](cases/case_034_aNCA_interval_add_impute.md) |
| 035 | `aNCA` | `interval_remove_impute` | [cases/case_035_aNCA_interval_remove_impute.md](cases/case_035_aNCA_interval_remove_impute.md) |
| 036 | `aNCA` | `keep_blq_timepoints` | [cases/case_036_aNCA_keep_blq_timepoints.md](cases/case_036_aNCA_keep_blq_timepoints.md) |
| 037 | `aNCA` | `l_pkcl01` | [cases/case_037_aNCA_l_pkcl01.md](cases/case_037_aNCA_l_pkcl01.md) |
| 038 | `aNCA` | `log_conversion` | [cases/case_038_aNCA_log_conversion.md](cases/case_038_aNCA_log_conversion.md) |
| 039 | `aNCA` | `multiple_matrix_ratios` | [cases/case_039_aNCA_multiple_matrix_ratios.md](cases/case_039_aNCA_multiple_matrix_ratios.md) |
| 040 | `aNCA` | `parse_annotation` | [cases/case_040_aNCA_parse_annotation.md](cases/case_040_aNCA_parse_annotation.md) |
| 041 | `aNCA` | `pk.calc.volpk` | [cases/case_041_aNCA_pk.calc.volpk.md](cases/case_041_aNCA_pk.calc.volpk.md) |
| 042 | `aNCA` | `prepare_plot_data` | [cases/case_042_aNCA_prepare_plot_data.md](cases/case_042_aNCA_prepare_plot_data.md) |
| 043 | `aNCA` | `process_data_individual` | [cases/case_043_aNCA_process_data_individual.md](cases/case_043_aNCA_process_data_individual.md) |
| 044 | `aNCA` | `process_data_mean` | [cases/case_044_aNCA_process_data_mean.md](cases/case_044_aNCA_process_data_mean.md) |
| 045 | `aNCA` | `remove_impute_method` | [cases/case_045_aNCA_remove_impute_method.md](cases/case_045_aNCA_remove_impute_method.md) |
| 046 | `aNCA` | `remove_pp_not_requested` | [cases/case_046_aNCA_remove_pp_not_requested.md](cases/case_046_aNCA_remove_pp_not_requested.md) |
| 047 | `aNCA` | `rm_impute_obs_params` | [cases/case_047_aNCA_rm_impute_obs_params.md](cases/case_047_aNCA_rm_impute_obs_params.md) |
| 048 | `aNCA` | `select_minimal_grouping_cols` | [cases/case_048_aNCA_select_minimal_grouping_cols.md](cases/case_048_aNCA_select_minimal_grouping_cols.md) |
| 049 | `aNCA` | `simplify_unit` | [cases/case_049_aNCA_simplify_unit.md](cases/case_049_aNCA_simplify_unit.md) |
| 050 | `aNCA` | `translate_terms` | [cases/case_050_aNCA_translate_terms.md](cases/case_050_aNCA_translate_terms.md) |
| 051 | `aNCA` | `update_main_intervals` | [cases/case_051_aNCA_update_main_intervals.md](cases/case_051_aNCA_update_main_intervals.md) |
| 052 | `aNCA` | `validate_pk` | [cases/case_052_aNCA_validate_pk.md](cases/case_052_aNCA_validate_pk.md) |
| 053 | `admiral` | `adjust_last_day_imputation` | [cases/case_053_admiral_adjust_last_day_imputation.md](cases/case_053_admiral_adjust_last_day_imputation.md) |
| 054 | `admiral` | `calculate_range_value` | [cases/case_054_admiral_calculate_range_value.md](cases/case_054_admiral_calculate_range_value.md) |
| 055 | `admiral` | `call_derivation` | [cases/case_055_admiral_call_derivation.md](cases/case_055_admiral_call_derivation.md) |
| 056 | `admiral` | `censor_source` | [cases/case_056_admiral_censor_source.md](cases/case_056_admiral_censor_source.md) |
| 057 | `admiral` | `chr2vars` | [cases/case_057_admiral_chr2vars.md](cases/case_057_admiral_chr2vars.md) |
| 058 | `admiral` | `compute_age_years` | [cases/case_058_admiral_compute_age_years.md](cases/case_058_admiral_compute_age_years.md) |
| 059 | `admiral` | `compute_bmi` | [cases/case_059_admiral_compute_bmi.md](cases/case_059_admiral_compute_bmi.md) |
| 060 | `admiral` | `compute_bsa` | [cases/case_060_admiral_compute_bsa.md](cases/case_060_admiral_compute_bsa.md) |
| 061 | `admiral` | `compute_dtf` | [cases/case_061_admiral_compute_dtf.md](cases/case_061_admiral_compute_dtf.md) |
| 062 | `admiral` | `compute_duration` | [cases/case_062_admiral_compute_duration.md](cases/case_062_admiral_compute_duration.md) |
| 063 | `admiral` | `compute_egfr` | [cases/case_063_admiral_compute_egfr.md](cases/case_063_admiral_compute_egfr.md) |
| 064 | `admiral` | `compute_framingham` | [cases/case_064_admiral_compute_framingham.md](cases/case_064_admiral_compute_framingham.md) |
| 065 | `admiral` | `compute_map` | [cases/case_065_admiral_compute_map.md](cases/case_065_admiral_compute_map.md) |
| 066 | `admiral` | `compute_qtc` | [cases/case_066_admiral_compute_qtc.md](cases/case_066_admiral_compute_qtc.md) |
| 067 | `admiral` | `compute_qual_imputation` | [cases/case_067_admiral_compute_qual_imputation.md](cases/case_067_admiral_compute_qual_imputation.md) |
| 068 | `admiral` | `compute_qual_imputation_dec` | [cases/case_068_admiral_compute_qual_imputation_dec.md](cases/case_068_admiral_compute_qual_imputation_dec.md) |
| 069 | `admiral` | `compute_rr` | [cases/case_069_admiral_compute_rr.md](cases/case_069_admiral_compute_rr.md) |
| 070 | `admiral` | `compute_scale` | [cases/case_070_admiral_compute_scale.md](cases/case_070_admiral_compute_scale.md) |
| 071 | `admiral` | `compute_tmf` | [cases/case_071_admiral_compute_tmf.md](cases/case_071_admiral_compute_tmf.md) |
| 072 | `admiral` | `convert_blanks_to_na` | [cases/case_072_admiral_convert_blanks_to_na.md](cases/case_072_admiral_convert_blanks_to_na.md) |
| 073 | `admiral` | `convert_date_to_dtm` | [cases/case_073_admiral_convert_date_to_dtm.md](cases/case_073_admiral_convert_date_to_dtm.md) |
| 074 | `admiral` | `convert_dtc_to_dt` | [cases/case_074_admiral_convert_dtc_to_dt.md](cases/case_074_admiral_convert_dtc_to_dt.md) |
| 075 | `admiral` | `convert_dtc_to_dtm` | [cases/case_075_admiral_convert_dtc_to_dtm.md](cases/case_075_admiral_convert_dtc_to_dtm.md) |
| 076 | `admiral` | `convert_min_after_start` | [cases/case_076_admiral_convert_min_after_start.md](cases/case_076_admiral_convert_min_after_start.md) |
| 077 | `admiral` | `convert_min_pre_eot` | [cases/case_077_admiral_convert_min_pre_eot.md](cases/case_077_admiral_convert_min_pre_eot.md) |
| 078 | `admiral` | `convert_na_to_blanks` | [cases/case_078_admiral_convert_na_to_blanks.md](cases/case_078_admiral_convert_na_to_blanks.md) |
| 079 | `admiral` | `convert_post_end_patterns` | [cases/case_079_admiral_convert_post_end_patterns.md](cases/case_079_admiral_convert_post_end_patterns.md) |
| 080 | `admiral` | `convert_predose_patterns` | [cases/case_080_admiral_convert_predose_patterns.md](cases/case_080_admiral_convert_predose_patterns.md) |
| 081 | `admiral` | `convert_ranges` | [cases/case_081_admiral_convert_ranges.md](cases/case_081_admiral_convert_ranges.md) |
| 082 | `admiral` | `convert_ranges_eot` | [cases/case_082_admiral_convert_ranges_eot.md](cases/case_082_admiral_convert_ranges_eot.md) |
| 083 | `admiral` | `convert_simple_units` | [cases/case_083_admiral_convert_simple_units.md](cases/case_083_admiral_convert_simple_units.md) |
| 084 | `admiral` | `convert_special_cases` | [cases/case_084_admiral_convert_special_cases.md](cases/case_084_admiral_convert_special_cases.md) |
| 085 | `admiral` | `convert_start_patterns` | [cases/case_085_admiral_convert_start_patterns.md](cases/case_085_admiral_convert_start_patterns.md) |
| 086 | `admiral` | `convert_time_units` | [cases/case_086_admiral_convert_time_units.md](cases/case_086_admiral_convert_time_units.md) |
| 087 | `admiral` | `convert_treatment_patterns` | [cases/case_087_admiral_convert_treatment_patterns.md](cases/case_087_admiral_convert_treatment_patterns.md) |
| 088 | `admiral` | `convert_xxtpt_to_hours` | [cases/case_088_admiral_convert_xxtpt_to_hours.md](cases/case_088_admiral_convert_xxtpt_to_hours.md) |
| 089 | `admiral` | `count_vals` | [cases/case_089_admiral_count_vals.md](cases/case_089_admiral_count_vals.md) |
| 090 | `admiral` | `create_period_dataset` | [cases/case_090_admiral_create_period_dataset.md](cases/case_090_admiral_create_period_dataset.md) |
| 091 | `admiral` | `date_source` | [cases/case_091_admiral_date_source.md](cases/case_091_admiral_date_source.md) |
| 092 | `admiral` | `default_qtc_paramcd` | [cases/case_092_admiral_default_qtc_paramcd.md](cases/case_092_admiral_default_qtc_paramcd.md) |
| 093 | `admiral` | `derive_basetype_records` | [cases/case_093_admiral_derive_basetype_records.md](cases/case_093_admiral_derive_basetype_records.md) |
| 094 | `admiral` | `derive_locf_records` | [cases/case_094_admiral_derive_locf_records.md](cases/case_094_admiral_derive_locf_records.md) |
| 095 | `admiral` | `derive_param_bmi` | [cases/case_095_admiral_derive_param_bmi.md](cases/case_095_admiral_derive_param_bmi.md) |
| 096 | `admiral` | `derive_param_bsa` | [cases/case_096_admiral_derive_param_bsa.md](cases/case_096_admiral_derive_param_bsa.md) |
| 097 | `admiral` | `derive_param_map` | [cases/case_097_admiral_derive_param_map.md](cases/case_097_admiral_derive_param_map.md) |
| 098 | `admiral` | `derive_param_qtc` | [cases/case_098_admiral_derive_param_qtc.md](cases/case_098_admiral_derive_param_qtc.md) |
| 099 | `admiral` | `derive_param_rr` | [cases/case_099_admiral_derive_param_rr.md](cases/case_099_admiral_derive_param_rr.md) |
| 100 | `admiral` | `derive_param_tte` | [cases/case_100_admiral_derive_param_tte.md](cases/case_100_admiral_derive_param_tte.md) |
| 101 | `admiral` | `derive_var_atoxgr_dir` | [cases/case_101_admiral_derive_var_atoxgr_dir.md](cases/case_101_admiral_derive_var_atoxgr_dir.md) |
| 102 | `admiral` | `derive_var_base` | [cases/case_102_admiral_derive_var_base.md](cases/case_102_admiral_derive_var_base.md) |
| 103 | `admiral` | `derive_var_ontrtfl` | [cases/case_103_admiral_derive_var_ontrtfl.md](cases/case_103_admiral_derive_var_ontrtfl.md) |
| 104 | `admiral` | `derive_var_pchg` | [cases/case_104_admiral_derive_var_pchg.md](cases/case_104_admiral_derive_var_pchg.md) |
| 105 | `admiral` | `derive_var_trtdurd` | [cases/case_105_admiral_derive_var_trtdurd.md](cases/case_105_admiral_derive_var_trtdurd.md) |
| 106 | `admiral` | `derive_var_trtemfl` | [cases/case_106_admiral_derive_var_trtemfl.md](cases/case_106_admiral_derive_var_trtemfl.md) |
| 107 | `admiral` | `derive_vars_aage` | [cases/case_107_admiral_derive_vars_aage.md](cases/case_107_admiral_derive_vars_aage.md) |
| 108 | `admiral` | `derive_vars_atc` | [cases/case_108_admiral_derive_vars_atc.md](cases/case_108_admiral_derive_vars_atc.md) |
| 109 | `admiral` | `derive_vars_cat` | [cases/case_109_admiral_derive_vars_cat.md](cases/case_109_admiral_derive_vars_cat.md) |
| 110 | `admiral` | `derive_vars_computed` | [cases/case_110_admiral_derive_vars_computed.md](cases/case_110_admiral_derive_vars_computed.md) |
| 111 | `admiral` | `derive_vars_crit_flag` | [cases/case_111_admiral_derive_vars_crit_flag.md](cases/case_111_admiral_derive_vars_crit_flag.md) |
| 112 | `admiral` | `derive_vars_dt` | [cases/case_112_admiral_derive_vars_dt.md](cases/case_112_admiral_derive_vars_dt.md) |
| 113 | `admiral` | `derive_vars_dtm` | [cases/case_113_admiral_derive_vars_dtm.md](cases/case_113_admiral_derive_vars_dtm.md) |
| 114 | `admiral` | `derive_vars_dtm_to_dt` | [cases/case_114_admiral_derive_vars_dtm_to_dt.md](cases/case_114_admiral_derive_vars_dtm_to_dt.md) |
| 115 | `admiral` | `derive_vars_dtm_to_tm` | [cases/case_115_admiral_derive_vars_dtm_to_tm.md](cases/case_115_admiral_derive_vars_dtm_to_tm.md) |
| 116 | `admiral` | `derive_vars_duration` | [cases/case_116_admiral_derive_vars_duration.md](cases/case_116_admiral_derive_vars_duration.md) |
| 117 | `admiral` | `derive_vars_dy` | [cases/case_117_admiral_derive_vars_dy.md](cases/case_117_admiral_derive_vars_dy.md) |
| 118 | `admiral` | `derive_vars_extreme_event` | [cases/case_118_admiral_derive_vars_extreme_event.md](cases/case_118_admiral_derive_vars_extreme_event.md) |
| 119 | `admiral` | `derive_vars_merged` | [cases/case_119_admiral_derive_vars_merged.md](cases/case_119_admiral_derive_vars_merged.md) |
| 120 | `admiral` | `derive_vars_merged_lookup` | [cases/case_120_admiral_derive_vars_merged_lookup.md](cases/case_120_admiral_derive_vars_merged_lookup.md) |
| 121 | `admiral` | `derive_vars_merged_summary` | [cases/case_121_admiral_derive_vars_merged_summary.md](cases/case_121_admiral_derive_vars_merged_summary.md) |
| 122 | `admiral` | `derive_vars_period` | [cases/case_122_admiral_derive_vars_period.md](cases/case_122_admiral_derive_vars_period.md) |
| 123 | `admiral` | `derive_vars_query` | [cases/case_123_admiral_derive_vars_query.md](cases/case_123_admiral_derive_vars_query.md) |
| 124 | `admiral` | `derive_vars_transposed` | [cases/case_124_admiral_derive_vars_transposed.md](cases/case_124_admiral_derive_vars_transposed.md) |
| 125 | `admiral` | `dt_level` | [cases/case_125_admiral_dt_level.md](cases/case_125_admiral_dt_level.md) |
| 126 | `admiral` | `dthcaus_source` | [cases/case_126_admiral_dthcaus_source.md](cases/case_126_admiral_dthcaus_source.md) |
| 127 | `admiral` | `dtm_level` | [cases/case_127_admiral_dtm_level.md](cases/case_127_admiral_dtm_level.md) |
| 128 | `admiral` | `event` | [cases/case_128_admiral_event.md](cases/case_128_admiral_event.md) |
| 129 | `admiral` | `event_joined` | [cases/case_129_admiral_event_joined.md](cases/case_129_admiral_event_joined.md) |
| 130 | `admiral` | `event_source` | [cases/case_130_admiral_event_source.md](cases/case_130_admiral_event_source.md) |
| 131 | `admiral` | `extend_source_datasets` | [cases/case_131_admiral_extend_source_datasets.md](cases/case_131_admiral_extend_source_datasets.md) |
| 132 | `admiral` | `extract_duplicate_records` | [cases/case_132_admiral_extract_duplicate_records.md](cases/case_132_admiral_extract_duplicate_records.md) |
| 133 | `admiral` | `extract_unit` | [cases/case_133_admiral_extract_unit.md](cases/case_133_admiral_extract_unit.md) |
| 134 | `admiral` | `filter_exist` | [cases/case_134_admiral_filter_exist.md](cases/case_134_admiral_filter_exist.md) |
| 135 | `admiral` | `filter_extreme` | [cases/case_135_admiral_filter_extreme.md](cases/case_135_admiral_filter_extreme.md) |
| 136 | `admiral` | `filter_not_exist` | [cases/case_136_admiral_filter_not_exist.md](cases/case_136_admiral_filter_not_exist.md) |
| 137 | `admiral` | `filter_relative` | [cases/case_137_admiral_filter_relative.md](cases/case_137_admiral_filter_relative.md) |
| 138 | `admiral` | `flag_event` | [cases/case_138_admiral_flag_event.md](cases/case_138_admiral_flag_event.md) |
| 139 | `admiral` | `format_imputed_dtc` | [cases/case_139_admiral_format_imputed_dtc.md](cases/case_139_admiral_format_imputed_dtc.md) |
| 140 | `admiral` | `get_admiral_option` | [cases/case_140_admiral_get_admiral_option.md](cases/case_140_admiral_get_admiral_option.md) |
| 141 | `admiral` | `get_dt_dtm_range` | [cases/case_141_admiral_get_dt_dtm_range.md](cases/case_141_admiral_get_dt_dtm_range.md) |
| 142 | `admiral` | `get_flagged_records` | [cases/case_142_admiral_get_flagged_records.md](cases/case_142_admiral_get_flagged_records.md) |
| 143 | `admiral` | `highest_impute_level` | [cases/case_143_admiral_highest_impute_level.md](cases/case_143_admiral_highest_impute_level.md) |
| 144 | `admiral` | `get_hori_data` | [cases/case_144_admiral_get_hori_data.md](cases/case_144_admiral_get_hori_data.md) |
| 145 | `admiral` | `impute_target_date` | [cases/case_145_admiral_impute_target_date.md](cases/case_145_admiral_impute_target_date.md) |
| 146 | `admiral` | `impute_target_time` | [cases/case_146_admiral_impute_target_time.md](cases/case_146_admiral_impute_target_time.md) |
| 147 | `admiral` | `get_imputation_targets` | [cases/case_147_admiral_get_imputation_targets.md](cases/case_147_admiral_get_imputation_targets.md) |
| 148 | `admiral` | `get_joined_sub_data` | [cases/case_148_admiral_get_joined_sub_data.md](cases/case_148_admiral_get_joined_sub_data.md) |
| 149 | `admiral` | `get_partialdatetime` | [cases/case_149_admiral_get_partialdatetime.md](cases/case_149_admiral_get_partialdatetime.md) |
| 150 | `admiral` | `get_summary_records` | [cases/case_150_admiral_get_summary_records.md](cases/case_150_admiral_get_summary_records.md) |
| 151 | `admiral` | `get_unified_time_unit` | [cases/case_151_admiral_get_unified_time_unit.md](cases/case_151_admiral_get_unified_time_unit.md) |
| 152 | `admiral` | `get_vars_query` | [cases/case_152_admiral_get_vars_query.md](cases/case_152_admiral_get_vars_query.md) |
| 153 | `admiral` | `impute_date_time` | [cases/case_153_admiral_impute_date_time.md](cases/case_153_admiral_impute_date_time.md) |
| 154 | `admiral` | `impute_dtc_dt` | [cases/case_154_admiral_impute_dtc_dt.md](cases/case_154_admiral_impute_dtc_dt.md) |
| 155 | `admiral` | `impute_dtc_dtm` | [cases/case_155_admiral_impute_dtc_dtm.md](cases/case_155_admiral_impute_dtc_dtm.md) |
| 156 | `admiral` | `is_partial_datetime` | [cases/case_156_admiral_is_partial_datetime.md](cases/case_156_admiral_is_partial_datetime.md) |
| 157 | `admiral` | `list_all_templates` | [cases/case_157_admiral_list_all_templates.md](cases/case_157_admiral_list_all_templates.md) |
| 158 | `admiral` | `list_tte_source_objects` | [cases/case_158_admiral_list_tte_source_objects.md](cases/case_158_admiral_list_tte_source_objects.md) |
| 159 | `admiral` | `print_named_list` | [cases/case_159_admiral_print_named_list.md](cases/case_159_admiral_print_named_list.md) |
| 160 | `admiral` | `propagate_na_values` | [cases/case_160_admiral_propagate_na_values.md](cases/case_160_admiral_propagate_na_values.md) |
| 161 | `admiral` | `restrict_dtc_dt` | [cases/case_161_admiral_restrict_dtc_dt.md](cases/case_161_admiral_restrict_dtc_dt.md) |
| 162 | `admiral` | `restrict_dtc_dtm` | [cases/case_162_admiral_restrict_dtc_dtm.md](cases/case_162_admiral_restrict_dtc_dtm.md) |
| 163 | `admiral` | `set_admiral_options` | [cases/case_163_admiral_set_admiral_options.md](cases/case_163_admiral_set_admiral_options.md) |
| 164 | `admiral` | `slice_derivation` | [cases/case_164_admiral_slice_derivation.md](cases/case_164_admiral_slice_derivation.md) |
| 165 | `admiral` | `yn_to_numeric` | [cases/case_165_admiral_yn_to_numeric.md](cases/case_165_admiral_yn_to_numeric.md) |
| 166 | `admiraldev` | `backquote` | [cases/case_166_admiraldev_backquote.md](cases/case_166_admiraldev_backquote.md) |
| 167 | `admiraldev` | `dquote` | [cases/case_167_admiraldev_dquote.md](cases/case_167_admiraldev_dquote.md) |
| 168 | `admiraldev` | `get_duplicates` | [cases/case_168_admiraldev_get_duplicates.md](cases/case_168_admiraldev_get_duplicates.md) |
| 169 | `admiraldev` | `get_source_vars` | [cases/case_169_admiraldev_get_source_vars.md](cases/case_169_admiraldev_get_source_vars.md) |
| 170 | `admiraldev` | `is_order_vars` | [cases/case_170_admiraldev_is_order_vars.md](cases/case_170_admiraldev_is_order_vars.md) |
| 171 | `admiraldev` | `is_valid_dtc` | [cases/case_171_admiraldev_is_valid_dtc.md](cases/case_171_admiraldev_is_valid_dtc.md) |
| 172 | `admiraldev` | `parse_code` | [cases/case_172_admiraldev_parse_code.md](cases/case_172_admiraldev_parse_code.md) |
| 173 | `admiraldev` | `process_set_values_to` | [cases/case_173_admiraldev_process_set_values_to.md](cases/case_173_admiraldev_process_set_values_to.md) |
| 174 | `admiraldev` | `replace_values_by_names` | [cases/case_174_admiraldev_replace_values_by_names.md](cases/case_174_admiraldev_replace_values_by_names.md) |
| 175 | `admiraldev` | `squote` | [cases/case_175_admiraldev_squote.md](cases/case_175_admiraldev_squote.md) |
| 176 | `admiraldev` | `suppress_warning` | [cases/case_176_admiraldev_suppress_warning.md](cases/case_176_admiraldev_suppress_warning.md) |
| 177 | `admiraldiscovery` | `admiral_pkg_versions` | [cases/case_177_admiraldiscovery_admiral_pkg_versions.md](cases/case_177_admiraldiscovery_admiral_pkg_versions.md) |
| 178 | `envsetup` | `detach_autos` | [cases/case_178_envsetup_detach_autos.md](cases/case_178_envsetup_detach_autos.md) |
| 179 | `ggsurvfit` | `scale_ggsurvfit` | [cases/case_179_ggsurvfit_scale_ggsurvfit.md](cases/case_179_ggsurvfit_scale_ggsurvfit.md) |
| 180 | `gridify` | `get_layouts` | [cases/case_180_gridify_get_layouts.md](cases/case_180_gridify_get_layouts.md) |
| 181 | `logrx` | `log_config` | [cases/case_181_logrx_log_config.md](cases/case_181_logrx_log_config.md) |
| 182 | `logrx` | `log_init` | [cases/case_182_logrx_log_init.md](cases/case_182_logrx_log_init.md) |
| 183 | `logrx` | `parse_log` | [cases/case_183_logrx_parse_log.md](cases/case_183_logrx_parse_log.md) |
| 184 | `logrx` | `reformat_subsections` | [cases/case_184_logrx_reformat_subsections.md](cases/case_184_logrx_reformat_subsections.md) |
| 185 | `metatools` | `add_labels` | [cases/case_185_metatools_add_labels.md](cases/case_185_metatools_add_labels.md) |
| 186 | `metatools` | `create_subgrps` | [cases/case_186_metatools_create_subgrps.md](cases/case_186_metatools_create_subgrps.md) |
| 187 | `metatools` | `dash_to_eq` | [cases/case_187_metatools_dash_to_eq.md](cases/case_187_metatools_dash_to_eq.md) |
| 188 | `metatools` | `remove_labels` | [cases/case_188_metatools_remove_labels.md](cases/case_188_metatools_remove_labels.md) |
| 189 | `metatools` | `validate_verbose` | [cases/case_189_metatools_validate_verbose.md](cases/case_189_metatools_validate_verbose.md) |
| 190 | `sdtm.oak` | `iso8601_sec` | [cases/case_190_sdtm.oak_iso8601_sec.md](cases/case_190_sdtm.oak_iso8601_sec.md) |
| 191 | `sdtm.oak` | `sbj_vars` | [cases/case_191_sdtm.oak_sbj_vars.md](cases/case_191_sdtm.oak_sbj_vars.md) |
| 192 | `sdtm.oak` | `str_to_anycase` | [cases/case_192_sdtm.oak_str_to_anycase.md](cases/case_192_sdtm.oak_str_to_anycase.md) |
| 193 | `sdtmchecks` | `fail` | [cases/case_193_sdtmchecks_fail.md](cases/case_193_sdtmchecks_fail.md) |
| 194 | `sdtmchecks` | `pass` | [cases/case_194_sdtmchecks_pass.md](cases/case_194_sdtmchecks_pass.md) |
| 195 | `tidytlg` | `add_indent` | [cases/case_195_tidytlg_add_indent.md](cases/case_195_tidytlg_add_indent.md) |
| 196 | `tidytlg` | `col_borders` | [cases/case_196_tidytlg_col_borders.md](cases/case_196_tidytlg_col_borders.md) |
| 197 | `tidytlg` | `replace_na_with_blank` | [cases/case_197_tidytlg_replace_na_with_blank.md](cases/case_197_tidytlg_replace_na_with_blank.md) |
