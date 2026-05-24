# Case 088: pharmaverse/admiral/convert_xxtpt_to_hours

## Metadata
- Task ID: `pharmaverse/admiral/convert_xxtpt_to_hours`
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

Write an R script for an R package task: perform convert xxtpt to hours using the admiral package. Use functions from the admiral package when suitable.

Input: range_method.tsv, treatment_duration.tsv, xxtpt.tsv
Output: result.csv

Computation: Parse timepoint text into hours: numeric hours stay numeric; HH:MM uses hours plus minutes/60; day counts such as 4D or DAY 4 map to 4 * 24 hours; pre-dose/screening style labels map to 0.


Required columns for result.csv: xxtpt, treatment_duration, range_method, result
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `range_method.tsv`

```text
range_method
midpoint
```

#### `treatment_duration.tsv`

```text
treatment_duration
24
```

#### `xxtpt.tsv`

```text
xxtpt
1 HOURS
2H 30MIN
3 DAYS
DAY 2
4D
```

## Code
### Ground Truth Code

```r
suppressPackageStartupMessages(library(admiral))

xxtpt <- as.character(read.delim(file.path("inputs", "xxtpt.tsv"), check.names = FALSE, stringsAsFactors = FALSE)[[1]])
treatment_duration <- as.numeric(read.delim(file.path("inputs", "treatment_duration.tsv"), check.names = FALSE, stringsAsFactors = FALSE)[[1]][1])
range_method <- as.character(read.delim(file.path("inputs", "range_method.tsv"), check.names = FALSE, stringsAsFactors = FALSE)[[1]][1])

convert_one <- function(x) {
  x_trim <- trimws(toupper(x))
  if (grepl("^[0-9.]+\\s*H(OUR|OURS|R)?$", x_trim)) {
    return(as.numeric(sub("^([0-9.]+).*", "\\1", x_trim)))
  }
  if (grepl("^[0-9.]+\\s*H(OUR|OURS|R)?S?\\s+[0-9.]+\\s*M(IN|INUTE)?S?$", x_trim)) {
    nums <- as.numeric(regmatches(x_trim, gregexpr("[0-9.]+", x_trim))[[1]])
    return(nums[1] + nums[2] / 60)
  }
  if (grepl("^[0-9.]+\\s*D(AY|AYS)?$", x_trim)) {
    return(as.numeric(sub("^([0-9.]+).*", "\\1", x_trim)) * 24)
  }
  if (grepl("^DAY\\s+[0-9.]+$", x_trim)) {
    return(as.numeric(sub("^DAY\\s+([0-9.]+)$", "\\1", x_trim)) * 24)
  }
  if (grepl("^[0-9.]+\\s+POST\\s*DOSE$", x_trim)) {
    return(as.numeric(sub("^([0-9.]+).*", "\\1", x_trim)) * 24)
  }
  NA_real_
}
result <- vapply(xxtpt, convert_one, numeric(1))

dir.create("outputs", showWarnings = FALSE)
write.csv(data.frame(xxtpt = xxtpt, treatment_duration = treatment_duration, range_method = range_method, result = result), file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

read_tsv <- function(path) {
  read.delim(path, sep = "\t", header = TRUE, stringsAsFactors = FALSE, check.names = FALSE)
}

parse_xxtpt_hours <- function(x) {
  text <- toupper(trimws(x))
  text <- gsub("\\s+", " ", text)

  vapply(text, function(value) {
    if (is.na(value) || value == "") {
      return(NA_real_)
    }

    if (grepl("PRE\\s*-?\\s*DOSE|PREDOSE|SCREEN|BASELINE|BEFORE", value)) {
      return(0)
    }

    hhmm <- regexec("^([0-9]+(?:\\.[0-9]+)?):([0-9]+(?:\\.[0-9]+)?)$", value)
    hhmm_match <- regmatches(value, hhmm)[[1]]
    if (length(hhmm_match) == 3) {
      return(as.numeric(hhmm_match[2]) + as.numeric(hhmm_match[3]) / 60)
    }

    day <- regexec("^(?:DAY\\s*([0-9]+(?:\\.[0-9]+)?)|([0-9]+(?:\\.[0-9]+)?)\\s*(?:D|DAY|DAYS))$", value)
    day_match <- regmatches(value, day)[[1]]
    if (length(day_match) >= 3) {
      day_value <- day_match[day_match != ""][2]
      return(as.numeric(day_value) * 24)
    }

    hour_min <- regexec(
      "^([0-9]+(?:\\.[0-9]+)?)\\s*(?:H|HR|HRS|HOUR|HOURS)(?:\\s*([0-9]+(?:\\.[0-9]+)?)\\s*(?:M|MIN|MINS|MINUTE|MINUTES))?$",
      value
    )
    hour_min_match <- regmatches(value, hour_min)[[1]]
    if (length(hour_min_match) >= 2) {
      minutes <- if (length(hour_min_match) >= 3 && nzchar(hour_min_match[3])) {
        as.numeric(hour_min_match[3]) / 60
      } else {
        0
      }
      return(as.numeric(hour_min_match[2]) + minutes)
    }

    numeric_value <- suppressWarnings(as.numeric(value))
    if (!is.na(numeric_value)) {
      return(numeric_value)
    }

    NA_real_
  }, numeric(1), USE.NAMES = FALSE)
}

xxtpt <- read_tsv("inputs/xxtpt.tsv")
treatment_duration <- read_tsv("inputs/treatment_duration.tsv")
range_method <- read_tsv("inputs/range_method.tsv")

result <- data.frame(
  xxtpt = xxtpt$xxtpt,
  treatment_duration = treatment_duration$treatment_duration[1],
  range_method = range_method$range_method[1],
  result = parse_xxtpt_hours(xxtpt$xxtpt),
  check.names = FALSE
)

dir.create("outputs", showWarnings = FALSE, recursive = TRUE)
write.csv(result, "outputs/result.csv", row.names = FALSE, na = "")
```

