# Case 045: pharmaverse/aNCA/remove_impute_method

## Metadata
- Task ID: `pharmaverse/aNCA/remove_impute_method`
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

Write an R script for an R package task: perform remove impute method using the aNCA package. Use functions from the aNCA package when suitable.

Input: impute_vals.tsv, target_impute.tsv
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

read_single_column_tsv <- function(path, column_name) {
  lines <- readLines(path, warn = FALSE)
  if (length(lines) == 0L) {
    stop("Input file is empty: ", path)
  }

  header <- strsplit(lines[[1L]], "\t", fixed = TRUE)[[1L]]
  column_index <- match(column_name, header)
  if (is.na(column_index)) {
    stop("Required column not found: ", column_name)
  }

  if (length(lines) == 1L) {
    return(character())
  }

  values <- vapply(lines[-1L], function(line) {
    fields <- strsplit(line, "\t", fixed = TRUE)[[1L]]
    if (length(fields) < column_index) "" else fields[[column_index]]
  }, character(1L), USE.NAMES = FALSE)

  values[values == "NA"] <- NA_character_
  values
}

remove_impute_token <- function(value, target) {
  if (is.na(value)) {
    return(NA_character_)
  }
  if (identical(value, "")) {
    return("")
  }
  if (is.na(target) || identical(target, "")) {
    return(value)
  }

  has_comma <- grepl(",", value, fixed = TRUE)
  tokens <- if (has_comma) {
    strsplit(value, "\\s*,\\s*")[[1L]]
  } else {
    strsplit(trimws(value), "\\s+")[[1L]]
  }

  tokens <- tokens[tokens != target]
  if (length(tokens) == 0L) {
    return("")
  }

  paste(tokens, collapse = if (has_comma) "," else " ")
}

dir.create("outputs", showWarnings = FALSE, recursive = TRUE)

impute_vals <- read_single_column_tsv("inputs/impute_vals.tsv", "impute_vals")
target_impute <- read_single_column_tsv("inputs/target_impute.tsv", "target_impute")

target <- if (length(target_impute) == 0L) NA_character_ else target_impute[[1L]]
result <- vapply(impute_vals, remove_impute_token, character(1L), target = target)

output <- data.frame(
  impute_vals = impute_vals,
  result = result,
  stringsAsFactors = FALSE
)

write.csv(output, "outputs/result.csv", row.names = FALSE, na = "NA")
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

read_single_column_tsv <- function(path, column_name) {
  lines <- readLines(path, warn = FALSE)
  if (length(lines) == 0L) {
    stop("Input file is empty: ", path)
  }

  header <- strsplit(lines[[1L]], "\t", fixed = TRUE)[[1L]]
  column_index <- match(column_name, header)
  if (is.na(column_index)) {
    stop("Required column not found: ", column_name)
  }

  if (length(lines) == 1L) {
    return(character())
  }

  values <- vapply(lines[-1L], function(line) {
    fields <- strsplit(line, "\t", fixed = TRUE)[[1L]]
    if (length(fields) < column_index) "" else fields[[column_index]]
  }, character(1L), USE.NAMES = FALSE)

  values[values == "NA"] <- NA_character_
  values
}

remove_impute_token <- function(value, target) {
  if (is.na(value)) {
    return(NA_character_)
  }
  if (identical(value, "")) {
    return("")
  }
  if (is.na(target) || identical(target, "")) {
    return(value)
  }

  has_comma <- grepl(",", value, fixed = TRUE)
  tokens <- if (has_comma) {
    strsplit(value, "\\s*,\\s*")[[1L]]
  } else {
    strsplit(trimws(value), "\\s+")[[1L]]
  }

  tokens <- tokens[tokens != target]
  if (length(tokens) == 0L) {
    return("")
  }

  paste(tokens, collapse = if (has_comma) "," else " ")
}

dir.create("outputs", showWarnings = FALSE, recursive = TRUE)

impute_vals <- read_single_column_tsv("inputs/impute_vals.tsv", "impute_vals")
target_impute <- read_single_column_tsv("inputs/target_impute.tsv", "target_impute")

target <- if (length(target_impute) == 0L) NA_character_ else target_impute[[1L]]
result <- vapply(impute_vals, remove_impute_token, character(1L), target = target)

output <- data.frame(
  impute_vals = impute_vals,
  result = result,
  stringsAsFactors = FALSE
)

write.csv(output, "outputs/result.csv", row.names = FALSE, na = "NA")
```
