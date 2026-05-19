# Case 044: pharmaverse/aNCA/process_data_mean

## Metadata
- Task ID: `pharmaverse/aNCA/process_data_mean`
- Package: `aNCA`
- Model: `openai/gpt-5.1`
- Status: `FAIL`
- Failure stage: `schema_mismatch`

## Pass/Error
```text
The generated output file schema did not match the ground truth schema.

Main signal:
result.csv: Shape mismatch: ref=(0, 1) vs llm=(2, 7).
```

## Prompt
```text
Write an R script to perform process data mean using the aNCA clinical task contract.

Input: extra_grouping_vars.tsv, facet_by.tsv, filtering_list.tsv, pknca_data.tsv, show_dose.tsv, use_time_since_last_dose.tsv, ylog_scale.tsv
Output: result.csv


Required columns for result.csv:
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `extra_grouping_vars.tsv`


```text
extra_grouping_vars
ARM
```

#### `facet_by.tsv`


```text
facet_by
PARAM
```

#### `filtering_list.tsv`


```text
variable	value
ARM	100 mg
```

#### `pknca_data.tsv`


```text
USUBJID	ARM	TIME	CONC	DOSE	PARAM	ROUTE	TSLD	BLQFL
101	100 mg	0	0.0	100	AUC	oral	0	Y
101	100 mg	1	34.2	100	AUC	oral	1	N
101	100 mg	2	61.5	100	AUC	oral	2	N
101	100 mg	4	38.1	100	AUC	oral	4	N
102	100 mg	0	0.0	100	AUC	oral	0	Y
102	100 mg	1	29.8	100	AUC	oral	1	N
102	100 mg	2	55.4	100	AUC	oral	2	N
102	100 mg	4	33.6	100	AUC	oral	4	N
```

#### `show_dose.tsv`


```text
show_dose
TRUE
```

#### `use_time_since_last_dose.tsv`


```text
use_time_since_last_dose
FALSE
```

#### `ylog_scale.tsv`


```text
ylog_scale
FALSE
```

## Code
### Ground Truth Code

```r
has_aNCA <- requireNamespace("aNCA", quietly = TRUE)

# 1. Read input data
pknca_data_path <- file.path("inputs", "pknca_data.tsv")
if (!file.exists(pknca_data_path)) {
  stop("pknca_data.tsv is required input")
}
pknca_data <- read.delim(pknca_data_path, check.names = FALSE, stringsAsFactors = FALSE)
extra_grouping_vars_path <- file.path("inputs", "extra_grouping_vars.tsv")
if (!file.exists(extra_grouping_vars_path)) {
  stop("extra_grouping_vars.tsv is required input")
}
extra_grouping_vars_df <- read.delim(extra_grouping_vars_path, check.names = FALSE, stringsAsFactors = FALSE)
extra_grouping_vars <- extra_grouping_vars_df$extra_grouping_vars
facet_by_path <- file.path("inputs", "facet_by.tsv")
if (!file.exists(facet_by_path)) {
  stop("facet_by.tsv is required input")
}
facet_by_df <- read.delim(facet_by_path, check.names = FALSE, stringsAsFactors = FALSE)
facet_by <- facet_by_df$facet_by
filtering_list_path <- file.path("inputs", "filtering_list.tsv")
if (!file.exists(filtering_list_path)) {
  stop("filtering_list.tsv is required input")
}
filtering_list <- read.delim(filtering_list_path, check.names = FALSE, stringsAsFactors = FALSE)
ylog_scale_path <- file.path("inputs", "ylog_scale.tsv")
if (!file.exists(ylog_scale_path)) {
  stop("ylog_scale.tsv is required input")
}
ylog_scale_df <- read.delim(ylog_scale_path, check.names = FALSE, stringsAsFactors = FALSE)
ylog_scale <- as.logical(ylog_scale_df$ylog_scale)
show_dose_path <- file.path("inputs", "show_dose.tsv")
if (!file.exists(show_dose_path)) {
  stop("show_dose.tsv is required input")
}
show_dose_df <- read.delim(show_dose_path, check.names = FALSE, stringsAsFactors = FALSE)
show_dose <- as.logical(show_dose_df$show_dose)
use_time_since_last_dose_path <- file.path("inputs", "use_time_since_last_dose.tsv")
if (!file.exists(use_time_since_last_dose_path)) {
  stop("use_time_since_last_dose.tsv is required input")
}
use_time_since_last_dose_df <- read.delim(use_time_since_last_dose_path, check.names = FALSE, stringsAsFactors = FALSE)
use_time_since_last_dose <- as.logical(use_time_since_last_dose_df$use_time_since_last_dose)

# 2. Validate data
# Check the basic data frame structure

# 2. Validate data
# Check the basic data frame structure
for (df_name in c("pknca_data", "filtering_list")) {
  if (exists(df_name)) {
    df <- get(df_name)
    if (is.data.frame(df)) {
      if (nrow(df) == 0) {
        stop(paste("Data frame", df_name, "is empty"))
      }
      if (ncol(df) == 0) {
        stop(paste("Data frame", df_name, "has no columns"))
      }
    }
  }
}

# 3. Execute function implementation
result <- tryCatch({
  if (!has_aNCA) stop("aNCA unavailable")
  aNCA:::process_data_mean(
    pknca_data = pknca_data,
    extra_grouping_vars = extra_grouping_vars,
    facet_by = facet_by,
    filtering_list = filtering_list,
    ylog_scale = ylog_scale,
    show_dose = show_dose,
    use_time_since_last_dose = use_time_since_last_dose
  )
}, error = function(e) {
  # If the function call fails, return NULL
  NULL
})


