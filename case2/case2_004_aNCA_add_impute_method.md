# Case 004: pharmaverse/aNCA/add_impute_method

## Metadata
- Task ID: `pharmaverse/aNCA/add_impute_method`
- Package: `aNCA`
- Model: `openai/gpt-5.1`
- Status: `FAIL`
- Failure stage: `schema_mismatch`

## Pass/Error
```text
The generated output files were produced, but the schema or output shape differed from the ground truth.

Main signal:
result.csv: Shape mismatch: ref=(4, 3) vs llm=(1, 3)
```

## Prompt
```text
Write an R script for an R package task: perform add impute method using the aNCA package. Use functions from the aNCA package when suitable.

Output: result.csv

Computation: Insert the target impute token into the impute_vals vector at the requested after position; this is vector insertion, not row filtering.


Required columns for result.csv: impute_vals, after, result
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `after.tsv`

```text
after
1
```

#### `impute_vals.tsv`

```text
impute_vals
start_predose
start_predose,end_obs
end_obs
start_conc0,end_obs
```

#### `target_impute.tsv`

```text
target_impute
start_conc0
```

## Code
### Ground Truth Code

```r
read_table <- function(name) {
  read.delim(file.path("inputs", name), check.names = FALSE, stringsAsFactors = FALSE)
}

first_value <- function(df, default = "") {
  if (nrow(df) == 0 || ncol(df) == 0 || is.na(df[[1]][[1]])) return(default)
  as.character(df[[1]][[1]])
}

insert_impute <- function(value, target, after) {
  parts <- unlist(strsplit(ifelse(is.na(value), "", as.character(value)), "[ ,]+"))
  parts <- parts[nzchar(parts) & parts != target]
  paste(append(parts, target, after = after), collapse = ",")
}

impute_vals <- read_table("impute_vals.tsv")[[1]]
target <- first_value(read_table("target_impute.tsv"))
after_raw <- suppressWarnings(as.integer(first_value(read_table("after.tsv"), "0")))
after <- ifelse(is.na(after_raw), 0L, max(0L, after_raw))
result_value <- vapply(impute_vals, insert_impute, character(1), target = target, after = after)
result <- data.frame(impute_vals = impute_vals, after = after, result = result_value, stringsAsFactors = FALSE)

dir.create("outputs", showWarnings = FALSE, recursive = TRUE)
write.csv(result, file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

suppressPackageStartupMessages({
  library(aNCA)
  library(utils)
})

# Create outputs directory if it does not exist
if (!dir.exists("outputs")) {
  dir.create("outputs", recursive = TRUE)
}

# Read inputs
after_df <- read.table("inputs/after.tsv", header = TRUE, sep = "\t", stringsAsFactors = FALSE)
impute_df <- read.table("inputs/impute_vals.tsv", header = TRUE, sep = "\t", stringsAsFactors = FALSE)
target_df <- read.table("inputs/target_impute.tsv", header = TRUE, sep = "\t", stringsAsFactors = FALSE)

after_val <- after_df$after[1]
impute_vec <- impute_df$impute_vals
target_token <- target_df$target_impute[1]

# Use aNCA-style insertion logic (vector insertion)
# If aNCA has a specific helper for impute method handling, use it; otherwise do base insertion
# Here we mimic an aNCA-compatible insertion of the token at the requested position

# Ensure position is within valid range
after_val <- max(0, min(after_val, length(impute_vec)))

# Perform insertion
if (after_val == 0) {
  result_vec <- c(target_token, impute_vec)
} else if (after_val == length(impute_vec)) {
  result_vec <- c(impute_vec, target_token)
} else {
  result_vec <- c(impute_vec[seq_len(after_val)],
                  target_token,
                  impute_vec[(after_val + 1):length(impute_vec)])
}

# Prepare result data.frame
result_df <- data.frame(
  impute_vals = paste(impute_vec, collapse = ";"),
  after       = after_val,
  result      = paste(result_vec, collapse = ";"),
  stringsAsFactors = FALSE
)

# Write output
write.csv(result_df, file = "outputs/result.csv", row.names = FALSE, quote = TRUE)
```

## Output
### Ground Truth Output
#### `result.csv`

```csv
"impute_vals","after","result"
"start_predose",1,"start_predose,start_conc0"
"start_predose,end_obs",1,"start_predose,start_conc0,end_obs"
"end_obs",1,"end_obs,start_conc0"
"start_conc0,end_obs",1,"end_obs,start_conc0"
```

### LLM Output
#### `result.csv`

```csv
"impute_vals","after","result"
"start_predose;start_predose,end_obs;end_obs;start_conc0,end_obs",1,"start_predose;start_conc0;start_predose,end_obs;end_obs;start_conc0,end_obs"
```
