# Case 123: pharmaverse/admiral/derive_vars_query

## Metadata
- Task ID: `pharmaverse/admiral/derive_vars_query`
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

Write an R script for an R package task: perform derive vars query using the admiral package. Use functions from the admiral package when suitable.

Input: dataset.tsv, dataset_queries.tsv
Output: result.csv

Computation: Evaluate each condition from dataset_queries.tsv against every record and create the corresponding Y/N flag column.


Required columns for result.csv: USUBJID, AETERM, AEREL, AESEV, CQ01FL, CQ02FL
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `dataset.tsv`

```text
USUBJID	AETERM	AEREL	AESEV
01	HEADACHE	RELATED	MILD
02	NAUSEA	NOT RELATED	MODERATE
03	HEADACHE	RELATED	SEVERE
```

#### `dataset_queries.tsv`

```text
QUERY_ID	SRCVAR	TERMCHAR	NEWVAR
Q1	AETERM	HEADACHE	CQ01FL
Q2	AEREL	RELATED	CQ02FL
```

## Code
### Ground Truth Code

```r
result_df <- read.delim(file.path("inputs", "dataset.tsv"), check.names = FALSE, stringsAsFactors = FALSE)
queries <- read.delim(file.path("inputs", "dataset_queries.tsv"), check.names = FALSE, stringsAsFactors = FALSE)
for (i in seq_len(nrow(queries))) {
  src <- queries$SRCVAR[i]
  new_var <- queries$NEWVAR[i]
  result_df[[new_var]] <- ifelse(result_df[[src]] == queries$TERMCHAR[i], "Y", "N")
}

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

queries <- read.delim(
  file = file.path("inputs", "dataset_queries.tsv"),
  sep = "\t",
  header = TRUE,
  stringsAsFactors = FALSE,
  colClasses = "character",
  check.names = FALSE
)

for (idx in seq_len(nrow(queries))) {
  src_var <- queries$SRCVAR[idx]
  term <- queries$TERMCHAR[idx]
  new_var <- queries$NEWVAR[idx]

  if (!src_var %in% names(dataset)) {
    stop(sprintf("Source variable '%s' is not present in dataset.tsv", src_var))
  }

  dataset[[new_var]] <- ifelse(!is.na(dataset[[src_var]]) & dataset[[src_var]] == term, "Y", "N")
}

required_cols <- c("USUBJID", "AETERM", "AEREL", "AESEV", "CQ01FL", "CQ02FL")
result <- dataset[, required_cols, drop = FALSE]

dir.create("outputs", showWarnings = FALSE, recursive = TRUE)
write.csv(result, file = file.path("outputs", "result.csv"), row.names = FALSE)
```

## Output
### Ground Truth Output
#### `result.csv`

```csv
"USUBJID","AETERM","AEREL","AESEV","CQ01FL","CQ02FL"
1,"HEADACHE","RELATED","MILD","Y","Y"
2,"NAUSEA","NOT RELATED","MODERATE","N","N"
3,"HEADACHE","RELATED","SEVERE","Y","Y"
```

### LLM Output
#### `result.csv`

```csv
"USUBJID","AETERM","AEREL","AESEV","CQ01FL","CQ02FL"
"01","HEADACHE","RELATED","MILD","Y","Y"
"02","NAUSEA","NOT RELATED","MODERATE","N","N"
"03","HEADACHE","RELATED","SEVERE","Y","Y"
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

dataset <- read.delim(
  file = file.path("inputs", "dataset.tsv"),
  sep = "\t",
  header = TRUE,
  stringsAsFactors = FALSE,
  colClasses = "character",
  check.names = FALSE
)

queries <- read.delim(
  file = file.path("inputs", "dataset_queries.tsv"),
  sep = "\t",
  header = TRUE,
  stringsAsFactors = FALSE,
  colClasses = "character",
  check.names = FALSE
)

for (idx in seq_len(nrow(queries))) {
  src_var <- queries$SRCVAR[idx]
  term <- queries$TERMCHAR[idx]
  new_var <- queries$NEWVAR[idx]

  if (!src_var %in% names(dataset)) {
    stop(sprintf("Source variable '%s' is not present in dataset.tsv", src_var))
  }

  dataset[[new_var]] <- ifelse(!is.na(dataset[[src_var]]) & dataset[[src_var]] == term, "Y", "N")
}

required_cols <- c("USUBJID", "AETERM", "AEREL", "AESEV", "CQ01FL", "CQ02FL")
result <- dataset[, required_cols, drop = FALSE]

dir.create("outputs", showWarnings = FALSE, recursive = TRUE)
write.csv(result, file = file.path("outputs", "result.csv"), row.names = FALSE)
```
