# Case 142: pharmaverse/admiral/get_flagged_records

## Metadata
- Task ID: `pharmaverse/admiral/get_flagged_records`
- Package: `admiral`
- Model: `codex/gpt-5.5`
- Agent: `Codex CLI`
- Status: `FAIL`
- Failure stage: `schema_mismatch`
- Pattern: ``

## Pass/Error
```text
FAIL
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
- Do not infer package function names from task metadata. Use a package API only when
  it is a normal exported function you can verify; otherwise implement the required
  transformation directly from the inputs.
- Do not modify `inputs/`, `task.json`, `AGENTS.md`, or hidden evaluator metadata.
- Do not use files outside this worktree.
- Do not commit changes.
- Before finishing, run `Rscript solution.R` if R is available and fix any concrete error.

Task prompt:

Write an R script for an R package task: perform get flagged records using the admiral package. Use functions from the admiral package when suitable.

Input: dataset.tsv, new_var.tsv, threshold.tsv
Output: result.csv

Computation: Return only records satisfying AVAL > threshold and set the requested flag variable to Y in those returned rows.


Required columns for result.csv: USUBJID, PARAMCD, AVAL, AVISIT, HIGHFL
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `dataset.tsv`

```text
USUBJID	PARAMCD	AVAL	AVISIT
01	ALT	45	WEEK 1
01	ALT	18	WEEK 2
02	AST	52	WEEK 1
02	ALT	38	WEEK 1
```

#### `new_var.tsv`

```text
new_var
HIGHFL
```

#### `threshold.tsv`

```text
threshold
40
```

## Code
### Ground Truth Code

```r
dataset <- read.delim(file.path("inputs", "dataset.tsv"), check.names = FALSE, stringsAsFactors = FALSE)
threshold <- as.numeric(read.delim(file.path("inputs", "threshold.tsv"), check.names = FALSE, stringsAsFactors = FALSE)$threshold[1])
new_var <- as.character(read.delim(file.path("inputs", "new_var.tsv"), check.names = FALSE, stringsAsFactors = FALSE)$new_var[1])
result_df <- dataset[dataset$PARAMCD == "ALT" & dataset$AVAL > threshold, , drop = FALSE]
result_df[[new_var]] <- "Y"

dir.create("outputs", showWarnings = FALSE)
write.csv(result_df, file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

dataset <- read.delim(
  file = file.path("inputs", "dataset.tsv"),
  sep = "\t",
  header = TRUE,
  stringsAsFactors = FALSE,
  colClasses = "character",
  check.names = FALSE
)

new_var <- read.delim(
  file = file.path("inputs", "new_var.tsv"),
  sep = "\t",
  header = TRUE,
  stringsAsFactors = FALSE,
  check.names = FALSE
)[["new_var"]][1]

threshold <- read.delim(
  file = file.path("inputs", "threshold.tsv"),
  sep = "\t",
  header = TRUE,
  stringsAsFactors = FALSE,
  check.names = FALSE
)[["threshold"]][1]

threshold <- as.numeric(threshold)
dataset[["AVAL"]] <- as.numeric(dataset[["AVAL"]])

result <- dataset[dataset[["AVAL"]] > threshold, c("USUBJID", "PARAMCD", "AVAL", "AVISIT"), drop = FALSE]
result[[new_var]] <- "Y"

required_cols <- c("USUBJID", "PARAMCD", "AVAL", "AVISIT", new_var)
result <- result[, required_cols, drop = FALSE]

dir.create("outputs", showWarnings = FALSE, recursive = TRUE)
write.csv(result, file = file.path("outputs", "result.csv"), row.names = FALSE, quote = TRUE)
```

## Output
### Ground Truth Output
#### `result.csv`

```csv
"USUBJID","PARAMCD","AVAL","AVISIT","HIGHFL"
1,"ALT",45,"WEEK 1","Y"
```

### LLM Output
#### `result.csv`

```csv
"USUBJID","PARAMCD","AVAL","AVISIT","HIGHFL"
"01","ALT",45,"WEEK 1","Y"
"02","AST",52,"WEEK 1","Y"
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
  "status": "FAIL",
  "tier": "output_bad",
  "failure_stage": "schema_mismatch",
  "pass": false,
  "score": 0.0,
  "message": "Failed at case_embedded",
  "validation_diagnostics": {
    "failure_stage": "schema_mismatch",
    "tier": "output_bad",
    "case_count": 1,
    "failed_case": "case_embedded"
  }
}
```

## Raw Agent Response
```text
#!/usr/bin/env Rscript

dataset <- read.delim(
  file = file.path("inputs", "dataset.tsv"),
  sep = "\t",
  header = TRUE,
  stringsAsFactors = FALSE,
  colClasses = "character",
  check.names = FALSE
)

new_var <- read.delim(
  file = file.path("inputs", "new_var.tsv"),
  sep = "\t",
  header = TRUE,
  stringsAsFactors = FALSE,
  check.names = FALSE
)[["new_var"]][1]

threshold <- read.delim(
  file = file.path("inputs", "threshold.tsv"),
  sep = "\t",
  header = TRUE,
  stringsAsFactors = FALSE,
  check.names = FALSE
)[["threshold"]][1]

threshold <- as.numeric(threshold)
dataset[["AVAL"]] <- as.numeric(dataset[["AVAL"]])

result <- dataset[dataset[["AVAL"]] > threshold, c("USUBJID", "PARAMCD", "AVAL", "AVISIT"), drop = FALSE]
result[[new_var]] <- "Y"

required_cols <- c("USUBJID", "PARAMCD", "AVAL", "AVISIT", new_var)
result <- result[, required_cols, drop = FALSE]

dir.create("outputs", showWarnings = FALSE, recursive = TRUE)
write.csv(result, file = file.path("outputs", "result.csv"), row.names = FALSE, quote = TRUE)
```
