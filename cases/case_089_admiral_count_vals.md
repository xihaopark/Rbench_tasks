# Case 089: pharmaverse/admiral/count_vals

## Metadata
- Task ID: `pharmaverse/admiral/count_vals`
- Package: `admiral`
- Model: `openai/gpt-5.1`
- Status: `PASS`
- Failure stage: `pass`

## Pass/Error
```text
pass
```

## Prompt
```text
Write an R script to perform count vals using the admiral clinical task contract.

Input: val.tsv, var.tsv
Output: result.csv


Required columns for result.csv: count
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `val.tsv`


```text
val
"Y"
```

#### `var.tsv`


```text
var
"Y"
"N"
"Y"
"Y"
"N"
```

## Code
### Ground Truth Code

```r
suppressPackageStartupMessages(library(admiral))

# 1. Read input data
var_path <- file.path("inputs", "var.tsv")
if (!file.exists(var_path)) {
  stop("var.tsv is required input")
}
var_df <- read.delim(var_path, check.names = FALSE, stringsAsFactors = FALSE)
var <- var_df[[1]]

val_path <- file.path("inputs", "val.tsv")
if (!file.exists(val_path)) {
  stop("val.tsv is required input")
}
val_df <- read.delim(val_path, check.names = FALSE, stringsAsFactors = FALSE)
val <- val_df[[1]][1]

# 2. Execute function implementation
result <- admiral::count_vals(var, val)

# 3. Create result and save
outputs_dir <- "outputs"
dir.create(outputs_dir, showWarnings = FALSE)
result_df <- data.frame(count = result, stringsAsFactors = FALSE)
write.csv(result_df, file.path("outputs", "result.csv"), row.names = FALSE)
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
val <- read_tsv(
  file = file.path("inputs", "val.tsv"),
  show_col_types = FALSE
)

var <- read_tsv(
  file = file.path("inputs", "var.tsv"),
  show_col_types = FALSE
)

# Perform count vals using admiral clinical task contract interpretation:
# Count how many rows in var match the value in val
target_val <- val$val[1]
count_res <- var %>%
  filter(var == target_val) %>%
  summarise(count = n())

# Write result
write_csv(count_res, file.path("outputs", "result.csv"))
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"count"
3
```

### LLM Output
#### `result.csv`


```csv
count
3
```
