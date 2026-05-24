# Case 087: pharmaverse/admiral/convert_treatment_patterns

## Metadata
- Task ID: `pharmaverse/admiral/convert_treatment_patterns`
- Package: `admiral`
- Model: `codex/gpt-5.5`
- Agent: `Codex CLI`
- Agent command: `codex exec -C <worktree> --dangerously-bypass-approvals-and-sandbox <prompt> -m gpt-5.5`
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

Write an R script for an R package task: perform convert treatment patterns using the admiral package. Use functions from the admiral package when suitable.

Input: treatment_duration.tsv, xxtpt.tsv
Output: result.csv

Computation: Map treatment-relative text patterns deterministically: treatment start/zero/pre-dose to 0, after-start to positive offsets, before-end/pre-EOT to negative offsets from end, and end/after-end patterns to the treatment end plus any offset.


Required columns for result.csv: xxtpt, treatment_duration, result
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.

## Input preview

### treatment_duration.tsv
treatment_duration
72
72
72
72
72
... [2 more lines omitted]

### xxtpt.tsv
xxtpt
START OF TREATMENT
PRE-DOSE
30 MIN AFTER START
2 HOURS AFTER START
END OF TREATMENT
... [2 more lines omitted]
```

## Input
#### `treatment_duration.tsv`

```text
treatment_duration
72
72
72
72
72
72
72
```

#### `xxtpt.tsv`

```text
xxtpt
START OF TREATMENT
PRE-DOSE
30 MIN AFTER START
2 HOURS AFTER START
END OF TREATMENT
4 HOURS BEFORE END
6 HOURS AFTER END
```

## Code
### Ground Truth Code

```r
xxtpt <- as.character(read.delim(file.path("inputs", "xxtpt.tsv"), check.names = FALSE, stringsAsFactors = FALSE)$xxtpt)
treatment_duration <- as.numeric(read.delim(file.path("inputs", "treatment_duration.tsv"), check.names = FALSE, stringsAsFactors = FALSE)$treatment_duration)

parse_hours <- function(label, duration) {
  text <- toupper(trimws(label))
  number <- suppressWarnings(as.numeric(sub(".*?([0-9]+(?:\\.[0-9]+)?).*", "\\1", text)))
  if (text == "START OF TREATMENT") {
    0
  } else if (text == "PRE-DOSE") {
    -1
  } else if (grepl("MIN AFTER START", text)) {
    number / 60
  } else if (grepl("HOURS? AFTER START", text)) {
    number
  } else if (text == "END OF TREATMENT") {
    duration
  } else if (grepl("HOURS? BEFORE END", text)) {
    duration - number
  } else if (grepl("HOURS? AFTER END", text)) {
    duration + number
  } else {
    NA_real_
  }
}

result <- mapply(parse_hours, xxtpt, treatment_duration)
result_df <- data.frame(xxtpt = xxtpt, treatment_duration = treatment_duration, result = as.numeric(result), stringsAsFactors = FALSE)

