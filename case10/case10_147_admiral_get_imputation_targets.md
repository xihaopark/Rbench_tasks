# Case 147: pharmaverse/admiral/get_imputation_targets

## Metadata
- Task ID: `pharmaverse/admiral/get_imputation_targets`
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

Reference package function list:
The hidden reference solution's R package function calls are listed below. If the list is empty, the reference solution does not call package functions.
- package_functions: []
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
#!/usr/bin/env Rscript

dir.create("outputs", showWarnings = FALSE, recursive = TRUE)

df <- read.delim("inputs/partial.tsv", stringsAsFactors = FALSE)

parse_partial <- function(x) {
  parts <- strsplit(x, "[-T:]")[[1]]
  n <- length(parts)

  year   <- as.integer(parts[1])
  month  <- if (n >= 2) as.integer(parts[2]) else 6L
  day    <- if (n >= 3) as.integer(parts[3]) else 15L
  hour   <- if (n >= 4) as.integer(parts[4]) else 0L
  minute <- if (n >= 5) as.integer(parts[5]) else 0L
  second <- if (n >= 6) as.integer(parts[6]) else 0L

  # year-only: impute month=06, day=15
  if (n == 1) {
    month <- 6L
    day   <- 15L
  }

  data.frame(year = year, month = month, day = day,
             hour = hour, minute = minute, second = second,
             stringsAsFactors = FALSE)
}

rows <- do.call(rbind, lapply(df$partial, parse_partial))
result <- cbind(partial = df$partial, rows)

write.csv(result, "outputs/result.csv", row.names = FALSE)
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
#### `result.csv`

```csv
"partial","year","month","day","hour","minute","second"
"2020",2020,6,15,0,0,0
"2020-02",2020,2,15,0,0,0
"2020-02-20T14",2020,2,20,14,0,0
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
#!/usr/bin/env Rscript

dir.create("outputs", showWarnings = FALSE, recursive = TRUE)

df <- read.delim("inputs/partial.tsv", stringsAsFactors = FALSE)

parse_partial <- function(x) {
  parts <- strsplit(x, "[-T:]")[[1]]
  n <- length(parts)

  year   <- as.integer(parts[1])
  month  <- if (n >= 2) as.integer(parts[2]) else 6L
  day    <- if (n >= 3) as.integer(parts[3]) else 15L
  hour   <- if (n >= 4) as.integer(parts[4]) else 0L
  minute <- if (n >= 5) as.integer(parts[5]) else 0L
  second <- if (n >= 6) as.integer(parts[6]) else 0L

  # year-only: impute month=06, day=15
  if (n == 1) {
    month <- 6L
    day   <- 15L
  }

  data.frame(year = year, month = month, day = day,
             hour = hour, minute = minute, second = second,
             stringsAsFactors = FALSE)
}

rows <- do.call(rbind, lapply(df$partial, parse_partial))
result <- cbind(partial = df$partial, rows)

write.csv(result, "outputs/result.csv", row.names = FALSE)
```
