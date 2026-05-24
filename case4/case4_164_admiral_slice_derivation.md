# Case 164: pharmaverse/admiral/slice_derivation

## Metadata
- Task ID: `pharmaverse/admiral/slice_derivation`
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

Write an R script for an R package task: perform slice derivation using the admiral package. Use functions from the admiral package when suitable.

Input: dataset.tsv, filter.tsv
Output: result.csv

Computation: Within each subject, use the filter condition to identify the first selected visit, then keep rows through that selected visit rather than only the matching row.


Required columns for result.csv: USUBJID, AVISITN, AVAL
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `dataset.tsv`

```text
USUBJID	AVISITN	AVAL
01	1	10
01	2	12
02	1	9
02	2	11
```

#### `filter.tsv`

```text
filter
AVISITN == 1
```

## Code
### Ground Truth Code

```r
suppressPackageStartupMessages(library(admiral))
suppressPackageStartupMessages(library(dplyr))
suppressPackageStartupMessages(library(rlang))

dataset <- read.delim(file.path("inputs", "dataset.tsv"), check.names = FALSE, stringsAsFactors = FALSE)
filter_expr <- read.delim(file.path("inputs", "filter.tsv"), check.names = FALSE, stringsAsFactors = FALSE)$filter[1]
slice <- admiral::derivation_slice(!!parse_expr(filter_expr))
result <- admiral::slice_derivation(dataset, derivation = dplyr::filter, slice)
dir.create("outputs", showWarnings = FALSE)
write.csv(result, file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

dataset <- read.delim(
  "inputs/dataset.tsv",
  sep = "\t",
  header = TRUE,
  stringsAsFactors = FALSE,
  colClasses = c(USUBJID = "character")
)

filter_spec <- read.delim(
  "inputs/filter.tsv",
  sep = "\t",
  header = TRUE,
  stringsAsFactors = FALSE
)

filter_condition <- filter_spec$filter[1]
selected <- eval(parse(text = filter_condition), envir = dataset, enclos = parent.frame())
selected[is.na(selected)] <- FALSE

first_selected_visit <- tapply(
  dataset$AVISITN[selected],
  dataset$USUBJID[selected],
  min
)

cutoff_visit <- first_selected_visit[dataset$USUBJID]
keep <- !is.na(cutoff_visit) & dataset$AVISITN <= cutoff_visit

result <- dataset[keep, c("USUBJID", "AVISITN", "AVAL"), drop = FALSE]

dir.create("outputs", showWarnings = FALSE, recursive = TRUE)
write.csv(result, "outputs/result.csv", row.names = FALSE, quote = FALSE)
```

## Output
### Ground Truth Output
#### `result.csv`

```csv
"USUBJID","AVISITN","AVAL"
1,1,10
2,1,9
1,2,12
2,2,11
```

### LLM Output
#### `result.csv`

```csv
USUBJID,AVISITN,AVAL
01,1,10
02,1,9
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
  "inputs/dataset.tsv",
  sep = "\t",
  header = TRUE,
  stringsAsFactors = FALSE,
  colClasses = c(USUBJID = "character")
)

filter_spec <- read.delim(
  "inputs/filter.tsv",
  sep = "\t",
  header = TRUE,
  stringsAsFactors = FALSE
)

filter_condition <- filter_spec$filter[1]
selected <- eval(parse(text = filter_condition), envir = dataset, enclos = parent.frame())
selected[is.na(selected)] <- FALSE

first_selected_visit <- tapply(
  dataset$AVISITN[selected],
  dataset$USUBJID[selected],
  min
)

cutoff_visit <- first_selected_visit[dataset$USUBJID]
keep <- !is.na(cutoff_visit) & dataset$AVISITN <= cutoff_visit

result <- dataset[keep, c("USUBJID", "AVISITN", "AVAL"), drop = FALSE]

dir.create("outputs", showWarnings = FALSE, recursive = TRUE)
write.csv(result, "outputs/result.csv", row.names = FALSE, quote = FALSE)
```
