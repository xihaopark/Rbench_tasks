# Case 022: pharmaverse/aNCA/format_pkncadata_intervals

## Metadata
- Task ID: `pharmaverse/aNCA/format_pkncadata_intervals`
- Package: `aNCA`
- Model: `openai/gpt-5.1`
- Status: `NO_OUTPUT`
- Failure stage: `execution_failure`

## Pass/Error
```text
The generated code failed while running before valid outputs could be evaluated.

Main signal:
missing artifacts: result.csv.
```

## Prompt
```text
Write an R script to perform format pkncadata intervals using the aNCA clinical task contract.

Input: keep_interval_cols.tsv, pknca_conc.tsv, pknca_dose.tsv, start_from_last_dose.tsv
Output: result.csv


Required columns for result.csv: start, end
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `keep_interval_cols.tsv`


```text
keep_interval_cols
USUBJID
ARM
ROUTE
```

#### `pknca_conc.tsv`


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

#### `pknca_dose.tsv`


```text
USUBJID	TIME	DOSE	DOSEU	ROUTE
101	0	100	mg	oral
102	0	100	mg	oral
```

#### `start_from_last_dose.tsv`


```text
start_from_last_dose
FALSE
```

## Code
### Ground Truth Code

```r
has_aNCA <- requireNamespace("aNCA", quietly = TRUE)
suppressPackageStartupMessages(library(purrr))

# 1. Read input data
pknca_conc_path <- file.path("inputs", "pknca_conc.tsv")
if (!file.exists(pknca_conc_path)) {
  stop("pknca_conc.tsv is required input")
}
pknca_conc <- read.delim(pknca_conc_path, check.names = FALSE, stringsAsFactors = FALSE)
pknca_dose_path <- file.path("inputs", "pknca_dose.tsv")
if (!file.exists(pknca_dose_path)) {
  stop("pknca_dose.tsv is required input")
}
pknca_dose <- read.delim(pknca_dose_path, check.names = FALSE, stringsAsFactors = FALSE)
start_from_last_dose_path <- file.path("inputs", "start_from_last_dose.tsv")
if (!file.exists(start_from_last_dose_path)) {
  stop("start_from_last_dose.tsv is required input")
}
start_from_last_dose_df <- read.delim(start_from_last_dose_path, check.names = FALSE, stringsAsFactors = FALSE)
start_from_last_dose <- as.logical(start_from_last_dose_df$start_from_last_dose)
keep_interval_cols_path <- file.path("inputs", "keep_interval_cols.tsv")
if (!file.exists(keep_interval_cols_path)) {
  stop("keep_interval_cols.tsv is required input")
}
keep_interval_cols_df <- read.delim(keep_interval_cols_path, check.names = FALSE, stringsAsFactors = FALSE)
keep_interval_cols <- keep_interval_cols_df$keep_interval_cols

# 2. Validate data
# Check the basic data frame structure

# 2. Validate data
# Check the basic data frame structure
for (df_name in c("pknca_conc", "pknca_dose")) {
  df <- get(df_name)
  if (nrow(df) == 0) {
    stop(paste("Data frame", df_name, "is empty"))
  }
  if (ncol(df) == 0) {
    stop(paste("Data frame", df_name, "has no columns"))
  }
}

# 3. Execute function implementation
# Extract the keep_interval_cols parameter
if (is.data.frame(keep_interval_cols_df) && ncol(keep_interval_cols_df) > 0 && nrow(keep_interval_cols_df) > 0) {
  keep_interval_cols <- keep_interval_cols_df[[1]]
} else {
  keep_interval_cols <- NULL
}

# Create a simplified PKNCAdata object
has_PKNCA <- requireNamespace("PKNCA", quietly = TRUE)
pknca_conc_obj <- tryCatch({
  if (!has_PKNCA) stop("PKNCA unavailable")
  PKNCA::PKNCAdata(conc = pknca_conc, dose = pknca_dose)
}, error = function(e) {
  # If creation fails, use NULL
  NULL
})

pknca_dose_obj <- tryCatch({
  if (!has_PKNCA) stop("PKNCA unavailable")
  PKNCA::PKNCAdata(conc = pknca_conc, dose = pknca_dose)
}, error = function(e) {
  NULL
})

result <- tryCatch({
  if (!has_aNCA) stop("aNCA unavailable")
  aNCA:::format_pkncadata_intervals(
    pknca_conc = pknca_conc_obj,
    pknca_dose = pknca_dose_obj,
    start_from_last_dose = start_from_last_dose,
    keep_interval_cols = keep_interval_cols
  )
}, error = function(e) {
  # If the function call fails, return a default data frame
  data.frame(
    start = 0,
    end = Inf,
    stringsAsFactors = FALSE
  )
})


# 4. Create result dataframe
# Combine function results with input data when possible
if (is.vector(result) || is.numeric(result) || is.character(result)) {
  result_df <- data.frame(
    start_from_last_dose = start_from_last_dose, keep_interval_cols = keep_interval_cols,
    result = result,
    stringsAsFactors = FALSE)
  } else if (is.data.frame(result)) {
  result_df <- result
} else {
  result_df <- data.frame(result = result, stringsAsFactors = FALSE)
}
# 7. Save outputs
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

suppressPackageStartupMessages({
  library(PKNCA)
  library(dplyr)
  library(readr)
})

dir.create("outputs", showWarnings = FALSE, recursive = TRUE)

keep_interval_cols <- read_tsv(
  file = "inputs/keep_interval_cols.tsv",
  show_col_types = FALSE
) %>%
  pull(keep_interval_cols)

conc_df <- read_tsv(
  file = "inputs/pknca_conc.tsv",
  show_col_types = FALSE
)

dose_df <- read_tsv(
  file = "inputs/pknca_dose.tsv",
  show_col_types = FALSE
)

start_from_last_dose <- read_tsv(
  file = "inputs/start_from_last_dose.tsv",
  show_col_types = FALSE
) %>%
  pull(start_from_last_dose) %>%
  as.logical()

conc_obj <- PKNCAconc(
  data = conc_df,
  conc = "CONC",
  time = "TIME",
  subject = "USUBJID",
  route = "ROUTE"
)

dose_obj <- PKNCAdose(
  data = dose_df,
  dose = "DOSE",
  time = "TIME",
  subject = "USUBJID",
  route = "ROUTE"
)

data_obj <- PKNCAdata(
  conc = conc_obj,
  dose = dose_obj
)

intervals_df <- as.data.frame(attr(data_obj, "intervals"))

intervals_df <- intervals_df %>%
  mutate(
    start = if (isTRUE(start_from_last_dose)) {
      ifelse(.data$start == 0, NA_real_, .data$start)
    } else {
      .data$start
    }
  )

if (length(keep_interval_cols) > 0) {
  keep_cols <- intersect(keep_interval_cols, names(intervals_df))
  intervals_df <- intervals_df %>%
    select(all_of(keep_cols), start, end)
} else {
  intervals_df <- intervals_df %>%
    select(start, end)
}

result <- intervals_df %>%
  select(start, end)

write_csv(result, "outputs/result.csv", na = "")
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"start","end"
0,Inf
```

### LLM Output
No LLM output artifact was produced.
