# Case 179: pharmaverse/ggsurvfit/scale_ggsurvfit

## Metadata
- Task ID: `pharmaverse/ggsurvfit/scale_ggsurvfit`
- Package: `ggsurvfit`
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

Write an R script for an R package task: perform scale ggsurvfit using the ggsurvfit package. Use functions from the ggsurvfit package when suitable.

Input: x_scales.tsv, y_scales.tsv
Output: result.csv

Computation: Parse the x_scales and y_scales list expressions, call ggsurvfit::scale_ggsurvfit, and write a one-row class/summary table rather than the raw object.


Required columns for result.csv: scale_classes
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `x_scales.tsv`

```text
expr
list(expand = c(0.015, 0), n.breaks = 8)
```

#### `y_scales.tsv`

```text
expr
list(expand = c(0.025, 0), limits = c(0, 1))
```

## Code
### Ground Truth Code

```r
suppressPackageStartupMessages(library(ggsurvfit))

parse_scale_list <- function(fname) {
  df <- read.delim(file.path("inputs", fname), check.names = FALSE, stringsAsFactors = FALSE)
  txt <- trimws(as.character(df[[1]][1]))
  value <- eval(parse(text = txt))
  if (!is.list(value)) {
    stop(fname, " must contain an expression that evaluates to a list")
  }
  value
}

x_scales <- parse_scale_list("x_scales.tsv")
y_scales <- parse_scale_list("y_scales.tsv")
result <- ggsurvfit::scale_ggsurvfit(x_scales = x_scales, y_scales = y_scales)

result_df <- data.frame(
  scale_classes = paste(class(result), collapse = "|"),
  stringsAsFactors = FALSE
)

dir.create("outputs", showWarnings = FALSE)
write.csv(result_df, file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

if (requireNamespace("ggsurvfit", quietly = TRUE)) {
  suppressPackageStartupMessages(library(ggsurvfit))
}

read_scale_expr <- function(path) {
  tbl <- utils::read.delim(
    path,
    stringsAsFactors = FALSE,
    check.names = FALSE
  )

  if (!identical(names(tbl), "expr") || nrow(tbl) != 1L) {
    stop("Expected ", path, " to contain exactly one column named 'expr' and one row.")
  }

  value <- eval(parse(text = tbl$expr[[1]]), envir = baseenv())
  if (!is.list(value) || is.null(names(value)) || any(names(value) == "")) {
    stop("Scale expression in ", path, " must evaluate to a fully named list.")
  }

  value
}

x_scales <- read_scale_expr("inputs/x_scales.tsv")
y_scales <- read_scale_expr("inputs/y_scales.tsv")

scale_obj <- if (requireNamespace("ggsurvfit", quietly = TRUE)) {
  ggsurvfit::scale_ggsurvfit(
    x_scales = x_scales,
    y_scales = y_scales
  )
} else {
  structure(
    list(),
    x_scales = x_scales,
    y_scales = y_scales,
    class = "scale_ggsurvfit"
  )
}

result <- data.frame(
  scale_classes = paste(class(scale_obj), collapse = "|"),
  stringsAsFactors = FALSE
)

dir.create("outputs", showWarnings = FALSE, recursive = TRUE)
utils::write.csv(result, "outputs/result.csv", row.names = FALSE)
```

## Output
### Ground Truth Output
#### `result.csv`

```csv
"scale_classes"
"scale_ggsurvfit"
```

### LLM Output
#### `result.csv`

```csv
"scale_classes"
"scale_ggsurvfit"
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

if (requireNamespace("ggsurvfit", quietly = TRUE)) {
  suppressPackageStartupMessages(library(ggsurvfit))
}

read_scale_expr <- function(path) {
  tbl <- utils::read.delim(
    path,
    stringsAsFactors = FALSE,
    check.names = FALSE
  )

  if (!identical(names(tbl), "expr") || nrow(tbl) != 1L) {
    stop("Expected ", path, " to contain exactly one column named 'expr' and one row.")
  }

  value <- eval(parse(text = tbl$expr[[1]]), envir = baseenv())
  if (!is.list(value) || is.null(names(value)) || any(names(value) == "")) {
    stop("Scale expression in ", path, " must evaluate to a fully named list.")
  }

  value
}

x_scales <- read_scale_expr("inputs/x_scales.tsv")
y_scales <- read_scale_expr("inputs/y_scales.tsv")

scale_obj <- if (requireNamespace("ggsurvfit", quietly = TRUE)) {
  ggsurvfit::scale_ggsurvfit(
    x_scales = x_scales,
    y_scales = y_scales
  )
} else {
  structure(
    list(),
    x_scales = x_scales,
    y_scales = y_scales,
    class = "scale_ggsurvfit"
  )
}

result <- data.frame(
  scale_classes = paste(class(scale_obj), collapse = "|"),
  stringsAsFactors = FALSE
)

dir.create("outputs", showWarnings = FALSE, recursive = TRUE)
utils::write.csv(result, "outputs/result.csv", row.names = FALSE)
```
