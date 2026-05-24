# Case 001: pharmaverse/aNCA/PKNCA_impute_method_start_c1

## Metadata
- Task ID: `pharmaverse/aNCA/PKNCA_impute_method_start_c1`
- Package: `aNCA`
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
- Read input files only from `inputs/` using relative paths.
- Write exactly the required output artifact(s): outputs/result.csv.
- Create `outputs/` if needed.
- You may inspect `task.json`, `TASK.md`, and input files.
- Do not modify `inputs/`, `task.json`, `AGENTS.md`, or hidden evaluator metadata.
- Do not use files outside this worktree.
- Do not commit changes.
- Before finishing, run `Rscript solution.R` if R is available and fix any concrete error.

Task prompt:

Write an R script for an R package task: perform PKNCA impute method start c1 using the aNCA package. Use functions from the aNCA package when suitable.

Input: conc.tsv, end.tsv, options.tsv, start.tsv, time.tsv
Output: result.csv

Computation: Insert a start-time row at the requested start time using the first concentration after dose, then write rows ordered by time to result.csv.


Required columns for result.csv: conc, time
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `conc.tsv`

```text
conc
0.1
1.5
10.0
100.0
1000.0
```

#### `end.tsv`

```text
end
4.5
```

#### `options.tsv`

```text
options
method=start_conc0
allow_blq=TRUE
```

#### `start.tsv`

```text
start
1.5
```

#### `time.tsv`

```text
time
1.0
2.0
3.0
4.0
5.0
```

## Code
### Ground Truth Code

```r
read_tsv <- function(name) {
  path <- file.path("inputs", name)
  if (!file.exists(path)) {
    stop(name, " is required")
  }
  read.delim(path, check.names = FALSE, stringsAsFactors = FALSE)
}

first_column <- function(df) {
  if (ncol(df) == 0) {
    return(character())
  }
  df[[1]]
}

scalar_numeric <- function(df) {
  values <- suppressWarnings(as.numeric(first_column(df)))
  values <- values[!is.na(values)]
  if (length(values) == 0) NA_real_ else values[[1]]
}

write_result <- function(result) {
  dir.create("outputs", showWarnings = FALSE, recursive = TRUE)
  write.csv(result, file.path("outputs", "result.csv"), row.names = FALSE, na = "NA")
}

conc <- suppressWarnings(as.numeric(first_column(read_tsv("conc.tsv"))))
time <- suppressWarnings(as.numeric(first_column(read_tsv("time.tsv"))))
start <- scalar_numeric(read_tsv("start.tsv"))
end <- scalar_numeric(read_tsv("end.tsv"))

keep <- !is.na(conc) & !is.na(time)
result <- data.frame(conc = conc[keep], time = time[keep])

if (!is.na(start) && !is.na(end) && !any(result$time == start)) {
  interval <- result[result$time >= start & result$time <= end, , drop = FALSE]
  if (nrow(interval) > 0) {
    interval <- interval[order(interval$time), , drop = FALSE]
    result <- rbind(result, data.frame(conc = interval$conc[[1]], time = start))
    result <- result[order(result$time), , drop = FALSE]
  }
}

write_result(result)
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

read_numeric_column <- function(path, column) {
  dat <- read.delim(path, stringsAsFactors = FALSE, check.names = FALSE)
  if (!column %in% names(dat)) {
    stop(sprintf("Expected column '%s' in %s", column, path), call. = FALSE)
  }
  value <- suppressWarnings(as.numeric(dat[[column]]))
  if (anyNA(value)) {
    stop(sprintf("Column '%s' in %s must be numeric", column, path), call. = FALSE)
  }
  value
}

parse_options <- function(path) {
  dat <- read.delim(path, stringsAsFactors = FALSE, check.names = FALSE)
  if (!"options" %in% names(dat)) {
    stop(sprintf("Expected column 'options' in %s", path), call. = FALSE)
  }

  entries <- strsplit(dat$options, "=", fixed = TRUE)
  opts <- list()
  for (entry in entries) {
    if (length(entry) != 2L || !nzchar(entry[[1]])) {
      stop("Options must be supplied as key=value rows", call. = FALSE)
    }
    key <- entry[[1]]
    raw_value <- entry[[2]]
    value <- type.convert(raw_value, as.is = TRUE)
    opts[[key]] <- value
  }
  opts
}

impute_start_c1 <- function(conc, time, start, end, options = list()) {
  if (requireNamespace("aNCA", quietly = TRUE)) {
    suppressPackageStartupMessages(library(aNCA))
    return(PKNCA_impute_method_start_c1(
      conc = conc,
      time = time,
      start = start,
      end = end,
      options = options
    ))
  }

  result <- data.frame(conc = conc, time = time)
  if (!start %in% time) {
    in_interval <- time >= start & time <= end
    interval_conc <- conc[in_interval]
    interval_time <- time[in_interval]
    if (!all(is.na(interval_conc)) && length(interval_conc) > 0L) {
      c1 <- interval_conc[which.min(interval_time)]
      result <- rbind(result, data.frame(conc = c1, time = start))
      result <- result[order(result$time), , drop = FALSE]
    }
  }
  row.names(result) <- NULL
  result
}

conc <- read_numeric_column("inputs/conc.tsv", "conc")
time <- read_numeric_column("inputs/time.tsv", "time")
start <- read_numeric_column("inputs/start.tsv", "start")
end <- read_numeric_column("inputs/end.tsv", "end")
options <- parse_options("inputs/options.tsv")

if (length(conc) != length(time)) {
  stop("conc and time must have the same length", call. = FALSE)
}
if (length(start) != 1L || length(end) != 1L) {
  stop("start and end must each contain exactly one value", call. = FALSE)
}

result <- impute_start_c1(
  conc = conc,
  time = time,
  start = start[[1]],
  end = end[[1]],
  options = options
)

result <- result[, c("conc", "time"), drop = FALSE]
result <- result[order(result$time), , drop = FALSE]
row.names(result) <- NULL

dir.create("outputs", showWarnings = FALSE, recursive = TRUE)
write.csv(result, "outputs/result.csv", row.names = FALSE)
```

