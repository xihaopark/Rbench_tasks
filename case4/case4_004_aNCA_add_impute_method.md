# Case 004: pharmaverse/aNCA/add_impute_method

## Metadata
- Task ID: `pharmaverse/aNCA/add_impute_method`
- Package: `aNCA`
- Model: `codex/gpt-5.5`
- Agent: `Codex CLI`
- Status: `PASS`
- Failure stage: `pass`
- Pattern: ``

## Pass/Error
```text
PASS
```

## Prompt
```text
You are running one RBioBench clinical R task in an isolated worktree.

Your goal is to write a complete, reproducible R script at `solution.R`.

Rules:
- Read input files only from `inputs/` using relative paths.
- Write exactly the required output artifact(s): outputs/result.csv.
- Create `outputs/` if needed.
- You may inspect `task.json`, `TASK.md`, and input files.
- Do not modify `inputs/`, `task.json`, `AGENTS.md`, or hidden evaluator metadata.
- Do not use files outside this worktree.
- Do not commit changes.
- Before finishing, run `Rscript solution.R` if R is available and fix any concrete error.

Task prompt:

Write an R script for an R package task: perform add impute method using the aNCA package. Use functions from the aNCA package when suitable.

Input: after.tsv, impute_vals.tsv, target_impute.tsv
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

read_required_tsv <- function(path, required_col) {
  data <- read.delim(path, stringsAsFactors = FALSE, check.names = FALSE)
  if (!required_col %in% names(data)) {
    stop(sprintf("Missing required column '%s' in %s", required_col, path), call. = FALSE)
  }
  data
}

local_add_impute_method <- function(impute_vals, target_impute, after) {
  if (length(impute_vals) == 0) {
    return(impute_vals)
  }

  impute_vals <- ifelse(is.na(impute_vals), "", impute_vals)
  vapply(
    lapply(
      lapply(strsplit(impute_vals, split = "[ ,]+"), setdiff, target_impute),
      append,
      values = target_impute,
      after = after
    ),
    paste,
    collapse = ",",
    FUN.VALUE = ""
  )
}

impute_data <- read_required_tsv("inputs/impute_vals.tsv", "impute_vals")
target_data <- read_required_tsv("inputs/target_impute.tsv", "target_impute")
after_data <- read_required_tsv("inputs/after.tsv", "after")

if (nrow(target_data) != 1L) {
  stop("target_impute.tsv must contain exactly one target_impute value.", call. = FALSE)
}
if (nrow(after_data) != 1L) {
  stop("after.tsv must contain exactly one after value.", call. = FALSE)
}

impute_vals <- impute_data[["impute_vals"]]
target_impute <- target_data[["target_impute"]][1L]
after <- suppressWarnings(as.numeric(after_data[["after"]][1L]))

if (is.na(after)) {
  stop("after must be numeric.", call. = FALSE)
}
if (!is.character(target_impute) || is.na(target_impute) || identical(target_impute, "")) {
  stop("target_impute must be a non-empty character value.", call. = FALSE)
}

add_impute <- local_add_impute_method
if (requireNamespace("aNCA", quietly = TRUE)) {
  suppressPackageStartupMessages(library(aNCA))
  add_impute <- getFromNamespace("add_impute_method", "aNCA")
}

result <- add_impute(
  impute_vals = impute_vals,
  target_impute = target_impute,
  after = after
)

output <- data.frame(
  impute_vals = impute_vals,
  after = after,
  result = result,
  stringsAsFactors = FALSE,
  check.names = FALSE
)

dir.create("outputs", showWarnings = FALSE, recursive = TRUE)
write.csv(output, "outputs/result.csv", row.names = FALSE, na = "")
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
"start_conc0,end_obs",1,"end_obs,start_conc0"
```

#### `case_01/stderr.txt`

