# Case 004: pharmaverse/aNCA/add_impute_method

## Metadata
- Task ID: `pharmaverse/aNCA/add_impute_method`
- Package: `aNCA`
- Model: `openai/gpt-5.1`
- Status: `FAIL`
- Failure stage: `schema_mismatch`

## Pass/Error
```text
The generated output file schema did not match the ground truth schema.

Main signal:
result.csv: Shape mismatch: ref=(4, 3) vs llm=(1, 3).
```

## Prompt
```text
Write an R script to perform add impute method using the aNCA clinical task contract.

Input: after.tsv, impute_vals.tsv, target_impute.tsv
Output: result.csv


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
  library(data.table)
})

# Create outputs directory if it doesn't exist
if (!dir.exists("outputs")) dir.create("outputs", recursive = TRUE)

# Read inputs
after_df <- fread("inputs/after.tsv", sep = "\t", header = TRUE)
impute_df <- fread("inputs/impute_vals.tsv", sep = "\t", header = TRUE)
target_df <- fread("inputs/target_impute.tsv", sep = "\t", header = TRUE)

# Ensure required columns exist
if (!("after" %in% names(after_df))) stop("after.tsv must contain column 'after'")
if (!("impute_vals" %in% names(impute_df))) stop("impute_vals.tsv must contain column 'impute_vals'")
if (!("target_impute" %in% names(target_df))) stop("target_impute.tsv must contain column 'target_impute'")

# 'Add impute' method: select rows from impute_vals that contain the target_impute token
target_token <- target_df$target_impute[1]

contains_token <- function(x, token) {
  # split on comma and check if token is one of the elements
  sapply(strsplit(x, ","), function(v) token %in% v)
}

matches <- impute_df[contains_token(impute_df$impute_vals, target_token), ]

# If no matches, result is empty
if (nrow(matches) == 0) {
  result_df <- data.table(
    impute_vals = character(0),
    after       = numeric(0),
    result      = character(0)
  )
} else {
  # Repeat 'after' value for each matching impute_vals
  after_val <- after_df$after[1]
  result_df <- data.table(
    impute_vals = matches$impute_vals,
    after       = after_val,
    result      = matches$impute_vals
  )
}

# Write output
fwrite(result_df, file = "outputs/result.csv", sep = ",", quote = TRUE, row.names = FALSE)
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
"start_conc0,end_obs",1,"start_conc0,end_obs"
```
