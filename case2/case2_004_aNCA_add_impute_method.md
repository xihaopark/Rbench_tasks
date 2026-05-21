# Case 004: pharmaverse/aNCA/add_impute_method

## Metadata
- Task ID: `pharmaverse/aNCA/add_impute_method`
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
Write an R script for an R package task: perform add impute method using the aNCA package. The function(s) you use to solve the task must come from the aNCA package.

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
})

# Create outputs directory if it doesn't exist
if (!dir.exists("outputs")) dir.create("outputs", recursive = TRUE)

# Read inputs
after_df <- read.delim("inputs/after.tsv", stringsAsFactors = FALSE)
impute_df <- read.delim("inputs/impute_vals.tsv", stringsAsFactors = FALSE)
target_df <- read.delim("inputs/target_impute.tsv", stringsAsFactors = FALSE)

after_val   <- after_df$after[1]
impute_vec  <- impute_df$impute_vals
target_val  <- target_df$target_impute[1]

# Perform add impute method using aNCA
# aNCA::addImputeMethod(impute_vals, after, target_impute)
result_vec <- addImputeMethod(
  impute_vals   = impute_vec,
  after         = after_val,
  target_impute = target_val
)

# Prepare result
result_df <- data.frame(
  impute_vals = impute_vec,
  after       = after_val,
  result      = I(list(result_vec)),
  stringsAsFactors = FALSE
)

# Write result
write.csv(result_df, file = "outputs/result.csv", row.names = FALSE)
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
No LLM output artifact was produced.
