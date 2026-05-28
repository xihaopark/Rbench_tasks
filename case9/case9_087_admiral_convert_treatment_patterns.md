# Case 087: pharmaverse/admiral/convert_treatment_patterns

## Metadata
- Task ID: `pharmaverse/admiral/convert_treatment_patterns`
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
library(admiral)

# Read inputs
xxtpt_df <- read.delim("inputs/xxtpt.tsv", stringsAsFactors = FALSE)
treatment_duration_df <- read.delim("inputs/treatment_duration.tsv", stringsAsFactors = FALSE)

xxtpt <- xxtpt_df$xxtpt
treatment_duration <- treatment_duration_df$treatment_duration

# Use admiral's convert_xxtpt_to_hours for primary conversion
result <- convert_xxtpt_to_hours(xxtpt, treatment_duration = treatment_duration)

# Custom fallback for patterns not handled by convert_xxtpt_to_hours
# e.g. "START OF TREATMENT", "4 HOURS BEFORE END", "6 HOURS AFTER END"
parse_offset_hours <- function(tpt_upper) {
  m <- regmatches(
    tpt_upper,
    regexpr("([0-9]+(?:[.][0-9]+)?)\\s*(HOUR|HR|H|MIN|M|DAY|D)", tpt_upper, perl = TRUE)
  )
  if (length(m) == 0) return(NA_real_)
  num <- as.numeric(sub("([0-9]+(?:[.][0-9]+)?).*", "\\1", m, perl = TRUE))
  unit <- sub("[0-9]+(?:[.][0-9]+)?\\s*", "", m, perl = TRUE)
  if (grepl("^(MIN|M)$", unit)) num <- num / 60
  if (grepl("^(DAY|D)$", unit)) num <- num * 24
  num
}

for (i in seq_along(result)) {
  if (!is.na(result[i])) next

  tpt <- toupper(trimws(xxtpt[i]))
  dur <- treatment_duration[i]

  if (grepl("^(START OF TREATMENT|START|ZERO)$", tpt)) {
    result[i] <- 0

  } else if (grepl("AFTER START|POST START", tpt)) {
    # e.g., "30 MIN AFTER START", "2 HOURS AFTER START"
    offset <- parse_offset_hours(tpt)
    if (!is.na(offset)) result[i] <- offset

  } else if (grepl("BEFORE END|PRE[- ]?EOT|BEFORE EOT|PRE END", tpt)) {
    offset <- parse_offset_hours(tpt)
    if (!is.na(offset)) result[i] <- dur - offset

  } else if (grepl("AFTER END|POST END", tpt)) {
    offset <- parse_offset_hours(tpt)
    if (!is.na(offset)) result[i] <- dur + offset
  }
}

# Build output data frame
out <- data.frame(
  xxtpt = xxtpt,
  treatment_duration = treatment_duration,
  result = result,
  stringsAsFactors = FALSE
)

# Write output
dir.create("outputs", showWarnings = FALSE)
write.csv(out, "outputs/result.csv", row.names = FALSE)

cat("Done. Output rows:", nrow(out), "\n")
print(out)
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
Error in convert_special_cases(xxtpt, result, na_idx, treatment_duration) : 
  unused arguments (result, na_idx, treatment_duration)
Calls: convert_xxtpt_to_hours
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

# Read inputs
xxtpt_df <- read.delim("inputs/xxtpt.tsv", stringsAsFactors = FALSE)
treatment_duration_df <- read.delim("inputs/treatment_duration.tsv", stringsAsFactors = FALSE)

xxtpt <- xxtpt_df$xxtpt
treatment_duration <- treatment_duration_df$treatment_duration

# Use admiral's convert_xxtpt_to_hours for primary conversion
result <- convert_xxtpt_to_hours(xxtpt, treatment_duration = treatment_duration)

# Custom fallback for patterns not handled by convert_xxtpt_to_hours
# e.g. "START OF TREATMENT", "4 HOURS BEFORE END", "6 HOURS AFTER END"
parse_offset_hours <- function(tpt_upper) {
  m <- regmatches(
    tpt_upper,
    regexpr("([0-9]+(?:[.][0-9]+)?)\\s*(HOUR|HR|H|MIN|M|DAY|D)", tpt_upper, perl = TRUE)
  )
  if (length(m) == 0) return(NA_real_)
  num <- as.numeric(sub("([0-9]+(?:[.][0-9]+)?).*", "\\1", m, perl = TRUE))
  unit <- sub("[0-9]+(?:[.][0-9]+)?\\s*", "", m, perl = TRUE)
  if (grepl("^(MIN|M)$", unit)) num <- num / 60
  if (grepl("^(DAY|D)$", unit)) num <- num * 24
  num
}

for (i in seq_along(result)) {
  if (!is.na(result[i])) next

  tpt <- toupper(trimws(xxtpt[i]))
  dur <- treatment_duration[i]

  if (grepl("^(START OF TREATMENT|START|ZERO)$", tpt)) {
    result[i] <- 0

  } else if (grepl("AFTER START|POST START", tpt)) {
    # e.g., "30 MIN AFTER START", "2 HOURS AFTER START"
    offset <- parse_offset_hours(tpt)
    if (!is.na(offset)) result[i] <- offset

  } else if (grepl("BEFORE END|PRE[- ]?EOT|BEFORE EOT|PRE END", tpt)) {
    offset <- parse_offset_hours(tpt)
    if (!is.na(offset)) result[i] <- dur - offset

  } else if (grepl("AFTER END|POST END", tpt)) {
    offset <- parse_offset_hours(tpt)
    if (!is.na(offset)) result[i] <- dur + offset
  }
}

# Build output data frame
out <- data.frame(
  xxtpt = xxtpt,
  treatment_duration = treatment_duration,
  result = result,
  stringsAsFactors = FALSE
)

# Write output
dir.create("outputs", showWarnings = FALSE)
write.csv(out, "outputs/result.csv", row.names = FALSE)

cat("Done. Output rows:", nrow(out), "\n")
print(out)
```
