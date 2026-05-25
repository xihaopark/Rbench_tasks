# Case 111: pharmaverse/admiral/derive_vars_crit_flag

## Metadata
- Task ID: `pharmaverse/admiral/derive_vars_crit_flag`
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

Write an R script for an R package task: perform derive vars crit flag using the admiral package. Use functions from the admiral package when suitable.

Input: dataset.tsv, description.tsv
Output: result.csv

Computation: Set CRIT1FL to Y when AVAL > 3 * ULN and to N otherwise.


Required columns for result.csv: USUBJID, PARAMCD, AVAL, ULN, CRIT1, CRIT1FL
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.

## Input preview

### dataset.tsv
USUBJID	PARAMCD	AVAL	ULN
01	ALT	45	20
02	ALT	75	20
03	AST	30	20

### description.tsv
description
ALT greater than three times ULN
```

## Input
#### `dataset.tsv`

```text
USUBJID	PARAMCD	AVAL	ULN
01	ALT	45	20
02	ALT	75	20
03	AST	30	20
```

#### `description.tsv`

```text
description
ALT greater than three times ULN
```

## Code
### Ground Truth Code

```r
result_df <- read.delim(file.path("inputs", "dataset.tsv"), check.names = FALSE, stringsAsFactors = FALSE)
description <- as.character(read.delim(file.path("inputs", "description.tsv"), check.names = FALSE, stringsAsFactors = FALSE)$description[1])
result_df$CRIT1 <- description
result_df$CRIT1FL <- ifelse(result_df$PARAMCD == "ALT" & result_df$AVAL > 3 * result_df$ULN, "Y", "N")

dir.create("outputs", showWarnings = FALSE)
write.csv(result_df, file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
library(admiral)
library(dplyr)
library(readr)

# Read inputs
dataset <- read_tsv("inputs/dataset.tsv", col_types = cols(
  USUBJID = col_character(),
  PARAMCD = col_character(),
  AVAL    = col_double(),
  ULN     = col_double()
))

description_df <- read_tsv("inputs/description.tsv", col_types = cols(.default = col_character()))
crit1_desc <- description_df$description[1]

# Create outputs directory
dir.create("outputs", showWarnings = FALSE, recursive = TRUE)

# Derive criterion flag variables using admiral
# values_yn = TRUE -> CRIT1FL is "Y" or "N" (not "Y" or NA)
result <- derive_vars_crit_flag(
  dataset,
  crit_nr     = 1,
  condition   = AVAL > 3 * ULN,
  description = crit1_desc,
  values_yn   = TRUE
)

# Select and order required columns
result <- result %>%
  select(USUBJID, PARAMCD, AVAL, ULN, CRIT1, CRIT1FL)

# Write output
write.csv(result, "outputs/result.csv", row.names = FALSE)
```

## Output
### Ground Truth Output
#### `result.csv`

```csv
"USUBJID","PARAMCD","AVAL","ULN","CRIT1","CRIT1FL"
1,"ALT",45,20,"ALT greater than three times ULN","N"
2,"ALT",75,20,"ALT greater than three times ULN","Y"
3,"AST",30,20,"ALT greater than three times ULN","N"
```

### LLM Output
No LLM output artifact was produced.

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
library(admiral)
library(dplyr)
library(readr)

# Read inputs
dataset <- read_tsv("inputs/dataset.tsv", col_types = cols(
  USUBJID = col_character(),
  PARAMCD = col_character(),
  AVAL    = col_double(),
  ULN     = col_double()
))

description_df <- read_tsv("inputs/description.tsv", col_types = cols(.default = col_character()))
crit1_desc <- description_df$description[1]

# Create outputs directory
dir.create("outputs", showWarnings = FALSE, recursive = TRUE)

# Derive criterion flag variables using admiral
# values_yn = TRUE -> CRIT1FL is "Y" or "N" (not "Y" or NA)
result <- derive_vars_crit_flag(
  dataset,
  crit_nr     = 1,
  condition   = AVAL > 3 * ULN,
  description = crit1_desc,
  values_yn   = TRUE
)

# Select and order required columns
result <- result %>%
  select(USUBJID, PARAMCD, AVAL, ULN, CRIT1, CRIT1FL)

# Write output
write.csv(result, "outputs/result.csv", row.names = FALSE)
```
