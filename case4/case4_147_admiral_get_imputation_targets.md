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
- Do not infer package function names from task metadata. Use a package API only when
  it is a normal exported function you can verify; otherwise implement the required
  transformation directly from the inputs.
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
#!/usr/bin/env Rscript

input <- read.delim(
  file.path("inputs", "partial.tsv"),
  stringsAsFactors = FALSE,
  colClasses = "character"
)

parse_partial <- function(value) {
  match <- regexec(
    "^(\\d{4})(?:-(\\d{2})(?:-(\\d{2})(?:T(\\d{2})(?::(\\d{2})(?::(\\d{2}(?:\\.\\d+)?))?)?)?)?)?$",
    value
  )
  pieces <- regmatches(value, match)[[1]]

  if (length(pieces) == 0) {
    stop(sprintf("Unsupported partial date/time value: %s", value), call. = FALSE)
  }

  component <- function(index, default) {
    if (length(pieces) >= index && nzchar(pieces[[index]])) {
      pieces[[index]]
    } else {
      default
    }
  }

  data.frame(
    year = component(2, NA_character_),
    month = component(3, "06"),
    day = component(4, "15"),
    hour = component(5, "00"),
    minute = component(6, "00"),
    second = component(7, "00"),
    stringsAsFactors = FALSE
  )
}

components <- do.call(rbind, lapply(input$partial, parse_partial))
result <- cbind(partial = input$partial, components)

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
#!/usr/bin/env Rscript

input <- read.delim(
  file.path("inputs", "partial.tsv"),
  stringsAsFactors = FALSE,
  colClasses = "character"
)

parse_partial <- function(value) {
  match <- regexec(
    "^(\\d{4})(?:-(\\d{2})(?:-(\\d{2})(?:T(\\d{2})(?::(\\d{2})(?::(\\d{2}(?:\\.\\d+)?))?)?)?)?)?$",
    value
  )
  pieces <- regmatches(value, match)[[1]]

  if (length(pieces) == 0) {
    stop(sprintf("Unsupported partial date/time value: %s", value), call. = FALSE)
  }

  component <- function(index, default) {
    if (length(pieces) >= index && nzchar(pieces[[index]])) {
      pieces[[index]]
    } else {
      default
    }
  }

  data.frame(
    year = component(2, NA_character_),
    month = component(3, "06"),
    day = component(4, "15"),
    hour = component(5, "00"),
    minute = component(6, "00"),
    second = component(7, "00"),
    stringsAsFactors = FALSE
  )
}

components <- do.call(rbind, lapply(input$partial, parse_partial))
result <- cbind(partial = input$partial, components)

dir.create("outputs", showWarnings = FALSE, recursive = TRUE)
write.csv(result, file.path("outputs", "result.csv"), row.names = FALSE, quote = FALSE)
```
