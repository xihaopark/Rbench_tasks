# Case 004: pharmaverse/aNCA/add_impute_method

## Metadata
- Task ID: `pharmaverse/aNCA/add_impute_method`
- Package: `aNCA`
- Model: `openai/gpt-5.5`
- Status: `FAIL`
- Failure stage: `value_mismatch`

## Pass/Error
```text
The generated output files were produced, but one or more values differed from the ground truth.

Main signal:
result.csv: Value mismatch in column: result
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
suppressPackageStartupMessages({
  library(aNCA)
})

dir.create("outputs", showWarnings = FALSE, recursive = TRUE)

after_df <- read.delim("inputs/after.tsv", stringsAsFactors = FALSE, check.names = FALSE)
impute_vals_df <- read.delim("inputs/impute_vals.tsv", stringsAsFactors = FALSE, check.names = FALSE)
target_impute_df <- read.delim("inputs/target_impute.tsv", stringsAsFactors = FALSE, check.names = FALSE)

after_vals <- as.integer(after_df[["after"]])
target_vals <- as.character(target_impute_df[["target_impute"]])

insert_impute <- function(impute_val, after_pos, target_impute) {
  x <- if (is.na(impute_val) || impute_val == "") character(0) else strsplit(impute_val, ",", fixed = TRUE)[[1]]
  paste(append(x, target_impute, after = after_pos), collapse = ",")
}

n <- nrow(impute_vals_df)

if (length(after_vals) == 1L) {
  after_out <- rep(after_vals, n)
} else {
  after_out <- after_vals
}

if (length(target_vals) == 1L) {
  target_out <- rep(target_vals, n)
} else {
  target_out <- target_vals
}

result <- data.frame(
  impute_vals = impute_vals_df[["impute_vals"]],
  after = after_out,
  result = mapply(
    insert_impute,
    impute_vals_df[["impute_vals"]],
    after_out,
    target_out,
    USE.NAMES = FALSE
  ),
  stringsAsFactors = FALSE,
  check.names = FALSE
)

write.csv(result, "outputs/result.csv", row.names = FALSE, quote = TRUE)
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
"start_predose",1,"start_predose,start_conc0"
"start_predose,end_obs",1,"start_predose,start_conc0,end_obs"
"end_obs",1,"end_obs,start_conc0"
"start_conc0,end_obs",1,"start_conc0,start_conc0,end_obs"
```