```text
[RBioBench Library Aliases] Library aliasing disabled (using stub layer)
[Admiraldev Stub] Created admiraldev namespace with 10 stub functions
[RBioBench Stub Layer] Loaded admiraldev stubs
[aNCA Stub] Created aNCA namespace with 57 stub functions
[RBioBench Stub Layer] Loaded aNCA stubs
[Logrx Stub] Created logrx namespace with 2 stub functions
[RBioBench Stub Layer] Loaded logrx stubs
[Sdtmchecks Stub] Created sdtmchecks namespace with 2 stub functions
[RBioBench Stub Layer] Loaded sdtmchecks stubs
[Other Stubs] Registered 5 stub functions from 5 packages
[RBioBench Stub Layer] Loaded other package stubs
[RBioBench Stub Layer] Registered attach hook for admiral
[Admiral Stub] Injected 40 functions into admiral namespace
[Admiral Stub] Injected 40 functions into admiral namespace
[RBioBench Stub Layer] Stubs registered in admiral namespace
[Admiral Stub] Injected 40 functions into admiral namespace
[Admiral Stub] Injected 40 functions into admiral namespace
[RBioBench Stub Layer] Stubs registered in admiral namespace
[RBioBench Stub Layer] .Rprofile loaded. Stubs will be auto-injected when admiral loads.
Registered S3 method overwritten by 'tern':
  method   from 
  tidy.glm broom
```

## Evaluation Result
```json
{
  "status": "PASS",
  "tier": "pass",
  "failure_stage": "pass",
  "pass": true,
  "score": 1.0,
  "message": "",
  "validation_diagnostics": {
    "failure_stage": "pass",
    "tier": "pass",
    "case_count": 1,
    "failed_case": null
  }
}
```

## Raw Agent Response
```text
#!/usr/bin/env Rscript

read_required_tsv <- function(path, required_col) {
  data <- read.delim(path, stringsAsFactors = FALSE, check.names = FALSE)
  if (!required_col %in% names(data)) {
    stop(sprintf("Missing required column '%s' in %s", required_col, path), call. = FALSE)
  }
  data
}

local_add_impute_method <- function(impute_vals, target_impute, after) {
  if (length(impute_vals) == 0) {
    return(impute_vals)
  }

  impute_vals <- ifelse(is.na(impute_vals), "", impute_vals)
  vapply(
    lapply(
      lapply(strsplit(impute_vals, split = "[ ,]+"), setdiff, target_impute),
      append,
      values = target_impute,
      after = after
    ),
    paste,
    collapse = ",",
    FUN.VALUE = ""
  )
}

impute_data <- read_required_tsv("inputs/impute_vals.tsv", "impute_vals")
target_data <- read_required_tsv("inputs/target_impute.tsv", "target_impute")
after_data <- read_required_tsv("inputs/after.tsv", "after")

if (nrow(target_data) != 1L) {
  stop("target_impute.tsv must contain exactly one target_impute value.", call. = FALSE)
}
if (nrow(after_data) != 1L) {
  stop("after.tsv must contain exactly one after value.", call. = FALSE)
}

impute_vals <- impute_data[["impute_vals"]]
target_impute <- target_data[["target_impute"]][1L]
after <- suppressWarnings(as.numeric(after_data[["after"]][1L]))

if (is.na(after)) {
  stop("after must be numeric.", call. = FALSE)
}
if (!is.character(target_impute) || is.na(target_impute) || identical(target_impute, "")) {
  stop("target_impute must be a non-empty character value.", call. = FALSE)
}

add_impute <- local_add_impute_method
if (requireNamespace("aNCA", quietly = TRUE)) {
  suppressPackageStartupMessages(library(aNCA))
  add_impute <- getFromNamespace("add_impute_method", "aNCA")
}

result <- add_impute(
  impute_vals = impute_vals,
  target_impute = target_impute,
  after = after
)

output <- data.frame(
  impute_vals = impute_vals,
  after = after,
  result = result,
  stringsAsFactors = FALSE,
  check.names = FALSE
)

dir.create("outputs", showWarnings = FALSE, recursive = TRUE)
write.csv(output, "outputs/result.csv", row.names = FALSE, na = "")
```
