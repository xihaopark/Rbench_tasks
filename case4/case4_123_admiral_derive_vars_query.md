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
- Read input files only from `inputs/` using relative paths.
- Write exactly the required output artifact(s): outputs/result.csv.
- Create `outputs/` if needed.
- You may inspect `task.json`, `TASK.md`, and input files.
- Do not modify `inputs/`, `task.json`, `AGENTS.md`, or hidden evaluator metadata.
- Do not use files outside this worktree.
- Do not commit changes.
- Before finishing, run `Rscript solution.R` if R is available and fix any concrete error.

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

library(admiral)

read_tsv_chars <- function(path) {
  read.delim(
    file = path,
    sep = "\t",
    header = TRUE,
    stringsAsFactors = FALSE,
    colClasses = "character",
    check.names = FALSE,
    na.strings = c("", "NA")
  )
}

dataset <- read_tsv_chars(file.path("inputs", "dataset.tsv"))
dataset_queries <- read_tsv_chars(file.path("inputs", "dataset_queries.tsv"))

required_dataset_cols <- c("USUBJID", "AETERM", "AEREL", "AESEV")
required_query_cols <- c("SRCVAR", "TERMCHAR", "NEWVAR")

missing_dataset_cols <- setdiff(required_dataset_cols, names(dataset))
if (length(missing_dataset_cols) > 0) {
  stop("Missing required dataset columns: ", paste(missing_dataset_cols, collapse = ", "))
}

missing_query_cols <- setdiff(required_query_cols, names(dataset_queries))
if (length(missing_query_cols) > 0) {
  stop("Missing required query columns: ", paste(missing_query_cols, collapse = ", "))
}

for (i in seq_len(nrow(dataset_queries))) {
  srcvar <- dataset_queries$SRCVAR[i]
  term <- dataset_queries$TERMCHAR[i]
  newvar <- dataset_queries$NEWVAR[i]

  if (is.na(srcvar) || is.na(term) || is.na(newvar)) {
    stop("Query row ", i, " has missing SRCVAR, TERMCHAR, or NEWVAR.")
  }
  if (!srcvar %in% names(dataset)) {
    stop("Query row ", i, " references missing source variable: ", srcvar)
  }

  matches <- !is.na(dataset[[srcvar]]) & toupper(dataset[[srcvar]]) == toupper(term)

  if (newvar %in% names(dataset)) {
    dataset[[newvar]] <- ifelse(dataset[[newvar]] == "Y" | matches, "Y", "N")
  } else {
    dataset[[newvar]] <- ifelse(matches, "Y", "N")
  }
}

result_cols <- c(required_dataset_cols, unique(dataset_queries$NEWVAR))
result <- dataset[, result_cols, drop = FALSE]

dir.create("outputs", showWarnings = FALSE, recursive = TRUE)
write.csv(result, file = file.path("outputs", "result.csv"), row.names = FALSE, quote = FALSE, na = "")
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
USUBJID,AETERM,AEREL,AESEV,CQ01FL,CQ02FL
01,HEADACHE,RELATED,MILD,Y,Y
02,NAUSEA,NOT RELATED,MODERATE,N,N
03,HEADACHE,RELATED,SEVERE,Y,Y
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

library(admiral)

read_tsv_chars <- function(path) {
  read.delim(
    file = path,
    sep = "\t",
    header = TRUE,
    stringsAsFactors = FALSE,
    colClasses = "character",
    check.names = FALSE,
    na.strings = c("", "NA")
  )
}

dataset <- read_tsv_chars(file.path("inputs", "dataset.tsv"))
dataset_queries <- read_tsv_chars(file.path("inputs", "dataset_queries.tsv"))

required_dataset_cols <- c("USUBJID", "AETERM", "AEREL", "AESEV")
required_query_cols <- c("SRCVAR", "TERMCHAR", "NEWVAR")

missing_dataset_cols <- setdiff(required_dataset_cols, names(dataset))
if (length(missing_dataset_cols) > 0) {
  stop("Missing required dataset columns: ", paste(missing_dataset_cols, collapse = ", "))
}

missing_query_cols <- setdiff(required_query_cols, names(dataset_queries))
if (length(missing_query_cols) > 0) {
  stop("Missing required query columns: ", paste(missing_query_cols, collapse = ", "))
}

for (i in seq_len(nrow(dataset_queries))) {
  srcvar <- dataset_queries$SRCVAR[i]
  term <- dataset_queries$TERMCHAR[i]
  newvar <- dataset_queries$NEWVAR[i]

  if (is.na(srcvar) || is.na(term) || is.na(newvar)) {
    stop("Query row ", i, " has missing SRCVAR, TERMCHAR, or NEWVAR.")
  }
  if (!srcvar %in% names(dataset)) {
    stop("Query row ", i, " references missing source variable: ", srcvar)
  }

  matches <- !is.na(dataset[[srcvar]]) & toupper(dataset[[srcvar]]) == toupper(term)

  if (newvar %in% names(dataset)) {
    dataset[[newvar]] <- ifelse(dataset[[newvar]] == "Y" | matches, "Y", "N")
  } else {
    dataset[[newvar]] <- ifelse(matches, "Y", "N")
  }
}

result_cols <- c(required_dataset_cols, unique(dataset_queries$NEWVAR))
result <- dataset[, result_cols, drop = FALSE]

dir.create("outputs", showWarnings = FALSE, recursive = TRUE)
write.csv(result, file = file.path("outputs", "result.csv"), row.names = FALSE, quote = FALSE, na = "")
```
