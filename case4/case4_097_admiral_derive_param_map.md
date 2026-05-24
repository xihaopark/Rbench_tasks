# Case 097: pharmaverse/admiral/derive_param_map

## Metadata
- Task ID: `pharmaverse/admiral/derive_param_map`
- Package: `admiral`
- Model: `codex/gpt-5.5`
- Agent: `Codex CLI`
- Status: `NO_OUTPUT`
- Failure stage: `execution_failure`
- Pattern: ``

## Pass/Error
```text
NO_OUTPUT
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

Write an R script for an R package task: perform derive param map using the admiral package. Use functions from the admiral package when suitable.

Input: dataset.tsv
Output: result.csv

Computation: Derive only MAP rows from paired DIABP and SYSBP records using (2 * DIABP + SYSBP) / 3, and set PARAMCD to MAP.


Required columns for result.csv: USUBJID, PARAMCD, AVAL
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `dataset.tsv`

```text
USUBJID	PARAMCD	AVAL
01	SYSBP	120
01	DIABP	80
02	SYSBP	135
02	DIABP	85
```

## Code
### Ground Truth Code

```r
dataset <- read.delim(file.path("inputs", "dataset.tsv"), check.names = FALSE, stringsAsFactors = FALSE)
wide <- reshape(dataset, idvar = "USUBJID", timevar = "PARAMCD", direction = "wide")
result_df <- data.frame(
  USUBJID = wide$USUBJID,
  PARAMCD = "MAP",
  AVAL = (2 * as.numeric(wide$AVAL.DIABP) + as.numeric(wide$AVAL.SYSBP)) / 3,
  stringsAsFactors = FALSE
)

dir.create("outputs", showWarnings = FALSE)
write.csv(result_df, file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

library(admiral)
library(rlang)

input_path <- file.path("inputs", "dataset.tsv")
output_dir <- "outputs"
output_path <- file.path(output_dir, "result.csv")

dataset <- read.delim(
  input_path,
  sep = "\t",
  header = TRUE,
  stringsAsFactors = FALSE,
  colClasses = c(USUBJID = "character", PARAMCD = "character", AVAL = "numeric")
)

required_cols <- c("USUBJID", "PARAMCD", "AVAL")
missing_cols <- setdiff(required_cols, names(dataset))
if (length(missing_cols) > 0) {
  stop("Missing required input columns: ", paste(missing_cols, collapse = ", "))
}

result <- derive_param_map(
  dataset = dataset,
  by_vars = exprs(USUBJID),
  set_values_to = exprs(PARAMCD = "MAP"),
  sysbp_code = "SYSBP",
  diabp_code = "DIABP",
  get_unit_expr = "mmHg"
)

result <- result[result$PARAMCD == "MAP", required_cols]
result <- result[order(result$USUBJID), ]

dir.create(output_dir, showWarnings = FALSE, recursive = TRUE)
write.csv(result, output_path, row.names = FALSE)
```

## Output
### Ground Truth Output
#### `result.csv`

```csv
"USUBJID","PARAMCD","AVAL"
1,"MAP",93.3333333333333
2,"MAP",101.666666666667
```

### LLM Output
No LLM output artifact was produced.

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
Error in assert_parameters_argument(constant_parameters, optional = TRUE) : 
  unused argument (optional = TRUE)
Calls: derive_param_map -> withCallingHandlers -> derive_param_computed
Execution halted
```

## Evaluation Result
```json
{
  "status": "NO_OUTPUT",
  "tier": "exec_fail",
  "failure_stage": "execution_failure",
  "pass": false,
  "score": 0.0,
  "message": "Failed at case_embedded",
  "validation_diagnostics": {
    "failure_stage": "execution_failure",
    "tier": "exec_fail",
    "case_count": 1,
    "failed_case": "case_embedded"
  }
}
```

## Raw Agent Response
```text
#!/usr/bin/env Rscript

library(admiral)
library(rlang)

input_path <- file.path("inputs", "dataset.tsv")
output_dir <- "outputs"
output_path <- file.path(output_dir, "result.csv")

dataset <- read.delim(
  input_path,
  sep = "\t",
  header = TRUE,
  stringsAsFactors = FALSE,
  colClasses = c(USUBJID = "character", PARAMCD = "character", AVAL = "numeric")
)

required_cols <- c("USUBJID", "PARAMCD", "AVAL")
missing_cols <- setdiff(required_cols, names(dataset))
if (length(missing_cols) > 0) {
  stop("Missing required input columns: ", paste(missing_cols, collapse = ", "))
}

result <- derive_param_map(
  dataset = dataset,
  by_vars = exprs(USUBJID),
  set_values_to = exprs(PARAMCD = "MAP"),
  sysbp_code = "SYSBP",
  diabp_code = "DIABP",
  get_unit_expr = "mmHg"
)

result <- result[result$PARAMCD == "MAP", required_cols]
result <- result[order(result$USUBJID), ]

dir.create(output_dir, showWarnings = FALSE, recursive = TRUE)
write.csv(result, output_path, row.names = FALSE)
```