## Output
### Ground Truth Output
#### `result.csv`

```csv
"xxtpt","treatment_duration","range_method","result"
"1 HOURS",24,"midpoint",1
"2H 30MIN",24,"midpoint",2.5
"3 DAYS",24,"midpoint",72
"DAY 2",24,"midpoint",48
"4D",24,"midpoint",96
```

### LLM Output
#### `result.csv`

```csv
"xxtpt","treatment_duration","range_method","result"
"1 HOURS",24,"midpoint",1
"2H 30MIN",24,"midpoint",2.5
"3 DAYS",24,"midpoint",72
"DAY 2",24,"midpoint",48
"4D",24,"midpoint",96
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

read_tsv <- function(path) {
  read.delim(path, sep = "\t", header = TRUE, stringsAsFactors = FALSE, check.names = FALSE)
}

parse_xxtpt_hours <- function(x) {
  text <- toupper(trimws(x))
  text <- gsub("\\s+", " ", text)

  vapply(text, function(value) {
    if (is.na(value) || value == "") {
      return(NA_real_)
    }

    if (grepl("PRE\\s*-?\\s*DOSE|PREDOSE|SCREEN|BASELINE|BEFORE", value)) {
      return(0)
    }

    hhmm <- regexec("^([0-9]+(?:\\.[0-9]+)?):([0-9]+(?:\\.[0-9]+)?)$", value)
    hhmm_match <- regmatches(value, hhmm)[[1]]
    if (length(hhmm_match) == 3) {
      return(as.numeric(hhmm_match[2]) + as.numeric(hhmm_match[3]) / 60)
    }

    day <- regexec("^(?:DAY\\s*([0-9]+(?:\\.[0-9]+)?)|([0-9]+(?:\\.[0-9]+)?)\\s*(?:D|DAY|DAYS))$", value)
    day_match <- regmatches(value, day)[[1]]
    if (length(day_match) >= 3) {
      day_value <- day_match[day_match != ""][2]
      return(as.numeric(day_value) * 24)
    }

    hour_min <- regexec(
      "^([0-9]+(?:\\.[0-9]+)?)\\s*(?:H|HR|HRS|HOUR|HOURS)(?:\\s*([0-9]+(?:\\.[0-9]+)?)\\s*(?:M|MIN|MINS|MINUTE|MINUTES))?$",
      value
    )
    hour_min_match <- regmatches(value, hour_min)[[1]]
    if (length(hour_min_match) >= 2) {
      minutes <- if (length(hour_min_match) >= 3 && nzchar(hour_min_match[3])) {
        as.numeric(hour_min_match[3]) / 60
      } else {
        0
      }
      return(as.numeric(hour_min_match[2]) + minutes)
    }

    numeric_value <- suppressWarnings(as.numeric(value))
    if (!is.na(numeric_value)) {
      return(numeric_value)
    }

    NA_real_
  }, numeric(1), USE.NAMES = FALSE)
}

xxtpt <- read_tsv("inputs/xxtpt.tsv")
treatment_duration <- read_tsv("inputs/treatment_duration.tsv")
range_method <- read_tsv("inputs/range_method.tsv")

result <- data.frame(
  xxtpt = xxtpt$xxtpt,
  treatment_duration = treatment_duration$treatment_duration[1],
  range_method = range_method$range_method[1],
  result = parse_xxtpt_hours(xxtpt$xxtpt),
  check.names = FALSE
)

dir.create("outputs", showWarnings = FALSE, recursive = TRUE)
write.csv(result, "outputs/result.csv", row.names = FALSE, na = "")
```
