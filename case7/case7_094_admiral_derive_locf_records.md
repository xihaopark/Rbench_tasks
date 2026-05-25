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
library(dplyr)

# ---------------------------------------------------------------------------
# Read inputs
# ---------------------------------------------------------------------------
dataset <- read.table(
  "inputs/dataset.tsv", header = TRUE, sep = "\t",
  colClasses = c("character", "character", "numeric", "numeric")
)

visits <- read.table(
  "inputs/visits.tsv", header = TRUE, sep = "\t",
  colClasses = c("character", "character", "numeric")
)

# ---------------------------------------------------------------------------
# Derive LOCF records using admiral if available, otherwise implement directly
# ---------------------------------------------------------------------------

use_admiral <- tryCatch({
  library(admiral)
  packageVersion("admiral")
  TRUE
}, error = function(e) FALSE)

if (use_admiral) {
  # admiral::derive_locf_records creates LOCF-imputed records for visits that
  # are in dataset_ref but missing in dataset, for each by_vars group ordered
  # by order.
  locf_all <- derive_locf_records(
    dataset      = dataset,
    dataset_ref  = visits,
    by_vars      = exprs(USUBJID, PARAMCD),
    analysis_var = AVAL,
    order        = exprs(AVISITN)
  )

  result <- locf_all %>%
    filter(DTYPE == "LOCF") %>%
    select(USUBJID, PARAMCD, AVISITN, AVAL, DTYPE)

} else {
  # ------------------------------------------------------------------
  # Direct LOCF implementation (no admiral)
  # ------------------------------------------------------------------
  # Step 1: left-join reference visits onto dataset to flag missing rows
  joined <- visits %>%
    left_join(dataset, by = c("USUBJID", "PARAMCD", "AVISITN"))

  # Step 2: within each group ordered by AVISITN, carry AVAL forward
  locf_filled <- joined %>%
    arrange(USUBJID, PARAMCD, AVISITN) %>%
    group_by(USUBJID, PARAMCD) %>%
    mutate(AVAL_locf = zoo::na.locf(AVAL, na.rm = FALSE)) %>%
    ungroup()

  # Step 3: keep only rows that were missing in the original dataset
  #         (those are the LOCF-imputed rows)
  result <- locf_filled %>%
    filter(is.na(AVAL) & !is.na(AVAL_locf)) %>%
    mutate(
      AVAL  = AVAL_locf,
      DTYPE = "LOCF"
    ) %>%
    select(USUBJID, PARAMCD, AVISITN, AVAL, DTYPE)
}

# ---------------------------------------------------------------------------
# Write output
# ---------------------------------------------------------------------------
dir.create("outputs", showWarnings = FALSE)
write.csv(result, "outputs/result.csv", row.names = FALSE)

cat("Done. LOCF rows written:", nrow(result), "\n")
print(result)
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
No LLM output artifact was produced.

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
library(dplyr)

# ---------------------------------------------------------------------------
# Read inputs
# ---------------------------------------------------------------------------
dataset <- read.table(
  "inputs/dataset.tsv", header = TRUE, sep = "\t",
  colClasses = c("character", "character", "numeric", "numeric")
)

visits <- read.table(
  "inputs/visits.tsv", header = TRUE, sep = "\t",
  colClasses = c("character", "character", "numeric")
)

# ---------------------------------------------------------------------------
# Derive LOCF records using admiral if available, otherwise implement directly
# ---------------------------------------------------------------------------

use_admiral <- tryCatch({
  library(admiral)
  packageVersion("admiral")
  TRUE
}, error = function(e) FALSE)

if (use_admiral) {
  # admiral::derive_locf_records creates LOCF-imputed records for visits that
  # are in dataset_ref but missing in dataset, for each by_vars group ordered
  # by order.
  locf_all <- derive_locf_records(
    dataset      = dataset,
    dataset_ref  = visits,
    by_vars      = exprs(USUBJID, PARAMCD),
    analysis_var = AVAL,
    order        = exprs(AVISITN)
  )

  result <- locf_all %>%
    filter(DTYPE == "LOCF") %>%
    select(USUBJID, PARAMCD, AVISITN, AVAL, DTYPE)

} else {
  # ------------------------------------------------------------------
  # Direct LOCF implementation (no admiral)
  # ------------------------------------------------------------------
  # Step 1: left-join reference visits onto dataset to flag missing rows
  joined <- visits %>%
    left_join(dataset, by = c("USUBJID", "PARAMCD", "AVISITN"))

  # Step 2: within each group ordered by AVISITN, carry AVAL forward
  locf_filled <- joined %>%
    arrange(USUBJID, PARAMCD, AVISITN) %>%
    group_by(USUBJID, PARAMCD) %>%
    mutate(AVAL_locf = zoo::na.locf(AVAL, na.rm = FALSE)) %>%
    ungroup()

  # Step 3: keep only rows that were missing in the original dataset
  #         (those are the LOCF-imputed rows)
  result <- locf_filled %>%
    filter(is.na(AVAL) & !is.na(AVAL_locf)) %>%
    mutate(
      AVAL  = AVAL_locf,
      DTYPE = "LOCF"
    ) %>%
    select(USUBJID, PARAMCD, AVISITN, AVAL, DTYPE)
}

# ---------------------------------------------------------------------------
# Write output
# ---------------------------------------------------------------------------
dir.create("outputs", showWarnings = FALSE)
write.csv(result, "outputs/result.csv", row.names = FALSE)

cat("Done. LOCF rows written:", nrow(result), "\n")
print(result)
```
