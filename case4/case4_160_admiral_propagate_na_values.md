# Case 160: pharmaverse/admiral/propagate_na_values

## Metadata
- Task ID: `pharmaverse/admiral/propagate_na_values`
- Package: `admiral`
- Model: `codex/gpt-5.5`
- Agent: `Codex CLI`
- Status: `FAIL`
- Failure stage: `value_mismatch`
- Pattern: `invalid_or_internal_package_api`

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

Write an R script for an R package task: perform propagate na values using the admiral package. Use functions from the admiral package when suitable.

Input: partial.tsv
Output: result.csv

Computation: For ordered datetime components, keep components before the first missing component and blank all later components.


Required columns for result.csv: year, month, day, hour, minute, second
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `partial.tsv`

```text
year	month	day	hour	minute	second
2020	05	NA	10	30	45
2021	NA	12	09	15	00
2022	07	04	NA	20	30
```

## Code
### Ground Truth Code

```r
result_df <- read.delim(file.path("inputs", "partial.tsv"), check.names = FALSE, stringsAsFactors = FALSE, na.strings = c("NA", ""))
cols <- c("year", "month", "day", "hour", "minute", "second")
for (i in seq_len(nrow(result_df))) {
  missing_seen <- FALSE
  for (col in cols) {
    if (missing_seen || is.na(result_df[[col]][i]) || result_df[[col]][i] == "") {
      result_df[[col]][i] <- NA_character_
      missing_seen <- TRUE
    }
  }
}

dir.create("outputs", showWarnings = FALSE)
write.csv(result_df, file.path("outputs", "result.csv"), row.names = FALSE, na = "")
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

library(admiral)

input_path <- file.path("inputs", "partial.tsv")
output_dir <- "outputs"
output_path <- file.path(output_dir, "result.csv")

components <- c("year", "month", "day", "hour", "minute", "second")

partial <- read.delim(
  input_path,
  sep = "\t",
  header = TRUE,
  colClasses = "character",
  na.strings = "NA",
  check.names = FALSE
)

missing_cols <- setdiff(components, names(partial))
if (length(missing_cols) > 0) {
  stop("Missing required column(s): ", paste(missing_cols, collapse = ", "))
}

partial <- partial[components]

propagate_na_values <- getFromNamespace("propagate_na_values", "admiral")

result <- as.data.frame(
  propagate_na_values(as.list(partial)),
  stringsAsFactors = FALSE,
  check.names = FALSE
)

dir.create(output_dir, showWarnings = FALSE, recursive = TRUE)
write.csv(result[components], output_path, row.names = FALSE, na = "", quote = FALSE)
```

## Output
### Ground Truth Output
#### `result.csv`

```csv
"year","month","day","hour","minute","second"
2020,"5",,,,
2021,,,,,
2022,"7","4",,,
```

### LLM Output
#### `result.csv`

```csv
year,month,day,hour,minute,second
2020,05,,10,30,45
2021,,12,09,15,00
2022,07,04,,20,30
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

library(admiral)

input_path <- file.path("inputs", "partial.tsv")
output_dir <- "outputs"
output_path <- file.path(output_dir, "result.csv")

components <- c("year", "month", "day", "hour", "minute", "second")

partial <- read.delim(
  input_path,
  sep = "\t",
  header = TRUE,
  colClasses = "character",
  na.strings = "NA",
  check.names = FALSE
)

missing_cols <- setdiff(components, names(partial))
if (length(missing_cols) > 0) {
  stop("Missing required column(s): ", paste(missing_cols, collapse = ", "))
}

partial <- partial[components]

propagate_na_values <- getFromNamespace("propagate_na_values", "admiral")

result <- as.data.frame(
  propagate_na_values(as.list(partial)),
  stringsAsFactors = FALSE,
  check.names = FALSE
)

dir.create(output_dir, showWarnings = FALSE, recursive = TRUE)
write.csv(result[components], output_path, row.names = FALSE, na = "", quote = FALSE)
```
