# Case 147: pharmaverse/admiral/get_imputation_targets

## Metadata
- Task ID: `pharmaverse/admiral/get_imputation_targets`
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

Write an R script for an R package task: perform get imputation targets using the admiral package. Use functions from the admiral package when suitable.

Input: partial.tsv
Output: result.csv

Computation: For partial date/time targets, year-only dates use month 06 and day 15; missing time components use 00.


Required columns for result.csv: partial, year, month, day, hour, minute, second
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.

## Input preview

### partial.tsv
partial
2020
2020-02
2020-02-20T14
```

## Input
#### `partial.tsv`

```text
partial
2020
2020-02
2020-02-20T14
```

## Code
### Ground Truth Code

```r
partial <- as.character(read.delim(file.path("inputs", "partial.tsv"), check.names = FALSE, stringsAsFactors = FALSE)$partial)
year <- substr(partial, 1, 4)
month <- ifelse(nchar(partial) >= 7, substr(partial, 6, 7), "06")
day <- ifelse(nchar(partial) >= 10, substr(partial, 9, 10), "15")
hour <- ifelse(nchar(partial) >= 13, substr(partial, 12, 13), "00")
minute <- ifelse(nchar(partial) >= 16, substr(partial, 15, 16), "00")
second <- ifelse(nchar(partial) >= 19, substr(partial, 18, 19), "00")
result_df <- data.frame(partial = partial, year = year, month = month, day = day, hour = hour, minute = minute, second = second, stringsAsFactors = FALSE)

dir.create("outputs", showWarnings = FALSE)
write.csv(result_df, file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
library(admiral)

# Read input
partial_data <- read.delim("inputs/partial.tsv", stringsAsFactors = FALSE)

# Determine whether the partial string contains a time component
has_time <- function(partial_str) {
  grepl("T", partial_str)
}

# For each partial date/time string, parse components and apply imputation targets
process_partial <- function(partial_str) {
  create_dt <- has_time(partial_str)

  # Parse into partial components list
  partial_list <- admiral:::get_partialdatetime(partial_str, create_datetime = create_dt)

  # Get imputation targets:
  # - date_imputation = "06-15": year-only -> month=06, day=15 (per task rules)
  # - time_imputation = "first": missing time components -> "00"
  time_imp <- if (create_dt) "first" else NULL
  targets <- admiral:::get_imputation_targets(
    partial_list,
    date_imputation = "06-15",
    time_imputation = time_imp
  )

  # Apply targets: use actual value if present, otherwise use the imputation target
  # The "xxxx" sentinel means "keep original year value"
  resolve <- function(actual, target) {
    if (!is.null(actual) && !is.na(actual)) {
      as.integer(actual)
    } else if (!is.null(target) && !is.na(target) && target != "xxxx") {
      as.integer(target)
    } else {
      NA_integer_
    }
  }

  year_val  <- resolve(partial_list$year,  targets$year)
  month_val <- resolve(partial_list$month, targets$month)
  day_val   <- resolve(partial_list$day,   targets$day)

  if (create_dt) {
    hour_val   <- resolve(partial_list$hour,   targets$hour)
    minute_val <- resolve(partial_list$minute, targets$minute)
    second_val <- resolve(partial_list$second, targets$second)
  } else {
    # No time part in partial string -> impute all time components to 00
    hour_val   <- 0L
    minute_val <- 0L
    second_val <- 0L
  }

  data.frame(
    partial = partial_str,
    year    = year_val,
    month   = month_val,
    day     = day_val,
    hour    = hour_val,
    minute  = minute_val,
    second  = second_val,
    stringsAsFactors = FALSE
  )
}

result <- do.call(rbind, lapply(partial_data$partial, process_partial))

dir.create("outputs", showWarnings = FALSE, recursive = TRUE)
write.csv(result, "outputs/result.csv", row.names = FALSE)

cat("Done. Output written to outputs/result.csv\n")
print(result)
```

## Output
### Ground Truth Output
#### `result.csv`

```csv
"partial","year","month","day","hour","minute","second"
"2020","2020","06","15","00","00","00"
"2020-02","2020","02","15","00","00","00"
"2020-02-20T14","2020","02","20","14","00","00"
```

### LLM Output
No LLM output artifact was produced.

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
library(admiral)

# Read input
partial_data <- read.delim("inputs/partial.tsv", stringsAsFactors = FALSE)

# Determine whether the partial string contains a time component
has_time <- function(partial_str) {
  grepl("T", partial_str)
}

# For each partial date/time string, parse components and apply imputation targets
process_partial <- function(partial_str) {
  create_dt <- has_time(partial_str)

  # Parse into partial components list
  partial_list <- admiral:::get_partialdatetime(partial_str, create_datetime = create_dt)

  # Get imputation targets:
  # - date_imputation = "06-15": year-only -> month=06, day=15 (per task rules)
  # - time_imputation = "first": missing time components -> "00"
  time_imp <- if (create_dt) "first" else NULL
  targets <- admiral:::get_imputation_targets(
    partial_list,
    date_imputation = "06-15",
    time_imputation = time_imp
  )

  # Apply targets: use actual value if present, otherwise use the imputation target
  # The "xxxx" sentinel means "keep original year value"
  resolve <- function(actual, target) {
    if (!is.null(actual) && !is.na(actual)) {
      as.integer(actual)
    } else if (!is.null(target) && !is.na(target) && target != "xxxx") {
      as.integer(target)
    } else {
      NA_integer_
    }
  }

  year_val  <- resolve(partial_list$year,  targets$year)
  month_val <- resolve(partial_list$month, targets$month)
  day_val   <- resolve(partial_list$day,   targets$day)

  if (create_dt) {
    hour_val   <- resolve(partial_list$hour,   targets$hour)
    minute_val <- resolve(partial_list$minute, targets$minute)
    second_val <- resolve(partial_list$second, targets$second)
  } else {
    # No time part in partial string -> impute all time components to 00
    hour_val   <- 0L
    minute_val <- 0L
    second_val <- 0L
  }

  data.frame(
    partial = partial_str,
    year    = year_val,
    month   = month_val,
    day     = day_val,
    hour    = hour_val,
    minute  = minute_val,
    second  = second_val,
    stringsAsFactors = FALSE
  )
}

result <- do.call(rbind, lapply(partial_data$partial, process_partial))

dir.create("outputs", showWarnings = FALSE, recursive = TRUE)
write.csv(result, "outputs/result.csv", row.names = FALSE)

cat("Done. Output written to outputs/result.csv\n")
print(result)
```
