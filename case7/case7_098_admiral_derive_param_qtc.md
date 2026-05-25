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

library(dplyr)

# Read input data
dataset <- read.table("inputs/dataset.tsv", header = TRUE, sep = "\t",
                      colClasses = c("character", "character", "numeric"))

# Separate QT and RR records
qt_data <- dataset %>%
  filter(PARAMCD == "QT") %>%
  rename(QT = AVAL) %>%
  select(USUBJID, QT)

rr_data <- dataset %>%
  filter(PARAMCD == "RR") %>%
  rename(RR = AVAL) %>%
  select(USUBJID, RR)

# Join paired QT and RR by subject
paired <- inner_join(qt_data, rr_data, by = "USUBJID")

# Derive QTCBR using Bazett's formula: QTCBR = QT / sqrt(RR)
qtcbr <- paired %>%
  mutate(
    PARAMCD = "QTCBR",
    AVAL    = QT / sqrt(RR)
  ) %>%
  select(USUBJID, PARAMCD, AVAL)

# Create outputs directory
dir.create("outputs", showWarnings = FALSE, recursive = TRUE)

# Write result
write.csv(qtcbr, "outputs/result.csv", row.names = FALSE)

cat("Done. Rows written:", nrow(qtcbr), "\n")
print(qtcbr)
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
No LLM output artifact was produced.

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

library(dplyr)

# Read input data
dataset <- read.table("inputs/dataset.tsv", header = TRUE, sep = "\t",
                      colClasses = c("character", "character", "numeric"))

# Separate QT and RR records
qt_data <- dataset %>%
  filter(PARAMCD == "QT") %>%
  rename(QT = AVAL) %>%
  select(USUBJID, QT)

rr_data <- dataset %>%
  filter(PARAMCD == "RR") %>%
  rename(RR = AVAL) %>%
  select(USUBJID, RR)

# Join paired QT and RR by subject
paired <- inner_join(qt_data, rr_data, by = "USUBJID")

# Derive QTCBR using Bazett's formula: QTCBR = QT / sqrt(RR)
qtcbr <- paired %>%
  mutate(
    PARAMCD = "QTCBR",
    AVAL    = QT / sqrt(RR)
  ) %>%
  select(USUBJID, PARAMCD, AVAL)

# Create outputs directory
dir.create("outputs", showWarnings = FALSE, recursive = TRUE)

# Write result
write.csv(qtcbr, "outputs/result.csv", row.names = FALSE)

cat("Done. Rows written:", nrow(qtcbr), "\n")
print(qtcbr)
```