# 4. Create result dataframe
# Combine function results with input data when possible
if (is.vector(result) || is.numeric(result) || is.character(result)) {
  result_df <- data.frame(
    extra_grouping_vars = extra_grouping_vars, facet_by = facet_by,
    result = result,
    stringsAsFactors = FALSE)
  } else if (is.data.frame(result)) {
  result_df <- result
} else {
  result_df <- data.frame(result = result, stringsAsFactors = FALSE)
}

# 5. Generate summary statistics
if (is.data.frame(result_df)) {
  summary_stats <- data.frame(
    n_rows = nrow(result_df),
    n_cols = ncol(result_df),
    col_names = paste(colnames(result_df), collapse = ", "),
    stringsAsFactors = FALSE)
  } else if (is.vector(result) || is.numeric(result)) {
  summary_stats <- data.frame(
    n_values = length(result),
    mean_value = mean(result, na.rm = TRUE),
    min_value = min(result, na.rm = TRUE),
    max_value = max(result, na.rm = TRUE),
    median_value = median(result, na.rm = TRUE),
    stringsAsFactors = FALSE)
  } else {
  summary_stats <- data.frame(
    result_type = class(result)[1],
    stringsAsFactors = FALSE
  )
}

# 7. Save outputs
outputs_dir <- "outputs"
dir.create(outputs_dir, showWarnings = FALSE)
unlink(file.path(outputs_dir, "summary.csv"))
if (exists("result_df") && is.data.frame(result_df)) {
  write.csv(result_df, file.path(outputs_dir, "result.csv"), row.names = FALSE)
  } else if (exists("result")) {
  if (is.data.frame(result)) {
    write.csv(result, file.path(outputs_dir, "result.csv"), row.names = FALSE)
  } else if (is.list(result)) {
    saveRDS(result, file.path(outputs_dir, "result.rds"))
  } else if (is.vector(result) || is.numeric(result) || is.character(result)) {
    result_df <- data.frame(result = result, stringsAsFactors = FALSE)
    write.csv(result_df, file.path(outputs_dir, "result.csv"), row.names = FALSE)
  } else {
    saveRDS(result, file.path(outputs_dir, "result.rds"))
  }
} else {
  stop("No result variable found. Function implementation may not have returned a value.")
}
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

suppressPackageStartupMessages({
  library(dplyr)
  library(readr)
})

# Directories
in_dir  <- "inputs"
out_dir <- "outputs"

if (!dir.exists(out_dir)) dir.create(out_dir, recursive = TRUE)

# Read inputs
extra_grouping_vars <- read.delim(file.path(in_dir, "extra_grouping_vars.tsv"),
                                  stringsAsFactors = FALSE)$extra_grouping_vars
facet_by            <- read.delim(file.path(in_dir, "facet_by.tsv"),
                                  stringsAsFactors = FALSE)$facet_by
filtering_list      <- read.delim(file.path(in_dir, "filtering_list.tsv"),
                                  stringsAsFactors = FALSE)
pknca_data          <- read.delim(file.path(in_dir, "pknca_data.tsv"),
                                  stringsAsFactors = FALSE)
show_dose           <- read.delim(file.path(in_dir, "show_dose.tsv"),
                                  stringsAsFactors = FALSE)$show_dose[1]
use_tslast          <- read.delim(file.path(in_dir, "use_time_since_last_dose.tsv"),
                                  stringsAsFactors = FALSE)$use_time_since_last_dose[1]
ylog_scale          <- read.delim(file.path(in_dir, "ylog_scale.tsv"),
                                  stringsAsFactors = FALSE)$ylog_scale[1]

# Coerce logical-like single-value flags
to_logical <- function(x) {
  if (is.logical(x)) return(x)
  if (is.numeric(x)) return(x != 0)
  if (is.character(x)) return(tolower(x) %in% c("true", "t", "yes", "y", "1"))
  FALSE
}
show_dose  <- to_logical(show_dose)
use_tslast <- to_logical(use_tslast)
ylog_scale <- to_logical(ylog_scale)

# Apply filtering
if (nrow(filtering_list) > 0) {
  for (i in seq_len(nrow(filtering_list))) {
    var   <- filtering_list$variable[i]
    value <- filtering_list$value[i]
    if (var %in% names(pknca_data)) {
      pknca_data <- pknca_data %>% filter(.data[[var]] == value)
    }
  }
}

# Determine grouping variables
group_vars <- c("USUBJID")
group_vars <- unique(c(group_vars, extra_grouping_vars))
group_vars <- group_vars[group_vars %in% names(pknca_data)]

# Summary: mean concentration per group and facet
all_group_vars <- unique(c(group_vars, facet_by))
all_group_vars <- all_group_vars[all_group_vars %in% names(pknca_data)]

result <- pknca_data %>%
  group_by(across(all_of(all_group_vars))) %>%
  summarise(
    MEAN_CONC = mean(CONC, na.rm = TRUE),
    .groups   = "drop"
  )

# Add settings/flags columns for traceability
result <- result %>%
  mutate(
    SHOW_DOSE                = show_dose,
    USE_TIME_SINCE_LAST_DOSE = use_tslast,
    YLOG_SCALE               = ylog_scale
  )

# Write output
write.csv(result, file.path(out_dir, "result.csv"), row.names = FALSE, na = "")
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
""
```

### LLM Output
#### `result.csv`


```csv
"USUBJID","ARM","PARAM","MEAN_CONC","SHOW_DOSE","USE_TIME_SINCE_LAST_DOSE","YLOG_SCALE"
101,"100 mg","AUC",33.45,TRUE,FALSE,FALSE
102,"100 mg","AUC",29.7,TRUE,FALSE,FALSE
```
