# Case 147: pharmaverse/admiral/get_imputation_targets

## Metadata
- Task ID: `pharmaverse/admiral/get_imputation_targets`
- Package: `admiral`
- Model: `claude-code/claude-sonnet-4-6`
- Agent: `Claude Code`
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
input <- read.table("inputs/partial.tsv", header = TRUE, sep = "\t",
                    stringsAsFactors = FALSE, colClasses = "character")

# Function to parse a partial date/time string into named list of components
parse_partial <- function(partial) {
  # Split on 'T' to separate date and time parts
  parts <- strsplit(partial, "T")[[1]]
  date_part <- parts[1]
  time_part  <- if (length(parts) > 1) parts[2] else NA_character_

  # Split date into year, month, day
  date_components <- strsplit(date_part, "-")[[1]]
  year  <- if (length(date_components) >= 1) date_components[1] else NA_character_
  month <- if (length(date_components) >= 2) date_components[2] else NA_character_
  day   <- if (length(date_components) >= 3) date_components[3] else NA_character_

  # Split time into hour, minute, second
  if (!is.na(time_part)) {
    time_components <- strsplit(time_part, ":")[[1]]
    hour   <- if (length(time_components) >= 1) time_components[1] else NA_character_
    minute <- if (length(time_components) >= 2) time_components[2] else NA_character_
    second <- if (length(time_components) >= 3) time_components[3] else NA_character_
  } else {
    hour   <- NA_character_
    minute <- NA_character_
    second <- NA_character_
  }

  list(year = year, month = month, day = day,
       hour = hour, minute = minute, second = second)
}

# For each row, get imputation targets and combine with actual partial values
results <- lapply(input$partial, function(p) {
  components <- parse_partial(p)

  # Call get_imputation_targets (internal admiral function)
  # date_imputation = "06-15": year-only -> month=06, day=15
  # time_imputation = "00:00:00": missing time -> 00:00:00
  targets <- admiral:::get_imputation_targets(
    partial          = components,
    date_imputation  = "06-15",
    time_imputation  = "00:00:00"
  )

  # Resolve each component: use partial value if present, else use target
  # (target year is "xxxx" = undefined; year always comes from partial)
  resolve <- function(actual, target) {
    if (!is.na(actual)) actual else target
  }

  year   <- resolve(components$year,   NA_character_)  # year always present in partial
  month  <- resolve(components$month,  targets$month)
  day    <- resolve(components$day,    targets$day)
  hour   <- resolve(components$hour,   targets$hour)
  minute <- resolve(components$minute, targets$minute)
  second <- resolve(components$second, targets$second)

  data.frame(
    partial = p,
    year    = year,
    month   = month,
    day     = day,
    hour    = hour,
    minute  = minute,
    second  = second,
    stringsAsFactors = FALSE
  )
})

result <- do.call(rbind, results)

# Create output directory and write CSV
dir.create("outputs", showWarnings = FALSE)
write.csv(result, "outputs/result.csv", row.names = FALSE)

cat("Done. Output:\n")
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
Error in admiral:::get_imputation_targets(partial = components, date_imputation = "06-15",  : 
  unused arguments (partial = components, date_imputation = "06-15", time_imputation = "00:00:00")
Calls: lapply -> FUN
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

# Read input
input <- read.table("inputs/partial.tsv", header = TRUE, sep = "\t",
                    stringsAsFactors = FALSE, colClasses = "character")

# Function to parse a partial date/time string into named list of components
parse_partial <- function(partial) {
  # Split on 'T' to separate date and time parts
  parts <- strsplit(partial, "T")[[1]]
  date_part <- parts[1]
  time_part  <- if (length(parts) > 1) parts[2] else NA_character_

  # Split date into year, month, day
  date_components <- strsplit(date_part, "-")[[1]]
  year  <- if (length(date_components) >= 1) date_components[1] else NA_character_
  month <- if (length(date_components) >= 2) date_components[2] else NA_character_
  day   <- if (length(date_components) >= 3) date_components[3] else NA_character_

  # Split time into hour, minute, second
  if (!is.na(time_part)) {
    time_components <- strsplit(time_part, ":")[[1]]
    hour   <- if (length(time_components) >= 1) time_components[1] else NA_character_
    minute <- if (length(time_components) >= 2) time_components[2] else NA_character_
    second <- if (length(time_components) >= 3) time_components[3] else NA_character_
  } else {
    hour   <- NA_character_
    minute <- NA_character_
    second <- NA_character_
  }

  list(year = year, month = month, day = day,
       hour = hour, minute = minute, second = second)
}

# For each row, get imputation targets and combine with actual partial values
results <- lapply(input$partial, function(p) {
  components <- parse_partial(p)

  # Call get_imputation_targets (internal admiral function)
  # date_imputation = "06-15": year-only -> month=06, day=15
  # time_imputation = "00:00:00": missing time -> 00:00:00
  targets <- admiral:::get_imputation_targets(
    partial          = components,
    date_imputation  = "06-15",
    time_imputation  = "00:00:00"
  )

  # Resolve each component: use partial value if present, else use target
  # (target year is "xxxx" = undefined; year always comes from partial)
  resolve <- function(actual, target) {
    if (!is.na(actual)) actual else target
  }

  year   <- resolve(components$year,   NA_character_)  # year always present in partial
  month  <- resolve(components$month,  targets$month)
  day    <- resolve(components$day,    targets$day)
  hour   <- resolve(components$hour,   targets$hour)
  minute <- resolve(components$minute, targets$minute)
  second <- resolve(components$second, targets$second)

  data.frame(
    partial = p,
    year    = year,
    month   = month,
    day     = day,
    hour    = hour,
    minute  = minute,
    second  = second,
    stringsAsFactors = FALSE
  )
})

result <- do.call(rbind, results)

# Create output directory and write CSV
dir.create("outputs", showWarnings = FALSE)
write.csv(result, "outputs/result.csv", row.names = FALSE)

cat("Done. Output:\n")
print(result)
```
