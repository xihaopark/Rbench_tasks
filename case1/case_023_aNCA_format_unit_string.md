# Case 023: pharmaverse/aNCA/format_unit_string

## Metadata
- Task ID: `pharmaverse/aNCA/format_unit_string`
- Package: `aNCA`
- Model: `openai/gpt-5.1`
- Status: `FAIL`
- Failure stage: `schema_mismatch`

## Pass/Error
```text
The generated output file schema did not match the ground truth schema.

Main signal:
result.csv: Shape mismatch: ref=(1, 2) vs llm=(3, 2).
```

## Prompt
```text
Write an R script to perform format unit string using the aNCA clinical task contract.

Input: data.tsv, unit_var.tsv
Output: result.csv


Required columns for result.csv: id, result
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `data.tsv`


```text
id	conc	concu
1	10.5	ng/mL
2	20.3	ng/mL
3	30.7	ng/mL
```

#### `unit_var.tsv`


```text
unit_var
concu
```

## Code
### Ground Truth Code

```r
has_aNCA <- requireNamespace("aNCA", quietly = TRUE)

suppressPackageStartupMessages(library(dplyr))

# 1. Read input data
data_path <- file.path("inputs", "data.tsv")
if (!file.exists(data_path)) {
  stop("data.tsv is required input")
}
data <- read.delim(data_path, check.names = FALSE, stringsAsFactors = FALSE)
unit_var_path <- file.path("inputs", "unit_var.tsv")
if (!file.exists(unit_var_path)) {
  stop("unit_var.tsv is required input")
}
unit_var_df <- read.delim(unit_var_path, check.names = FALSE, stringsAsFactors = FALSE)
unit_var <- if (ncol(unit_var_df) > 0 && nrow(unit_var_df) > 0) {
  unit_var_df[[1]][1]
} else {
  NULL
}

# 2. Validate data
# Check the basic data frame structure

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
  aNCA:::format_unit_string(data = data, unit_var = unit_var)
}, error = function(e) {
  # If the function is missing or fails, implement the logic manually
  if (is.null(unit_var) || !unit_var %in% names(data)) {
    ""
  } else {
    distinct_units <- unique(data[[unit_var]])
    if (length(distinct_units) == 1) {
      paste0(" (", distinct_units[1], ")")
    } else {
      ""
    }
  }
})

# 4. Create result dataframe
# Combine function results with input data when possible
if (is.vector(result) || is.numeric(result) || is.character(result)) {
  result_df <- data.frame(
    id = seq_along(result),
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
  library(readr)
  library(dplyr)
})

# Create outputs directory if it doesn't exist
if (!dir.exists("outputs")) {
  dir.create("outputs", recursive = TRUE)
}

# Read inputs
data_path <- file.path("inputs", "data.tsv")
unit_var_path <- file.path("inputs", "unit_var.tsv")

dat <- read_tsv(data_path, show_col_types = FALSE)
unit_var <- read_tsv(unit_var_path, show_col_types = FALSE)

# Get unit variable name(s)
unit_cols <- unit_var$unit_var

# Function to format unit string according to aNCA clinical task contract
format_unit <- function(x) {
  # Example implementation: normalize spacing around slash and per-symbol
  # Adjust as needed to fully comply with the contract specification.
  x <- trimws(x)
  x <- gsub("\\s*/\\s*", "/", x)
  x <- gsub("\\s+per\\s+", "/", x, ignore.case = TRUE)
  x
}

# Apply formatting
dat_formatted <- dat %>%
  mutate(across(all_of(unit_cols), format_unit))

# Prepare result: id + formatted unit string(s) collapsed if multiple
if (length(unit_cols) == 1) {
  res <- dat_formatted %>%
    transmute(
      id = .data[["id"]],
      result = .data[[unit_cols]]
    )
} else {
  res <- dat_formatted %>%
    mutate(result = do.call(
      paste,
      c(across(all_of(unit_cols)), sep = " ")
    )) %>%
    select(id, result)
}

# Write output
write_csv(res, file.path("outputs", "result.csv"), na = "")
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"id","result"
1," (ng/mL)"
```

### LLM Output
#### `result.csv`


```csv
id,result
1,ng/mL
2,ng/mL
3,ng/mL
```
