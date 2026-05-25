# Case 142: pharmaverse/admiral/get_flagged_records

## Metadata
- Task ID: `pharmaverse/admiral/get_flagged_records`
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

Write an R script for an R package task: perform get flagged records using the admiral package. Use functions from the admiral package when suitable.

Input: dataset.tsv, new_var.tsv, threshold.tsv
Output: result.csv

Computation: Return only records satisfying AVAL > threshold and set the requested flag variable to Y in those returned rows.


Required columns for result.csv: USUBJID, PARAMCD, AVAL, AVISIT, HIGHFL
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.

## Input preview

### dataset.tsv
USUBJID	PARAMCD	AVAL	AVISIT
01	ALT	45	WEEK 1
01	ALT	18	WEEK 2
02	AST	52	WEEK 1
02	ALT	38	WEEK 1

### new_var.tsv
new_var
HIGHFL

### threshold.tsv
threshold
40
```

## Input
#### `dataset.tsv`

```text
USUBJID	PARAMCD	AVAL	AVISIT
01	ALT	45	WEEK 1
01	ALT	18	WEEK 2
02	AST	52	WEEK 1
02	ALT	38	WEEK 1
```

#### `new_var.tsv`

```text
new_var
HIGHFL
```

#### `threshold.tsv`

```text
threshold
40
```

## Code
### Ground Truth Code

```r
dataset <- read.delim(file.path("inputs", "dataset.tsv"), check.names = FALSE, stringsAsFactors = FALSE)
threshold <- as.numeric(read.delim(file.path("inputs", "threshold.tsv"), check.names = FALSE, stringsAsFactors = FALSE)$threshold[1])
new_var <- as.character(read.delim(file.path("inputs", "new_var.tsv"), check.names = FALSE, stringsAsFactors = FALSE)$new_var[1])
result_df <- dataset[dataset$PARAMCD == "ALT" & dataset$AVAL > threshold, , drop = FALSE]
result_df[[new_var]] <- "Y"

dir.create("outputs", showWarnings = FALSE)
write.csv(result_df, file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
library(admiral)
library(dplyr)

# Read inputs
dataset   <- read.delim("inputs/dataset.tsv",   stringsAsFactors = FALSE)
new_var   <- read.delim("inputs/new_var.tsv",    stringsAsFactors = FALSE)
threshold <- read.delim("inputs/threshold.tsv",  stringsAsFactors = FALSE)

# Extract scalar values
flag_var  <- new_var$new_var[1]      # "HIGHFL"
thr_val   <- threshold$threshold[1]  # 40

# Filter records where AVAL > threshold and set the flag variable to "Y"
result <- dataset %>%
  filter(AVAL > thr_val) %>%
  mutate(!!flag_var := "Y") %>%
  select(USUBJID, PARAMCD, AVAL, AVISIT, all_of(flag_var))

# Create outputs directory if needed and write result
dir.create("outputs", showWarnings = FALSE)
write.csv(result, "outputs/result.csv", row.names = FALSE)
```

## Output
### Ground Truth Output
#### `result.csv`

```csv
"USUBJID","PARAMCD","AVAL","AVISIT","HIGHFL"
1,"ALT",45,"WEEK 1","Y"
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
library(admiral)
library(dplyr)

# Read inputs
dataset   <- read.delim("inputs/dataset.tsv",   stringsAsFactors = FALSE)
new_var   <- read.delim("inputs/new_var.tsv",    stringsAsFactors = FALSE)
threshold <- read.delim("inputs/threshold.tsv",  stringsAsFactors = FALSE)

# Extract scalar values
flag_var  <- new_var$new_var[1]      # "HIGHFL"
thr_val   <- threshold$threshold[1]  # 40

# Filter records where AVAL > threshold and set the flag variable to "Y"
result <- dataset %>%
  filter(AVAL > thr_val) %>%
  mutate(!!flag_var := "Y") %>%
  select(USUBJID, PARAMCD, AVAL, AVISIT, all_of(flag_var))

# Create outputs directory if needed and write result
dir.create("outputs", showWarnings = FALSE)
write.csv(result, "outputs/result.csv", row.names = FALSE)
```
