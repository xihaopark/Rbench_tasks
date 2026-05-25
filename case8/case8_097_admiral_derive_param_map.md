# Case 097: pharmaverse/admiral/derive_param_map

## Metadata
- Task ID: `pharmaverse/admiral/derive_param_map`
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
library(admiral)
library(dplyr)

# Read input dataset
dataset <- read.table("inputs/dataset.tsv", header = TRUE, sep = "\t",
                      stringsAsFactors = FALSE)
dataset$AVAL <- as.numeric(dataset$AVAL)

# Create outputs directory
dir.create("outputs", showWarnings = FALSE, recursive = TRUE)

# Try using admiral's derive_param_map to add MAP rows to the dataset
result_full <- tryCatch({
  derive_param_map(
    dataset,
    by_vars = exprs(USUBJID),
    set_values_to = exprs(PARAMCD = "MAP"),
    sysbp_code = "SYSBP",
    diabp_code = "DIABP"
  )
}, error = function(e) {
  message("derive_param_map failed: ", e$message, " - computing manually")
  # Manual computation
  sysbp <- dataset[dataset$PARAMCD == "SYSBP", c("USUBJID", "AVAL")]
  names(sysbp)[2] <- "AVAL_SYSBP"
  diabp <- dataset[dataset$PARAMCD == "DIABP", c("USUBJID", "AVAL")]
  names(diabp)[2] <- "AVAL_DIABP"
  merged <- merge(sysbp, diabp, by = "USUBJID")
  map_rows <- data.frame(
    USUBJID = merged$USUBJID,
    PARAMCD = "MAP",
    AVAL = (2 * merged$AVAL_DIABP + merged$AVAL_SYSBP) / 3,
    stringsAsFactors = FALSE
  )
  rbind(dataset, map_rows)
})

# Filter to MAP rows only and select required columns
result <- result_full[result_full$PARAMCD == "MAP", c("USUBJID", "PARAMCD", "AVAL")]

# Write output
write.csv(result, "outputs/result.csv", row.names = FALSE)
message("Done. Rows written: ", nrow(result))
print(result)
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

# Read input dataset
dataset <- read.table("inputs/dataset.tsv", header = TRUE, sep = "\t",
                      stringsAsFactors = FALSE)
dataset$AVAL <- as.numeric(dataset$AVAL)

# Create outputs directory
dir.create("outputs", showWarnings = FALSE, recursive = TRUE)

# Try using admiral's derive_param_map to add MAP rows to the dataset
result_full <- tryCatch({
  derive_param_map(
    dataset,
    by_vars = exprs(USUBJID),
    set_values_to = exprs(PARAMCD = "MAP"),
    sysbp_code = "SYSBP",
    diabp_code = "DIABP"
  )
}, error = function(e) {
  message("derive_param_map failed: ", e$message, " - computing manually")
  # Manual computation
  sysbp <- dataset[dataset$PARAMCD == "SYSBP", c("USUBJID", "AVAL")]
  names(sysbp)[2] <- "AVAL_SYSBP"
  diabp <- dataset[dataset$PARAMCD == "DIABP", c("USUBJID", "AVAL")]
  names(diabp)[2] <- "AVAL_DIABP"
  merged <- merge(sysbp, diabp, by = "USUBJID")
  map_rows <- data.frame(
    USUBJID = merged$USUBJID,
    PARAMCD = "MAP",
    AVAL = (2 * merged$AVAL_DIABP + merged$AVAL_SYSBP) / 3,
    stringsAsFactors = FALSE
  )
  rbind(dataset, map_rows)
})

# Filter to MAP rows only and select required columns
result <- result_full[result_full$PARAMCD == "MAP", c("USUBJID", "PARAMCD", "AVAL")]

# Write output
write.csv(result, "outputs/result.csv", row.names = FALSE)
message("Done. Rows written: ", nrow(result))
print(result)
```