## Output
### Ground Truth Output
#### `result.csv`

```csv
"conc","time"
0.1,1
1.5,1.5
1.5,2
10,3
100,4
1000,5
```

### LLM Output
#### `result.csv`

```csv
"conc","time"
0.1,1
1.5,1.5
1.5,2
10,3
100,4
1000,5
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
Registered S3 method overwritten by 'tern':
  method   from 
  tidy.glm broom
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

read_numeric_column <- function(path, column) {
  dat <- read.delim(path, stringsAsFactors = FALSE, check.names = FALSE)
  if (!column %in% names(dat)) {
    stop(sprintf("Expected column '%s' in %s", column, path), call. = FALSE)
  }
  value <- suppressWarnings(as.numeric(dat[[column]]))
  if (anyNA(value)) {
    stop(sprintf("Column '%s' in %s must be numeric", column, path), call. = FALSE)
  }
  value
}

parse_options <- function(path) {
  dat <- read.delim(path, stringsAsFactors = FALSE, check.names = FALSE)
  if (!"options" %in% names(dat)) {
    stop(sprintf("Expected column 'options' in %s", path), call. = FALSE)
  }

  entries <- strsplit(dat$options, "=", fixed = TRUE)
  opts <- list()
  for (entry in entries) {
    if (length(entry) != 2L || !nzchar(entry[[1]])) {
      stop("Options must be supplied as key=value rows", call. = FALSE)
    }
    key <- entry[[1]]
    raw_value <- entry[[2]]
    value <- type.convert(raw_value, as.is = TRUE)
    opts[[key]] <- value
  }
  opts
}

impute_start_c1 <- function(conc, time, start, end, options = list()) {
  if (requireNamespace("aNCA", quietly = TRUE)) {
    suppressPackageStartupMessages(library(aNCA))
    return(PKNCA_impute_method_start_c1(
      conc = conc,
      time = time,
      start = start,
      end = end,
      options = options
    ))
  }

  result <- data.frame(conc = conc, time = time)
  if (!start %in% time) {
    in_interval <- time >= start & time <= end
    interval_conc <- conc[in_interval]
    interval_time <- time[in_interval]
    if (!all(is.na(interval_conc)) && length(interval_conc) > 0L) {
      c1 <- interval_conc[which.min(interval_time)]
      result <- rbind(result, data.frame(conc = c1, time = start))
      result <- result[order(result$time), , drop = FALSE]
    }
  }
  row.names(result) <- NULL
  result
}

conc <- read_numeric_column("inputs/conc.tsv", "conc")
time <- read_numeric_column("inputs/time.tsv", "time")
start <- read_numeric_column("inputs/start.tsv", "start")
end <- read_numeric_column("inputs/end.tsv", "end")
options <- parse_options("inputs/options.tsv")

if (length(conc) != length(time)) {
  stop("conc and time must have the same length", call. = FALSE)
}
if (length(start) != 1L || length(end) != 1L) {
  stop("start and end must each contain exactly one value", call. = FALSE)
}

result <- impute_start_c1(
  conc = conc,
  time = time,
  start = start[[1]],
  end = end[[1]],
  options = options
)

result <- result[, c("conc", "time"), drop = FALSE]
result <- result[order(result$time), , drop = FALSE]
row.names(result) <- NULL

dir.create("outputs", showWarnings = FALSE, recursive = TRUE)
write.csv(result, "outputs/result.csv", row.names = FALSE)
```
