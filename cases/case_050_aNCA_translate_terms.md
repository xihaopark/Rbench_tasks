# Case 050: pharmaverse/aNCA/translate_terms

## Metadata
- Task ID: `pharmaverse/aNCA/translate_terms`
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
Write an R script to perform translate terms using the aNCA clinical task contract.

Input: input_terms.tsv, mapping_col.tsv, metadata.tsv, target_col.tsv
Output: result.csv


Required columns for result.csv: input_terms, mapping_col, result
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `input_terms.tsv`


```text
input_terms
AVAL
PARAMCD
```

#### `mapping_col.tsv`


```text
mapping_col
Variable
```

#### `metadata.tsv`


```text
Variable	Label
AVAL	Analysis Value
PARAMCD	Parameter Code
USUBJID	Subject ID
```

#### `target_col.tsv`


```text
target_col
Label
```

## Code
### Ground Truth Code

```r
has_aNCA <- requireNamespace("aNCA", quietly = TRUE)
suppressPackageStartupMessages(library(purrr))

# 1. Read input data
input_terms_path <- file.path("inputs", "input_terms.tsv")
if (!file.exists(input_terms_path)) {
  stop("input_terms.tsv is required input")
}
input_terms_df <- read.delim(input_terms_path, check.names = FALSE, stringsAsFactors = FALSE)
input_terms <- input_terms_df$input_terms
mapping_col_path <- file.path("inputs", "mapping_col.tsv")
if (!file.exists(mapping_col_path)) {
  stop("mapping_col.tsv is required input")
}
mapping_col_df <- read.delim(mapping_col_path, check.names = FALSE, stringsAsFactors = FALSE)
mapping_col <- mapping_col_df$mapping_col
target_col_path <- file.path("inputs", "target_col.tsv")
if (!file.exists(target_col_path)) {
  stop("target_col.tsv is required input")
}
target_col_df <- read.delim(target_col_path, check.names = FALSE, stringsAsFactors = FALSE)
target_col <- target_col_df$target_col
metadata_path <- file.path("inputs", "metadata.tsv")
if (!file.exists(metadata_path)) {
  stop("metadata.tsv is required input")
}
metadata <- read.delim(metadata_path, check.names = FALSE, stringsAsFactors = FALSE)

# 2. Validate data
# Check the basic data frame structure
for (df_name in c("metadata")) {
  df <- get(df_name)
  if (nrow(df) == 0) {
    stop(paste("Data frame", df_name, "is empty"))
  }
    if (ncol(df) == 0) {
      stop(paste("Data frame", df_name, "has no columns"))
    }
}

# 3. Execute function implementation
# Extract scalar parameters
if (is.data.frame(mapping_col) && ncol(mapping_col) > 0 && nrow(mapping_col) > 0) {
  mapping_col <- mapping_col[[1]][1]
} else if (is.character(mapping_col)) {
  mapping_col <- mapping_col[1]
}
if (is.data.frame(target_col) && ncol(target_col) > 0 && nrow(target_col) > 0) {
  target_col <- target_col[[1]][1]
} else if (is.character(target_col)) {
  target_col <- target_col[1]
}

result <- tryCatch({
  if (!has_aNCA) stop("aNCA unavailable")
  aNCA:::translate_terms(
    input_terms = input_terms,
    mapping_col = mapping_col,
    target_col = target_col,
    metadata = metadata
  )
}, error = function(e) {
  # If the function call fails, return NULL
  NULL
})


# 4. Create result dataframe
# Combine function results with input data when possible
if (is.vector(result) || is.numeric(result) || is.character(result)) {
  result_df <- data.frame(
    input_terms = input_terms, mapping_col = mapping_col,
    result = result,
    stringsAsFactors = FALSE)
  } else if (is.data.frame(result)) {
  result_df <- result
} else {
  result_df <- data.frame(result = result, stringsAsFactors = FALSE)
}
# 6. Save outputs
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
  library(readr)
  library(dplyr)
})

# Create outputs directory if it doesn't exist
if (!dir.exists("outputs")) {
  dir.create("outputs", recursive = TRUE)
}

# Read input files
input_terms <- read_tsv("inputs/input_terms.tsv", show_col_types = FALSE)
mapping_col <- read_tsv("inputs/mapping_col.tsv", show_col_types = FALSE)
metadata    <- read_tsv("inputs/metadata.tsv", show_col_types = FALSE)
target_col  <- read_tsv("inputs/target_col.tsv", show_col_types = FALSE)

# Extract column names
map_col_name    <- mapping_col$mapping_col[1]
target_col_name <- target_col$target_col[1]

# Check columns exist in metadata
if (!map_col_name %in% names(metadata)) {
  stop(paste("Mapping column", map_col_name, "not found in metadata"))
}
if (!target_col_name %in% names(metadata)) {
  stop(paste("Target column", target_col_name, "not found in metadata"))
}

# Perform translation
result <- input_terms %>%
  left_join(
    metadata %>%
      select(all_of(map_col_name), all_of(target_col_name)),
    by = setNames(map_col_name, "input_terms")
  ) %>%
  transmute(
    input_terms = input_terms,
    mapping_col = .data[[map_col_name]],
    result      = .data[[target_col_name]]
  )

# Write output
write_csv(result, "outputs/result.csv", na = "")
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"input_terms","mapping_col","result"
"AVAL","Variable","Analysis Value"
"PARAMCD","Variable","Parameter Code"
```

### LLM Output
No LLM output artifact was produced.
