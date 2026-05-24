# Case 147: pharmaverse/admiral/get_imputation_targets

## Metadata
- Task ID: `pharmaverse/admiral/get_imputation_targets`
- Package: `admiral`
- Model: `codex/gpt-5.5`
- Agent: `Codex CLI`
- Status: `NO_OUTPUT`
- Failure stage: `execution_failure`
- Pattern: `invalid_or_internal_package_api`

## Pass/Error
```text
NO_OUTPUT
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

Write an R script for an R package task: perform get imputation targets using the admiral package. Use functions from the admiral package when suitable.

Input: partial.tsv
Output: result.csv

Computation: For partial date/time targets, year-only dates use month 06 and day 15; missing time components use 00.


Required columns for result.csv: partial, year, month, day, hour, minute, second
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
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

input_path <- file.path("inputs", "partial.tsv")
output_dir <- "outputs"
output_path <- file.path(output_dir, "result.csv")

if (!file.exists(input_path)) {
  stop("Input file not found: ", input_path)
}

partial_data <- read.delim(
  input_path,
  stringsAsFactors = FALSE,
  colClasses = c(partial = "character"),
  check.names = FALSE
)

if (!identical(names(partial_data), "partial")) {
  stop("Input file must contain exactly one column named 'partial'.")
}

get_targets <- getFromNamespace("get_imputation_targets", "admiral")

parse_partial <- function(value) {
  pattern <- paste0(
    "^([0-9]{4})",
    "(?:-([0-9]{2})",
    "(?:-([0-9]{2})",
    "(?:T([0-9]{2})",
    "(?::([0-9]{2})",
    "(?::([0-9]{2}))?)?)?)?)?$"
  )
  pieces <- regexec(pattern, value)
  matches <- regmatches(value, pieces)[[1]]

  if (length(matches) == 0) {
    stop("Invalid partial date/time value: ", value)
  }

  components <- matches[-1]
  components[components == ""] <- NA_character_
  names(components) <- c("year", "month", "day", "hour", "minute", "second")
  as.list(components)
}

impute_components <- function(value) {
  partial <- parse_partial(value)
  targets <- get_targets(
    partial = partial,
    date_imputation = "06-15",
    time_imputation = "first"
  )

  resolved <- partial
  for (component in names(resolved)) {
    if (is.na(resolved[[component]]) || resolved[[component]] == "") {
      resolved[[component]] <- targets[[component]]
    }
  }

  data.frame(
    partial = value,
    year = resolved$year,
    month = resolved$month,
    day = resolved$day,
    hour = resolved$hour,
    minute = resolved$minute,
    second = resolved$second,
    stringsAsFactors = FALSE,
    check.names = FALSE
  )
}

result <- do.call(rbind, lapply(partial_data$partial, impute_components))

dir.create(output_dir, showWarnings = FALSE, recursive = TRUE)
write.table(
  result,
  file = output_path,
  sep = ",",
  row.names = FALSE,
  col.names = TRUE,
  quote = FALSE,
  na = ""
)

if (!file.exists(output_path)) {
  stop("Failed to write output file: ", output_path)
}
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
Error in get_targets(partial = partial, date_imputation = "06-15", time_imputation = "first") : 
  unused arguments (partial = partial, date_imputation = "06-15", time_imputation = "first")
Calls: do.call -> lapply -> FUN
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
library(admiral)

input_path <- file.path("inputs", "partial.tsv")
output_dir <- "outputs"
output_path <- file.path(output_dir, "result.csv")

if (!file.exists(input_path)) {
  stop("Input file not found: ", input_path)
}

partial_data <- read.delim(
  input_path,
  stringsAsFactors = FALSE,
  colClasses = c(partial = "character"),
  check.names = FALSE
)

if (!identical(names(partial_data), "partial")) {
  stop("Input file must contain exactly one column named 'partial'.")
}

get_targets <- getFromNamespace("get_imputation_targets", "admiral")

parse_partial <- function(value) {
  pattern <- paste0(
    "^([0-9]{4})",
    "(?:-([0-9]{2})",
    "(?:-([0-9]{2})",
    "(?:T([0-9]{2})",
    "(?::([0-9]{2})",
    "(?::([0-9]{2}))?)?)?)?)?$"
  )
  pieces <- regexec(pattern, value)
  matches <- regmatches(value, pieces)[[1]]

  if (length(matches) == 0) {
    stop("Invalid partial date/time value: ", value)
  }

  components <- matches[-1]
  components[components == ""] <- NA_character_
  names(components) <- c("year", "month", "day", "hour", "minute", "second")
  as.list(components)
}

impute_components <- function(value) {
  partial <- parse_partial(value)
  targets <- get_targets(
    partial = partial,
    date_imputation = "06-15",
    time_imputation = "first"
  )

  resolved <- partial
  for (component in names(resolved)) {
    if (is.na(resolved[[component]]) || resolved[[component]] == "") {
      resolved[[component]] <- targets[[component]]
    }
  }

  data.frame(
    partial = value,
    year = resolved$year,
    month = resolved$month,
    day = resolved$day,
    hour = resolved$hour,
    minute = resolved$minute,
    second = resolved$second,
    stringsAsFactors = FALSE,
    check.names = FALSE
  )
}

result <- do.call(rbind, lapply(partial_data$partial, impute_components))

dir.create(output_dir, showWarnings = FALSE, recursive = TRUE)
write.table(
  result,
  file = output_path,
  sep = ",",
  row.names = FALSE,
  col.names = TRUE,
  quote = FALSE,
  na = ""
)

if (!file.exists(output_path)) {
  stop("Failed to write output file: ", output_path)
}
```
