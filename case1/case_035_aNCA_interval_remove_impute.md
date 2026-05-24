# Case 035: pharmaverse/aNCA/interval_remove_impute

## Metadata
- Task ID: `pharmaverse/aNCA/interval_remove_impute`
- Package: `aNCA`
- Model: `openai/gpt-5.1`
- Status: `FAIL`
- Failure stage: `value_mismatch`

## Pass/Error
```text
The generated output files were produced, but one or more values differed from the
ground truth.

Main signal:
result.csv: Numeric missingness mismatch in column: value.
```

## Prompt
```text
Write an R script to perform interval remove impute using the aNCA clinical task contract.

Input: data.tsv
Output: result.csv


Required columns for result.csv: id, value, group, category
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `data.tsv`


```text
id	value	group	category
1	10.5	A	Type1
2	20.3	B	Type2
3	30.7	A	Type1
4	40.2	B	Type2
5	50.9	A	Type1
```

## Code
### Ground Truth Code

```r
has_aNCA <- requireNamespace("aNCA", quietly = TRUE)

# 1. Read input data
data_path <- file.path("inputs", "data.tsv")
if (!file.exists(data_path)) {
  stop("data.tsv is required input")
}
data <- read.delim(data_path, check.names = FALSE, stringsAsFactors = FALSE)

# 2. Validate data
# Check the basic data frame structure
for (df_name in c("data")) {
  df <- get(df_name)
  if (nrow(df) == 0) {
    stop(paste("Data frame", df_name, "is empty"))
  }
  if (ncol(df) == 0) {
    stop(paste("Data frame", df_name, "has no columns"))
  }
}

# 3. Execute function implementation
result <- tryCatch({
  if (!has_aNCA) stop("aNCA unavailable")
  aNCA:::interval_remove_impute(data = data)
}, error = function(e) {
  # If the function call fails, return the original data
  data
})

# 4. Create result dataframe
# Combine function results with input data when possible
if (is.vector(result) || is.numeric(result) || is.character(result)) {
  result_df <- data.frame(
    target_impute = target_impute,
    result = result,
    stringsAsFactors = FALSE)
  } else if (is.data.frame(result)) {
  result_df <- result
} else {
  result_df <- data.frame(result = result, stringsAsFactors = FALSE)
}

# 4. Generate summary statistics
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

# 5. Save outputs
outputs_dir <- "outputs"
dir.create(outputs_dir, showWarnings = FALSE)
unlink(file.path(outputs_dir, c("result.rds", "summary.csv")))
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

# Interval remove impute using the aNCA clinical task contract

suppressPackageStartupMessages({
  library(dplyr)
  library(readr)
})

# Ensure outputs directory exists
if (!dir.exists("outputs")) {
  dir.create("outputs", recursive = TRUE)
}

# Read input
input_path <- file.path("inputs", "data.tsv")
dat <- read_tsv(input_path, col_types = cols())

# ---- Interval remove impute logic ----
# Placeholder implementation:
# 1. Keep rows where value is within [quantile 0.05, 0.95] per group/category
# 2. Impute removed values by group/category mean (if any were removed)

dat_processed <- dat %>%
  group_by(group, category) %>%
  mutate(
    q_low  = quantile(value, 0.05, na.rm = TRUE),
    q_high = quantile(value, 0.95, na.rm = TRUE)
  ) %>%
  ungroup()

# Mark out-of-interval rows
to_remove <- with(dat_processed, value < q_low | value > q_high)

# Compute group/category means
group_means <- dat_processed %>%
  filter(!to_remove) %>%
  group_by(group, category) %>%
  summarise(mean_value = mean(value, na.rm = TRUE), .groups = "drop")

# Join means and impute
dat_imputed <- dat_processed %>%
  left_join(group_means, by = c("group", "category")) %>%
  mutate(
    value = ifelse(to_remove, mean_value, value)
  ) %>%
  select(id, value, group, category)

# Write output
output_path <- file.path("outputs", "result.csv")
write_csv(dat_imputed, output_path)
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"id","value","group","category"
1,10.5,"A","Type1"
2,20.3,"B","Type2"
3,30.7,"A","Type1"
4,40.2,"B","Type2"
5,50.9,"A","Type1"
```

### LLM Output
#### `result.csv`


```csv
id,value,group,category
1,30.7,A,Type1
2,NA,B,Type2
3,30.7,A,Type1
4,NA,B,Type2
5,30.7,A,Type1
```
