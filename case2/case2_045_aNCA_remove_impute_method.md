# Case 045: pharmaverse/aNCA/remove_impute_method

## Metadata
- Task ID: `pharmaverse/aNCA/remove_impute_method`
- Package: `aNCA`
- Model: `openai/gpt-5.1`
- Status: `PASS`
- Failure stage: `pass`

## Pass/Error
```text
PASS
```

## Prompt
```text
Write an R script for an R package task: perform remove impute method using the aNCA package. Use functions from the aNCA package when suitable.

Output: result.csv

Computation: Remove the target token from comma- or space-separated imputation strings; do not filter rows. Preserve blanks as blank strings and missing values as missing where applicable.


Required columns for result.csv: impute_vals, result
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `impute_vals.tsv`

```text
impute_vals
start_predose,start_conc0
start_conc0,midpoint
start_conc0
```

#### `target_impute.tsv`

```text
target_impute
start_conc0
```

## Code
### Ground Truth Code

```r
impute_vals_df <- read.delim(file.path("inputs", "impute_vals.tsv"), check.names = FALSE, stringsAsFactors = FALSE)
target_impute_df <- read.delim(file.path("inputs", "target_impute.tsv"), check.names = FALSE, stringsAsFactors = FALSE)

impute_vals <- as.character(impute_vals_df[[1]])
target_impute <- as.character(target_impute_df[[1]][1])

remove_impute_method <- function(impute_vals, target_impute) {
  if (length(impute_vals) == 0) {
    return(impute_vals)
  }
  result <- vapply(impute_vals, function(value) {
    parts <- strsplit(ifelse(is.na(value), "", value), "[ ,]+")[[1]]
    parts <- parts[nzchar(parts)]
    paste(setdiff(parts, target_impute), collapse = ",")
  }, FUN.VALUE = character(1))
  ifelse(result == "", NA_character_, result)
}

result <- data.frame(
  impute_vals = impute_vals,
  result = remove_impute_method(impute_vals, target_impute),
  stringsAsFactors = FALSE
)

dir.create("outputs", showWarnings = FALSE, recursive = TRUE)
unlink(file.path("outputs", c("summary.csv", "result.rds")))
write.csv(result, file.path("outputs", "result.csv"), row.names = FALSE, na = "")
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

suppressPackageStartupMessages({
  library(aNCA)
})

# Create outputs directory if it does not exist
if (!dir.exists("outputs")) {
  dir.create("outputs", recursive = TRUE)
}

# Read inputs
impute_df <- read.delim(
  file = file.path("inputs", "impute_vals.tsv"),
  header = TRUE,
  sep = "\t",
  stringsAsFactors = FALSE,
  quote = "",
  na.strings = c("", "NA")
)

target_df <- read.delim(
  file = file.path("inputs", "target_impute.tsv"),
  header = TRUE,
  sep = "\t",
  stringsAsFactors = FALSE,
  quote = "",
  na.strings = c("", "NA")
)

target_token <- target_df$target_impute[1]

# Helper: remove target token from a single imputation string
remove_target_token <- function(x, target) {
  if (is.na(x)) return(NA_character_)
  if (!nzchar(x)) return("")
  
  # Determine separator: comma or space (prefer comma if present)
  sep <- if (grepl(",", x, fixed = TRUE)) "," else " "
  
  tokens <- strsplit(x, sep, fixed = TRUE)[[1]]
  tokens <- trimws(tokens)
  tokens <- tokens[tokens != target]
  
  if (!length(tokens)) {
    return("")
  }
  paste(tokens, collapse = sep)
}

# Apply token removal
result_vec <- vapply(
  impute_df$impute_vals,
  remove_target_token,
  character(1),
  target = target_token
)

# Assemble result dataframe
result_df <- data.frame(
  impute_vals = impute_df$impute_vals,
  result = result_vec,
  stringsAsFactors = FALSE
)

# Write result
write.csv(
  result_df,
  file = file.path("outputs", "result.csv"),
  row.names = FALSE,
  quote = TRUE
)
```

## Output
### Ground Truth Output
#### `result.csv`

```csv
"impute_vals","result"
"start_predose,start_conc0","start_predose"
"start_conc0,midpoint","midpoint"
"start_conc0",
```

### LLM Output
#### `result.csv`

```csv
"impute_vals","result"
"start_predose,start_conc0","start_predose"
"start_conc0,midpoint","midpoint"
"start_conc0",""
```
