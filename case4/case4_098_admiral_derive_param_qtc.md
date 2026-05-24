# Case 098: pharmaverse/admiral/derive_param_qtc

## Metadata
- Task ID: `pharmaverse/admiral/derive_param_qtc`
- Package: `admiral`
- Model: `codex/gpt-5.5`
- Agent: `Codex CLI`
- Agent command: `codex exec -C <worktree> --dangerously-bypass-approvals-and-sandbox <prompt> -m gpt-5.5`
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
- Read input files only from `inputs/` using relative paths.
- Write exactly the required output artifact(s): outputs/result.csv.
- Create `outputs/` if needed.
- You may inspect `task.json`, `TASK.md`, and input files.
- Do not modify `inputs/`, `task.json`, `AGENTS.md`, or hidden evaluator metadata.
- Do not use files outside this worktree.
- Do not commit changes.
- Before finishing, run `Rscript solution.R` if R is available and fix any concrete error.

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
#!/usr/bin/env Rscript

suppressPackageStartupMessages(library(admiral))

input_path <- file.path("inputs", "dataset.tsv")
output_dir <- "outputs"
output_path <- file.path(output_dir, "result.csv")

dataset <- read.delim(
  input_path,
  sep = "\t",
  header = TRUE,
  stringsAsFactors = FALSE,
  check.names = FALSE,
  colClasses = c(USUBJID = "character", PARAMCD = "character", AVAL = "numeric")
)

required_cols <- c("USUBJID", "PARAMCD", "AVAL")
missing_cols <- setdiff(required_cols, names(dataset))
if (length(missing_cols) > 0) {
  stop("Missing required input column(s): ", paste(missing_cols, collapse = ", "))
}

id_cols <- setdiff(names(dataset), c("PARAMCD", "AVAL"))

qt <- dataset[dataset$PARAMCD == "QT", c(id_cols, "AVAL"), drop = FALSE]
rr <- dataset[dataset$PARAMCD == "RR", c(id_cols, "AVAL"), drop = FALSE]
names(qt)[names(qt) == "AVAL"] <- "QT"
names(rr)[names(rr) == "AVAL"] <- "RR"

paired <- merge(qt, rr, by = id_cols, all = FALSE, sort = FALSE)

if (nrow(paired) == 0) {
  result <- data.frame(
    USUBJID = character(),
    PARAMCD = character(),
    AVAL = numeric(),
    stringsAsFactors = FALSE
  )
} else {
  if (any(is.na(paired$QT)) || any(is.na(paired$RR)) || any(paired$RR <= 0)) {
    stop("QT and RR pairs must have non-missing QT and positive RR values.")
  }

  result <- data.frame(
    USUBJID = paired$USUBJID,
    PARAMCD = "QTCBR",
    AVAL = paired$QT / sqrt(paired$RR),
    stringsAsFactors = FALSE
  )
}

dir.create(output_dir, showWarnings = FALSE, recursive = TRUE)
write.csv(result[, required_cols, drop = FALSE], output_path, row.names = FALSE)
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
#!/usr/bin/env Rscript

suppressPackageStartupMessages(library(admiral))

input_path <- file.path("inputs", "dataset.tsv")
output_dir <- "outputs"
output_path <- file.path(output_dir, "result.csv")

dataset <- read.delim(
  input_path,
  sep = "\t",
  header = TRUE,
  stringsAsFactors = FALSE,
  check.names = FALSE,
  colClasses = c(USUBJID = "character", PARAMCD = "character", AVAL = "numeric")
)

required_cols <- c("USUBJID", "PARAMCD", "AVAL")
missing_cols <- setdiff(required_cols, names(dataset))
if (length(missing_cols) > 0) {
  stop("Missing required input column(s): ", paste(missing_cols, collapse = ", "))
}

id_cols <- setdiff(names(dataset), c("PARAMCD", "AVAL"))

qt <- dataset[dataset$PARAMCD == "QT", c(id_cols, "AVAL"), drop = FALSE]
rr <- dataset[dataset$PARAMCD == "RR", c(id_cols, "AVAL"), drop = FALSE]
names(qt)[names(qt) == "AVAL"] <- "QT"
names(rr)[names(rr) == "AVAL"] <- "RR"

paired <- merge(qt, rr, by = id_cols, all = FALSE, sort = FALSE)

if (nrow(paired) == 0) {
  result <- data.frame(
    USUBJID = character(),
    PARAMCD = character(),
    AVAL = numeric(),
    stringsAsFactors = FALSE
  )
} else {
  if (any(is.na(paired$QT)) || any(is.na(paired$RR)) || any(paired$RR <= 0)) {
    stop("QT and RR pairs must have non-missing QT and positive RR values.")
  }

  result <- data.frame(
    USUBJID = paired$USUBJID,
    PARAMCD = "QTCBR",
    AVAL = paired$QT / sqrt(paired$RR),
    stringsAsFactors = FALSE
  )
}

dir.create(output_dir, showWarnings = FALSE, recursive = TRUE)
write.csv(result[, required_cols, drop = FALSE], output_path, row.names = FALSE)
```
