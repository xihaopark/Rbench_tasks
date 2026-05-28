# Case 098: pharmaverse/admiral/derive_param_qtc

## Metadata
- Task ID: `pharmaverse/admiral/derive_param_qtc`
- Package: `admiral`
- Model: `claude-code/claude-sonnet-4-6`
- Agent: `Claude Code`
- Status: `FAIL`
- Failure stage: `value_mismatch`
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

Write an R script for an R package task: perform derive param qtc using the admiral package. Use functions from the admiral package when suitable.

Input: dataset.tsv
Output: result.csv

Computation: Derive only QTCBR rows from paired QT and RR records using QT / sqrt(RR), preserving the expected identifier columns.


Required columns for result.csv: USUBJID, PARAMCD, AVAL
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.

## Input preview

### dataset.tsv
USUBJID	PARAMCD	AVAL
01	QT	400
01	RR	1000
02	QT	360
02	RR	800

Reference package function list:
The hidden reference solution's R package function calls are listed below. If the list is empty, the reference solution does not call package functions.
- package_functions: []
```

## Input
#### `dataset.tsv`

```text
USUBJID	PARAMCD	AVAL
01	QT	400
01	RR	1000
02	QT	360
02	RR	800
```

## Code
### Ground Truth Code

```r
dataset <- read.delim(file.path("inputs", "dataset.tsv"), check.names = FALSE, stringsAsFactors = FALSE)
wide <- reshape(dataset, idvar = "USUBJID", timevar = "PARAMCD", direction = "wide")
result_df <- data.frame(
  USUBJID = wide$USUBJID,
  PARAMCD = "QTCBR",
  AVAL = as.numeric(wide$AVAL.QT) / sqrt(as.numeric(wide$AVAL.RR) / 1000),
  stringsAsFactors = FALSE
)

dir.create("outputs", showWarnings = FALSE)
write.csv(result_df, file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
library(dplyr)

data <- read.delim("inputs/dataset.tsv", colClasses = c(USUBJID = "character"), stringsAsFactors = FALSE)

qt <- data[data$PARAMCD == "QT", c("USUBJID", "AVAL")]
rr <- data[data$PARAMCD == "RR", c("USUBJID", "AVAL")]

paired <- merge(qt, rr, by = "USUBJID", suffixes = c("_QT", "_RR"))

result <- data.frame(
  USUBJID = paired$USUBJID,
  PARAMCD = "QTCBR",
  AVAL    = paired$AVAL_QT / sqrt(paired$AVAL_RR),
  stringsAsFactors = FALSE
)

dir.create("outputs", showWarnings = FALSE)
write.csv(result, "outputs/result.csv", row.names = FALSE)
```

## Output
### Ground Truth Output
#### `result.csv`

```csv
"USUBJID","PARAMCD","AVAL"
1,"QTCBR",400
2,"QTCBR",402.492235949962
```

### LLM Output
#### `result.csv`

```csv
"USUBJID","PARAMCD","AVAL"
"01","QTCBR",12.6491106406735
"02","QTCBR",12.7279220613579
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

Attaching package: ‘dplyr’

The following objects are masked from ‘package:stats’:

    filter, lag

The following objects are masked from ‘package:base’:

    intersect, setdiff, setequal, union
```

## Evaluation Result
```json
{
  "status": "FAIL",
  "tier": "schema_ok",
  "failure_stage": "value_mismatch",
  "pass": false,
  "score": 0.0,
  "message": "Failed at case_embedded",
  "validation_diagnostics": {
    "failure_stage": "value_mismatch",
    "tier": "schema_ok",
    "case_count": 1,
    "failed_case": "case_embedded"
  }
}
```

## Raw Agent Response
```text
library(dplyr)

data <- read.delim("inputs/dataset.tsv", colClasses = c(USUBJID = "character"), stringsAsFactors = FALSE)

qt <- data[data$PARAMCD == "QT", c("USUBJID", "AVAL")]
rr <- data[data$PARAMCD == "RR", c("USUBJID", "AVAL")]

paired <- merge(qt, rr, by = "USUBJID", suffixes = c("_QT", "_RR"))

result <- data.frame(
  USUBJID = paired$USUBJID,
  PARAMCD = "QTCBR",
  AVAL    = paired$AVAL_QT / sqrt(paired$AVAL_RR),
  stringsAsFactors = FALSE
)

dir.create("outputs", showWarnings = FALSE)
write.csv(result, "outputs/result.csv", row.names = FALSE)
```
