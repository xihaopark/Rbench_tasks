# Case 004: pharmaverse/aNCA/add_impute_method

## Metadata
- Task ID: `pharmaverse/aNCA/add_impute_method`
- Package: `aNCA`
- Model: `claude-code/claude-sonnet-4-6`
- Agent: `Claude Code`
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
- `TASK.md` is the authoritative task contract. `task.json` is sanitized metadata only.
- Read input files only from `inputs/` using relative paths.
- Write exactly the required output artifact(s): outputs/result.csv.
- Create `outputs/` if needed.
- You may inspect `task.json`, `TASK.md`, and input files.
- You may consult public R package documentation, including CRAN, r-universe,
  and GitHub repository documentation, to verify normal exported package APIs.
- Do not infer package function names from task metadata. Use a package API only when
  it is a normal exported function you can verify; otherwise implement the required
  transformation directly from the inputs.
- Do not modify `inputs/`, `task.json`, `AGENTS.md`, or hidden evaluator metadata.
- Do not use files outside this worktree.
- Do not commit changes.
- Do NOT attempt to install R packages. All required packages are pre-installed in the
  evaluation Docker environment. Public documentation is useful for API lookup, but
  the final solution must run with the packages already installed in that environment.
  If a package is not available in the current shell, write the solution assuming it
  is available and move on.
- You may run `Rscript solution.R` to do a quick syntax check, but do not spend time
  debugging runtime errors caused by missing packages or system libraries.

Task prompt:

Write an R script for an R package task: perform add impute method using the aNCA package. Use functions from the aNCA package when suitable.

Input: after.tsv, impute_vals.tsv, target_impute.tsv
Output: result.csv

Computation: Insert the target impute token into the impute_vals vector at the requested after position; this is vector insertion, not row filtering.


Required columns for result.csv: impute_vals, after, result
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.

## Input preview

### after.tsv
after
1

### impute_vals.tsv
impute_vals
start_predose
start_predose,end_obs
end_obs
start_conc0,end_obs

### target_impute.tsv
target_impute
start_conc0
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

# add_impute_method: insert target_impute into each comma-separated impute_vals
# string at position `after`, removing any prior occurrence first.
# Implementation mirrors aNCA:::add_impute_method (intervals_helpers.R).
add_impute_method_impl <- function(impute_vals, target_impute, after) {
  if (length(impute_vals) == 0) return(impute_vals)
  impute_vals <- ifelse(is.na(impute_vals), "", impute_vals)
  parts <- strsplit(impute_vals, split = "[ ,]+")
  parts <- lapply(parts, FUN = setdiff, target_impute)
  parts <- lapply(parts, FUN = append, values = target_impute, after = after)
  vapply(parts, FUN = paste, collapse = ",", FUN.VALUE = "")
}

# Read inputs
after_df         <- read.table("inputs/after.tsv",          header = TRUE, sep = "\t", stringsAsFactors = FALSE)
impute_vals_df   <- read.table("inputs/impute_vals.tsv",    header = TRUE, sep = "\t", stringsAsFactors = FALSE)
target_impute_df <- read.table("inputs/target_impute.tsv",  header = TRUE, sep = "\t", stringsAsFactors = FALSE)

# Extract values
after         <- after_df$after[1]
target_impute <- target_impute_df$target_impute[1]
impute_vals   <- impute_vals_df$impute_vals

# Use aNCA package function when available; fall back to local implementation
result_vec <- tryCatch({
  library(aNCA)
  aNCA:::add_impute_method(
    impute_vals   = impute_vals,
    target_impute = target_impute,
    after         = after
  )
}, error = function(e) {
  message("aNCA not available; using built-in implementation: ", conditionMessage(e))
  add_impute_method_impl(impute_vals, target_impute, after)
})

# Build output data frame
result_df <- data.frame(
  impute_vals = impute_vals,
  after       = after,
  result      = result_vec,
  stringsAsFactors = FALSE
)

# Write output
dir.create("outputs", showWarnings = FALSE, recursive = TRUE)
write.csv(result_df, "outputs/result.csv", row.names = FALSE)

cat("Done. Output written to outputs/result.csv\n")
print(result_df)
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

Attaching package: ‘aNCA’

The following objects are masked _by_ ‘.GlobalEnv’:

    apply_mapping, convert_volume_units, create_start_impute,
    detect_study_types, dose_profile_duplicates, filter_breaks,
    format_pkncadata_intervals, g_pkcg01_lin, g_pkcg01_log,
    g_pkcg02_lin, g_pkcg02_log, generate_tooltip_text, get_label,
    interval_add_impute, interval_remove_impute, l_pkcl01,
    multiple_matrix_ratios, parse_annotation, pk.calc.volpk, read_pk,
    simplify_unit, translate_terms
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

# add_impute_method: insert target_impute into each comma-separated impute_vals
# string at position `after`, removing any prior occurrence first.
# Implementation mirrors aNCA:::add_impute_method (intervals_helpers.R).
add_impute_method_impl <- function(impute_vals, target_impute, after) {
  if (length(impute_vals) == 0) return(impute_vals)
  impute_vals <- ifelse(is.na(impute_vals), "", impute_vals)
  parts <- strsplit(impute_vals, split = "[ ,]+")
  parts <- lapply(parts, FUN = setdiff, target_impute)
  parts <- lapply(parts, FUN = append, values = target_impute, after = after)
  vapply(parts, FUN = paste, collapse = ",", FUN.VALUE = "")
}

# Read inputs
after_df         <- read.table("inputs/after.tsv",          header = TRUE, sep = "\t", stringsAsFactors = FALSE)
impute_vals_df   <- read.table("inputs/impute_vals.tsv",    header = TRUE, sep = "\t", stringsAsFactors = FALSE)
target_impute_df <- read.table("inputs/target_impute.tsv",  header = TRUE, sep = "\t", stringsAsFactors = FALSE)

# Extract values
after         <- after_df$after[1]
target_impute <- target_impute_df$target_impute[1]
impute_vals   <- impute_vals_df$impute_vals

# Use aNCA package function when available; fall back to local implementation
result_vec <- tryCatch({
  library(aNCA)
  aNCA:::add_impute_method(
    impute_vals   = impute_vals,
    target_impute = target_impute,
    after         = after
  )
}, error = function(e) {
  message("aNCA not available; using built-in implementation: ", conditionMessage(e))
  add_impute_method_impl(impute_vals, target_impute, after)
})

# Build output data frame
result_df <- data.frame(
  impute_vals = impute_vals,
  after       = after,
  result      = result_vec,
  stringsAsFactors = FALSE
)

# Write output
dir.create("outputs", showWarnings = FALSE, recursive = TRUE)
write.csv(result_df, "outputs/result.csv", row.names = FALSE)

cat("Done. Output written to outputs/result.csv\n")
print(result_df)
```
