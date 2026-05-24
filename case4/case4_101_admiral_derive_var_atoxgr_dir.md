# Case 101: pharmaverse/admiral/derive_var_atoxgr_dir

## Metadata
- Task ID: `pharmaverse/admiral/derive_var_atoxgr_dir`
- Package: `admiral`
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

Write an R script for an R package task: perform derive var atoxgr dir using the admiral package. Use functions from the admiral package when suitable.

Input: dataset.tsv
Output: result.csv

Computation: Set ATOXGRDIR to L when AVAL < ANRLO, H when AVAL > ANRHI, and N otherwise.


Required columns for result.csv: USUBJID, PARAMCD, AVAL, ANRLO, ANRHI, ATOXGRDIR
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `dataset.tsv`

```text
USUBJID	PARAMCD	AVAL	ANRLO	ANRHI
01	ALT	15	20	40
01	AST	55	10	35
02	BILI	0.8	0.2	1.2
03	CREAT	1.5	0.5	1.1
```

## Code
### Ground Truth Code

```r
result_df <- read.delim(file.path("inputs", "dataset.tsv"), check.names = FALSE, stringsAsFactors = FALSE)
result_df$ATOXGRDIR <- ifelse(
  result_df$AVAL < result_df$ANRLO,
  "L",
  ifelse(result_df$AVAL > result_df$ANRHI, "H", "N")
)

dir.create("outputs", showWarnings = FALSE)
write.csv(result_df, file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
suppressPackageStartupMessages({
  if (requireNamespace("admiral", quietly = TRUE)) {
    library(admiral)
  }
})

input_path <- file.path("inputs", "dataset.tsv")
output_dir <- "outputs"
output_path <- file.path(output_dir, "result.csv")

dataset <- read.delim(
  input_path,
  sep = "\t",
  header = TRUE,
  stringsAsFactors = FALSE,
  check.names = FALSE,
  colClasses = c(
    USUBJID = "character",
    PARAMCD = "character",
    AVAL = "numeric",
    ANRLO = "numeric",
    ANRHI = "numeric"
  )
)

required_vars <- c("USUBJID", "PARAMCD", "AVAL", "ANRLO", "ANRHI")
missing_vars <- setdiff(required_vars, names(dataset))
if (length(missing_vars) > 0) {
  stop("Missing required input columns: ", paste(missing_vars, collapse = ", "))
}

result <- dataset[required_vars]
result$ATOXGRDIR <- ifelse(
  is.na(result$AVAL) | is.na(result$ANRLO) | is.na(result$ANRHI),
  NA_character_,
  ifelse(
    result$AVAL < result$ANRLO,
    "L",
    ifelse(result$AVAL > result$ANRHI, "H", "N")
  )
)

dir.create(output_dir, showWarnings = FALSE, recursive = TRUE)
write.csv(result, output_path, row.names = FALSE, na = "")
```

## Output
### Ground Truth Output
#### `result.csv`

```csv
"USUBJID","PARAMCD","AVAL","ANRLO","ANRHI","ATOXGRDIR"
1,"ALT",15,20,40,"L"
1,"AST",55,10,35,"H"
2,"BILI",0.8,0.2,1.2,"N"
3,"CREAT",1.5,0.5,1.1,"H"
```

### LLM Output
#### `result.csv`

```csv
"USUBJID","PARAMCD","AVAL","ANRLO","ANRHI","ATOXGRDIR"
"01","ALT",15,20,40,"L"
"01","AST",55,10,35,"H"
"02","BILI",0.8,0.2,1.2,"N"
"03","CREAT",1.5,0.5,1.1,"H"
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
suppressPackageStartupMessages({
  if (requireNamespace("admiral", quietly = TRUE)) {
    library(admiral)
  }
})

input_path <- file.path("inputs", "dataset.tsv")
output_dir <- "outputs"
output_path <- file.path(output_dir, "result.csv")

dataset <- read.delim(
  input_path,
  sep = "\t",
  header = TRUE,
  stringsAsFactors = FALSE,
  check.names = FALSE,
  colClasses = c(
    USUBJID = "character",
    PARAMCD = "character",
    AVAL = "numeric",
    ANRLO = "numeric",
    ANRHI = "numeric"
  )
)

required_vars <- c("USUBJID", "PARAMCD", "AVAL", "ANRLO", "ANRHI")
missing_vars <- setdiff(required_vars, names(dataset))
if (length(missing_vars) > 0) {
  stop("Missing required input columns: ", paste(missing_vars, collapse = ", "))
}

result <- dataset[required_vars]
result$ATOXGRDIR <- ifelse(
  is.na(result$AVAL) | is.na(result$ANRLO) | is.na(result$ANRHI),
  NA_character_,
  ifelse(
    result$AVAL < result$ANRLO,
    "L",
    ifelse(result$AVAL > result$ANRHI, "H", "N")
  )
)

dir.create(output_dir, showWarnings = FALSE, recursive = TRUE)
write.csv(result, output_path, row.names = FALSE, na = "")
```
