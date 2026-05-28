# Case 101: pharmaverse/admiral/derive_var_atoxgr_dir

## Metadata
- Task ID: `pharmaverse/admiral/derive_var_atoxgr_dir`
- Package: `admiral`
- Model: `claude-code/claude-sonnet-4-6`
- Agent: `Claude Code`
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

Write an R script for an R package task: perform derive var atoxgr dir using the admiral package. Use functions from the admiral package when suitable.

Input: dataset.tsv
Output: result.csv

Computation: Set ATOXGRDIR to L when AVAL < ANRLO, H when AVAL > ANRHI, and N otherwise.


Required columns for result.csv: USUBJID, PARAMCD, AVAL, ANRLO, ANRHI, ATOXGRDIR
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.

## Input preview

### dataset.tsv
USUBJID	PARAMCD	AVAL	ANRLO	ANRHI
01	ALT	15	20	40
01	AST	55	10	35
02	BILI	0.8	0.2	1.2
03	CREAT	1.5	0.5	1.1

Reference package function list:
The hidden reference solution's R package function calls are listed below. If the list is empty, the reference solution does not call package functions.
- package_functions: []
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
#!/usr/bin/env Rscript

library(dplyr)

dataset <- read.delim("inputs/dataset.tsv", stringsAsFactors = FALSE)

result <- dataset %>%
  mutate(ATOXGRDIR = case_when(
    AVAL < ANRLO ~ "L",
    AVAL > ANRHI ~ "H",
    TRUE         ~ "N"
  )) %>%
  select(USUBJID, PARAMCD, AVAL, ANRLO, ANRHI, ATOXGRDIR)

dir.create("outputs", showWarnings = FALSE, recursive = TRUE)
write.csv(result, "outputs/result.csv", row.names = FALSE)
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
1,"ALT",15,20,40,"L"
1,"AST",55,10,35,"H"
2,"BILI",0.8,0.2,1.2,"N"
3,"CREAT",1.5,0.5,1.1,"H"
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

library(dplyr)

dataset <- read.delim("inputs/dataset.tsv", stringsAsFactors = FALSE)

result <- dataset %>%
  mutate(ATOXGRDIR = case_when(
    AVAL < ANRLO ~ "L",
    AVAL > ANRHI ~ "H",
    TRUE         ~ "N"
  )) %>%
  select(USUBJID, PARAMCD, AVAL, ANRLO, ANRHI, ATOXGRDIR)

dir.create("outputs", showWarnings = FALSE, recursive = TRUE)
write.csv(result, "outputs/result.csv", row.names = FALSE)
```
