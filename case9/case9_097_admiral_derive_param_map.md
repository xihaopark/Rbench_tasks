# Case 097: pharmaverse/admiral/derive_param_map

## Metadata
- Task ID: `pharmaverse/admiral/derive_param_map`
- Package: `admiral`
- Model: `claude-code/claude-sonnet-4-6`
- Agent: `Claude Code`
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

Write an R script for an R package task: perform derive param map using the admiral package. Use functions from the admiral package when suitable.

Input: dataset.tsv
Output: result.csv

Computation: Derive only MAP rows from paired DIABP and SYSBP records using (2 * DIABP + SYSBP) / 3, and set PARAMCD to MAP.


Required columns for result.csv: USUBJID, PARAMCD, AVAL
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.

## Input preview

### dataset.tsv
USUBJID	PARAMCD	AVAL
01	SYSBP	120
01	DIABP	80
02	SYSBP	135
02	DIABP	85
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
library(dplyr)

# Read input dataset
dataset <- read.table("inputs/dataset.tsv", sep = "\t", header = TRUE,
                      colClasses = c(USUBJID = "character", PARAMCD = "character", AVAL = "numeric"))

# Derive MAP parameter using admiral's derive_param_computed
# MAP = (2 * DIABP + SYSBP) / 3
result <- derive_param_computed(
  dataset,
  by_vars    = exprs(USUBJID),
  parameters = c("SYSBP", "DIABP"),
  set_values_to = exprs(
    AVAL    = (2 * AVAL.DIABP + AVAL.SYSBP) / 3,
    PARAMCD = "MAP"
  )
)

# Keep only MAP rows with required columns
map_rows <- result %>%
  filter(PARAMCD == "MAP") %>%
  select(USUBJID, PARAMCD, AVAL)

# Write output
dir.create("outputs", showWarnings = FALSE, recursive = TRUE)
write.csv(map_rows, "outputs/result.csv", row.names = FALSE)
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

Attaching package: ‘dplyr’

The following objects are masked from ‘package:stats’:

    filter, lag

The following objects are masked from ‘package:base’:

    intersect, setdiff, setequal, union

Error in assert_parameters_argument(constant_parameters, optional = TRUE) : 
  unused argument (optional = TRUE)
Calls: derive_param_computed
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
library(dplyr)

# Read input dataset
dataset <- read.table("inputs/dataset.tsv", sep = "\t", header = TRUE,
                      colClasses = c(USUBJID = "character", PARAMCD = "character", AVAL = "numeric"))

# Derive MAP parameter using admiral's derive_param_computed
# MAP = (2 * DIABP + SYSBP) / 3
result <- derive_param_computed(
  dataset,
  by_vars    = exprs(USUBJID),
  parameters = c("SYSBP", "DIABP"),
  set_values_to = exprs(
    AVAL    = (2 * AVAL.DIABP + AVAL.SYSBP) / 3,
    PARAMCD = "MAP"
  )
)

# Keep only MAP rows with required columns
map_rows <- result %>%
  filter(PARAMCD == "MAP") %>%
  select(USUBJID, PARAMCD, AVAL)

# Write output
dir.create("outputs", showWarnings = FALSE, recursive = TRUE)
write.csv(map_rows, "outputs/result.csv", row.names = FALSE)
```