dir.create("outputs", showWarnings = FALSE)
write.csv(result_df, file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

library(admiral)

read_required_tsv <- function(path) {
  if (!file.exists(path)) {
    stop("Missing required input file: ", path, call. = FALSE)
  }
  utils::read.delim(path, stringsAsFactors = FALSE, check.names = FALSE)
}

parse_hours <- function(value, unit) {
  value <- as.numeric(value)
  unit <- tolower(unit)
  ifelse(substr(unit, 1, 1) == "m", value / 60, value)
}

xxtpt_data <- read_required_tsv(file.path("inputs", "xxtpt.tsv"))
duration_data <- read_required_tsv(file.path("inputs", "treatment_duration.tsv"))

if (!"xxtpt" %in% names(xxtpt_data)) {
  stop("inputs/xxtpt.tsv must contain column xxtpt", call. = FALSE)
}
if (!"treatment_duration" %in% names(duration_data)) {
  stop("inputs/treatment_duration.tsv must contain column treatment_duration", call. = FALSE)
}
if (nrow(xxtpt_data) != nrow(duration_data)) {
  stop("Input files must have the same number of rows", call. = FALSE)
}

xxtpt <- trimws(xxtpt_data$xxtpt)
treatment_duration <- as.numeric(duration_data$treatment_duration)
if (any(is.na(treatment_duration))) {
  stop("treatment_duration must be numeric for all rows", call. = FALSE)
}
if (any(treatment_duration < 0, na.rm = TRUE)) {
  stop("treatment_duration must be non-negative", call. = FALSE)
}

na_idx <- is.na(xxtpt)
result <- rep(NA_real_, length(xxtpt))

convert_treatment_patterns <- getFromNamespace("convert_treatment_patterns", "admiral")
result <- convert_treatment_patterns(
  xxtpt = xxtpt,
  result = result,
  na_idx = na_idx,
  treatment_duration = treatment_duration
)

fill_pattern <- function(pattern, value_fun) {
  matches <- regexec(pattern, xxtpt, ignore.case = TRUE, perl = TRUE)
  parts <- regmatches(xxtpt, matches)
  matched <- lengths(parts) > 0 & is.na(result) & !na_idx
  if (any(matched)) {
    result[matched] <<- value_fun(parts[matched], matched)
  }
}

zero_pattern <- paste0(
  "^\\s*(?:",
  "start\\s+of\\s+(?:treatment|infusion)|",
  "start\\s+(?:treatment|infusion)|",
  "pre\\s*-?\\s*dose|",
  "predose|",
  "0\\s*(?:h|hr|hour|m|min|minute)?s?\\s*(?:post\\s*)?(?:dose|start)?",
  ")\\s*$"
)
result[grepl(zero_pattern, xxtpt, ignore.case = TRUE, perl = TRUE) &
  is.na(result) & !na_idx] <- 0

end_pattern <- "^\\s*(?:end\\s+of\\s+(?:treatment|infusion)|eot|eoi)\\s*$"
result[grepl(end_pattern, xxtpt, ignore.case = TRUE, perl = TRUE) &
  is.na(result) & !na_idx] <- treatment_duration[
    grepl(end_pattern, xxtpt, ignore.case = TRUE, perl = TRUE) &
      is.na(result) & !na_idx
  ]

time_unit <- "(\\d+(?:\\.\\d+)?)\\s*(m(?:in|inute)?|h(?:r|our)?)s?"

fill_pattern(
  paste0("^\\s*", time_unit, "\\s+(?:after|post)\\s+(?:start|start\\s+of\\s+(?:treatment|infusion)|start\\s+inf)\\s*$"),
  function(parts, matched) {
    vapply(parts, function(x) parse_hours(x[2], x[3]), numeric(1))
  }
)

fill_pattern(
  paste0("^\\s*", time_unit, "\\s+(?:before|pre|prior\\s+to)\\s+(?:end|end\\s+of\\s+(?:treatment|infusion)|eot|eoi)\\s*$"),
  function(parts, matched) {
    treatment_duration[matched] - vapply(parts, function(x) parse_hours(x[2], x[3]), numeric(1))
  }
)

fill_pattern(
  paste0("^\\s*", time_unit, "\\s+(?:after|post)\\s+(?:end|end\\s+of\\s+(?:treatment|infusion)|eot|eoi)\\s*$"),
  function(parts, matched) {
    treatment_duration[matched] + vapply(parts, function(x) parse_hours(x[2], x[3]), numeric(1))
  }
)

if (any(is.na(result) & !na_idx)) {
  stop(
    "Unconverted treatment timepoint pattern(s): ",
    paste(unique(xxtpt[is.na(result) & !na_idx]), collapse = "; "),
    call. = FALSE
  )
}

output <- data.frame(
  xxtpt = xxtpt,
  treatment_duration = treatment_duration,
  result = result,
  check.names = FALSE
)

dir.create("outputs", showWarnings = FALSE, recursive = TRUE)
utils::write.csv(output, file.path("outputs", "result.csv"), row.names = FALSE, na = "")
```

## Output
### Ground Truth Output
#### `result.csv`

```csv
"xxtpt","treatment_duration","result"
"START OF TREATMENT",72,0
"PRE-DOSE",72,-1
"30 MIN AFTER START",72,0.5
"2 HOURS AFTER START",72,2
"END OF TREATMENT",72,72
"4 HOURS BEFORE END",72,68
"6 HOURS AFTER END",72,78
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
Error in convert_treatment_patterns(xxtpt = xxtpt, result = result, na_idx = na_idx,  : 
  unused arguments (xxtpt = xxtpt, result = result, na_idx = na_idx, treatment_duration = treatment_duration)
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
#!/usr/bin/env Rscript

library(admiral)

read_required_tsv <- function(path) {
  if (!file.exists(path)) {
    stop("Missing required input file: ", path, call. = FALSE)
  }
  utils::read.delim(path, stringsAsFactors = FALSE, check.names = FALSE)
}

parse_hours <- function(value, unit) {
  value <- as.numeric(value)
  unit <- tolower(unit)
  ifelse(substr(unit, 1, 1) == "m", value / 60, value)
}

xxtpt_data <- read_required_tsv(file.path("inputs", "xxtpt.tsv"))
duration_data <- read_required_tsv(file.path("inputs", "treatment_duration.tsv"))

if (!"xxtpt" %in% names(xxtpt_data)) {
  stop("inputs/xxtpt.tsv must contain column xxtpt", call. = FALSE)
}
if (!"treatment_duration" %in% names(duration_data)) {
  stop("inputs/treatment_duration.tsv must contain column treatment_duration", call. = FALSE)
}
if (nrow(xxtpt_data) != nrow(duration_data)) {
  stop("Input files must have the same number of rows", call. = FALSE)
}

xxtpt <- trimws(xxtpt_data$xxtpt)
treatment_duration <- as.numeric(duration_data$treatment_duration)
if (any(is.na(treatment_duration))) {
  stop("treatment_duration must be numeric for all rows", call. = FALSE)
}
if (any(treatment_duration < 0, na.rm = TRUE)) {
  stop("treatment_duration must be non-negative", call. = FALSE)
}

na_idx <- is.na(xxtpt)
result <- rep(NA_real_, length(xxtpt))

convert_treatment_patterns <- getFromNamespace("convert_treatment_patterns", "admiral")
result <- convert_treatment_patterns(
  xxtpt = xxtpt,
  result = result,
  na_idx = na_idx,
  treatment_duration = treatment_duration
)

fill_pattern <- function(pattern, value_fun) {
  matches <- regexec(pattern, xxtpt, ignore.case = TRUE, perl = TRUE)
  parts <- regmatches(xxtpt, matches)
  matched <- lengths(parts) > 0 & is.na(result) & !na_idx
  if (any(matched)) {
    result[matched] <<- value_fun(parts[matched], matched)
  }
}

zero_pattern <- paste0(
  "^\\s*(?:",
  "start\\s+of\\s+(?:treatment|infusion)|",
  "start\\s+(?:treatment|infusion)|",
  "pre\\s*-?\\s*dose|",
  "predose|",
  "0\\s*(?:h|hr|hour|m|min|minute)?s?\\s*(?:post\\s*)?(?:dose|start)?",
  ")\\s*$"
)
result[grepl(zero_pattern, xxtpt, ignore.case = TRUE, perl = TRUE) &
  is.na(result) & !na_idx] <- 0

end_pattern <- "^\\s*(?:end\\s+of\\s+(?:treatment|infusion)|eot|eoi)\\s*$"
result[grepl(end_pattern, xxtpt, ignore.case = TRUE, perl = TRUE) &
  is.na(result) & !na_idx] <- treatment_duration[
    grepl(end_pattern, xxtpt, ignore.case = TRUE, perl = TRUE) &
      is.na(result) & !na_idx
  ]

time_unit <- "(\\d+(?:\\.\\d+)?)\\s*(m(?:in|inute)?|h(?:r|our)?)s?"

fill_pattern(
  paste0("^\\s*", time_unit, "\\s+(?:after|post)\\s+(?:start|start\\s+of\\s+(?:treatment|infusion)|start\\s+inf)\\s*$"),
  function(parts, matched) {
    vapply(parts, function(x) parse_hours(x[2], x[3]), numeric(1))
  }
)

fill_pattern(
  paste0("^\\s*", time_unit, "\\s+(?:before|pre|prior\\s+to)\\s+(?:end|end\\s+of\\s+(?:treatment|infusion)|eot|eoi)\\s*$"),
  function(parts, matched) {
    treatment_duration[matched] - vapply(parts, function(x) parse_hours(x[2], x[3]), numeric(1))
  }
)

fill_pattern(
  paste0("^\\s*", time_unit, "\\s+(?:after|post)\\s+(?:end|end\\s+of\\s+(?:treatment|infusion)|eot|eoi)\\s*$"),
  function(parts, matched) {
    treatment_duration[matched] + vapply(parts, function(x) parse_hours(x[2], x[3]), numeric(1))
  }
)

if (any(is.na(result) & !na_idx)) {
  stop(
    "Unconverted treatment timepoint pattern(s): ",
    paste(unique(xxtpt[is.na(result) & !na_idx]), collapse = "; "),
    call. = FALSE
  )
}

output <- data.frame(
  xxtpt = xxtpt,
  treatment_duration = treatment_duration,
  result = result,
  check.names = FALSE
)

dir.create("outputs", showWarnings = FALSE, recursive = TRUE)
utils::write.csv(output, file.path("outputs", "result.csv"), row.names = FALSE, na = "")
```
