# Case 147: pharmaverse/admiral/get_imputation_targets

## Metadata
- Task ID: `pharmaverse/admiral/get_imputation_targets`
- Package: `admiral`
- Model: `codex/gpt-5.5`
- Agent: `Codex CLI`
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
input <- read.delim(
  file.path("inputs", "partial.tsv"),
  stringsAsFactors = FALSE,
  colClasses = c(partial = "character")
)

get_component <- function(parts, index, default) {
  value <- if (length(parts) >= index) parts[[index]] else NA_character_
  if (is.na(value) || identical(value, "")) default else value
}

parse_partial <- function(partial) {
  date_time <- strsplit(partial, "T", fixed = TRUE)[[1]]
  date_part <- date_time[[1]]
  time_part <- if (length(date_time) >= 2) date_time[[2]] else ""

  date_parts <- strsplit(date_part, "-", fixed = TRUE)[[1]]
  time_parts <- if (nzchar(time_part)) {
    strsplit(time_part, ":", fixed = TRUE)[[1]]
  } else {
    character(0)
  }

  data.frame(
    year = get_component(date_parts, 1, NA_character_),
    month = get_component(date_parts, 2, "06"),
    day = get_component(date_parts, 3, "15"),
    hour = get_component(time_parts, 1, "00"),
    minute = get_component(time_parts, 2, "00"),
    second = get_component(time_parts, 3, "00"),
    stringsAsFactors = FALSE
  )
}

components <- do.call(rbind, lapply(input$partial, parse_partial))
result <- cbind(partial = input$partial, components, stringsAsFactors = FALSE)

dir.create("outputs", showWarnings = FALSE, recursive = TRUE)
write.csv(result, file.path("outputs", "result.csv"), row.names = FALSE, quote = FALSE)
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
partial,year,month,day,hour,minute,second
2020,2020,06,15,00,00,00
2020-02,2020,02,15,00,00,00
2020-02-20T14,2020,02,20,14,00,00
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
input <- read.delim(
  file.path("inputs", "partial.tsv"),
  stringsAsFactors = FALSE,
  colClasses = c(partial = "character")
)

get_component <- function(parts, index, default) {
  value <- if (length(parts) >= index) parts[[index]] else NA_character_
  if (is.na(value) || identical(value, "")) default else value
}

parse_partial <- function(partial) {
  date_time <- strsplit(partial, "T", fixed = TRUE)[[1]]
  date_part <- date_time[[1]]
  time_part <- if (length(date_time) >= 2) date_time[[2]] else ""

  date_parts <- strsplit(date_part, "-", fixed = TRUE)[[1]]
  time_parts <- if (nzchar(time_part)) {
    strsplit(time_part, ":", fixed = TRUE)[[1]]
  } else {
    character(0)
  }

  data.frame(
    year = get_component(date_parts, 1, NA_character_),
    month = get_component(date_parts, 2, "06"),
    day = get_component(date_parts, 3, "15"),
    hour = get_component(time_parts, 1, "00"),
    minute = get_component(time_parts, 2, "00"),
    second = get_component(time_parts, 3, "00"),
    stringsAsFactors = FALSE
  )
}

components <- do.call(rbind, lapply(input$partial, parse_partial))
result <- cbind(partial = input$partial, components, stringsAsFactors = FALSE)

dir.create("outputs", showWarnings = FALSE, recursive = TRUE)
write.csv(result, file.path("outputs", "result.csv"), row.names = FALSE, quote = FALSE)
```
