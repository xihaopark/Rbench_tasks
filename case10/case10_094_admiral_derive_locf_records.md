# Case 094: pharmaverse/admiral/derive_locf_records

## Metadata
- Task ID: `pharmaverse/admiral/derive_locf_records`
- Package: `admiral`
- Model: `claude-code/claude-sonnet-4-6`
- Agent: `Claude Code`
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

Write an R script for an R package task: perform derive locf records using the admiral package. Use functions from the admiral package when suitable.

Input: dataset.tsv, visits.tsv
Output: result.csv

Computation: Carry the last observed value forward by USUBJID and PARAMCD ordered by AVISITN, creating only carried rows with DTYPE set to LOCF.


Required columns for result.csv: USUBJID, PARAMCD, AVISITN, AVAL, DTYPE
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.

## Input preview

### dataset.tsv
USUBJID	PARAMCD	AVISITN	AVAL
01	ALT	1	20
01	ALT	3	30
02	ALT	1	15
02	ALT	4	24

### visits.tsv
USUBJID	PARAMCD	AVISITN
01	ALT	1
01	ALT	2
01	ALT	3
02	ALT	1
02	ALT	2
... [2 more lines omitted]

Reference package function list:
The hidden reference solution's R package function calls are listed below. If the list is empty, the reference solution does not call package functions.
- package_functions: []
```

## Input
#### `dataset.tsv`

```text
USUBJID	PARAMCD	AVISITN	AVAL
01	ALT	1	20
01	ALT	3	30
02	ALT	1	15
02	ALT	4	24
```

#### `visits.tsv`

```text
USUBJID	PARAMCD	AVISITN
01	ALT	1
01	ALT	2
01	ALT	3
02	ALT	1
02	ALT	2
02	ALT	3
02	ALT	4
```

## Code
### Ground Truth Code

```r
dataset <- read.delim(file.path("inputs", "dataset.tsv"), check.names = FALSE, stringsAsFactors = FALSE)
visits <- read.delim(file.path("inputs", "visits.tsv"), check.names = FALSE, stringsAsFactors = FALSE)
visits <- visits[order(visits$USUBJID, visits$PARAMCD, visits$AVISITN), , drop = FALSE]
dataset <- dataset[order(dataset$USUBJID, dataset$PARAMCD, dataset$AVISITN), , drop = FALSE]
result_df <- visits
result_df$AVAL <- NA_real_
result_df$DTYPE <- ""
for (i in seq_len(nrow(result_df))) {
  prior <- dataset[
    dataset$USUBJID == result_df$USUBJID[i] &
      dataset$PARAMCD == result_df$PARAMCD[i] &
      dataset$AVISITN <= result_df$AVISITN[i],
    ,
    drop = FALSE
  ]
  if (nrow(prior) > 0) {
    prior <- prior[order(prior$AVISITN), , drop = FALSE]
    last <- prior[nrow(prior), , drop = FALSE]
    result_df$AVAL[i] <- last$AVAL
    result_df$DTYPE[i] <- ifelse(last$AVISITN == result_df$AVISITN[i], "", "LOCF")
  }
}

dir.create("outputs", showWarnings = FALSE)
write.csv(result_df, file.path("outputs", "result.csv"), row.names = FALSE, na = "")
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

library(dplyr)

dataset <- read.table("inputs/dataset.tsv", header = TRUE, sep = "\t", stringsAsFactors = FALSE)
visits  <- read.table("inputs/visits.tsv",  header = TRUE, sep = "\t", stringsAsFactors = FALSE)

# Identify missing visits: in visits but not in dataset
missing_visits <- anti_join(visits, dataset, by = c("USUBJID", "PARAMCD", "AVISITN"))

# For each missing visit, cross-join with dataset observations that are prior
# then pick the last observed AVAL (max AVISITN <= visit)
locf_rows <- missing_visits %>%
  left_join(dataset, by = c("USUBJID", "PARAMCD"), suffix = c("", "_obs")) %>%
  filter(AVISITN_obs < AVISITN) %>%
  group_by(USUBJID, PARAMCD, AVISITN) %>%
  slice_max(AVISITN_obs, n = 1, with_ties = FALSE) %>%
  ungroup() %>%
  mutate(AVAL = AVAL, DTYPE = "LOCF") %>%
  select(USUBJID, PARAMCD, AVISITN, AVAL, DTYPE)

dir.create("outputs", showWarnings = FALSE, recursive = TRUE)
write.csv(locf_rows, "outputs/result.csv", row.names = FALSE)
```

## Output
### Ground Truth Output
#### `result.csv`

```csv
"USUBJID","PARAMCD","AVISITN","AVAL","DTYPE"
1,"ALT",1,20,""
1,"ALT",2,20,"LOCF"
1,"ALT",3,30,""
2,"ALT",1,15,""
2,"ALT",2,15,"LOCF"
2,"ALT",3,15,"LOCF"
2,"ALT",4,24,""
```

### LLM Output
#### `result.csv`

```csv
"USUBJID","PARAMCD","AVISITN","AVAL","DTYPE"
1,"ALT",2,20,"LOCF"
2,"ALT",2,15,"LOCF"
2,"ALT",3,15,"LOCF"
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

Warning message:
In left_join(., dataset, by = c("USUBJID", "PARAMCD"), suffix = c("",  :
  Detected an unexpected many-to-many relationship between `x` and `y`.
ℹ Row 1 of `x` matches multiple rows in `y`.
ℹ Row 3 of `y` matches multiple rows in `x`.
ℹ If a many-to-many relationship is expected, set `relationship =
  "many-to-many"` to silence this warning.
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

library(dplyr)

dataset <- read.table("inputs/dataset.tsv", header = TRUE, sep = "\t", stringsAsFactors = FALSE)
visits  <- read.table("inputs/visits.tsv",  header = TRUE, sep = "\t", stringsAsFactors = FALSE)

# Identify missing visits: in visits but not in dataset
missing_visits <- anti_join(visits, dataset, by = c("USUBJID", "PARAMCD", "AVISITN"))

# For each missing visit, cross-join with dataset observations that are prior
# then pick the last observed AVAL (max AVISITN <= visit)
locf_rows <- missing_visits %>%
  left_join(dataset, by = c("USUBJID", "PARAMCD"), suffix = c("", "_obs")) %>%
  filter(AVISITN_obs < AVISITN) %>%
  group_by(USUBJID, PARAMCD, AVISITN) %>%
  slice_max(AVISITN_obs, n = 1, with_ties = FALSE) %>%
  ungroup() %>%
  mutate(AVAL = AVAL, DTYPE = "LOCF") %>%
  select(USUBJID, PARAMCD, AVISITN, AVAL, DTYPE)

dir.create("outputs", showWarnings = FALSE, recursive = TRUE)
write.csv(locf_rows, "outputs/result.csv", row.names = FALSE)
```
