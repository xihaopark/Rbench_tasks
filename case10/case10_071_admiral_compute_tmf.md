# Case 071: pharmaverse/admiral/compute_tmf

## Metadata
- Task ID: `pharmaverse/admiral/compute_tmf`
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

Write an R script for an R package task: perform compute tmf using the admiral package. Use functions from the admiral package when suitable.

Input: dtc.tsv, dtm.tsv, ignore_seconds_flag.tsv
Output: result.csv

Computation: For this fixture's partial-time flags, map hour-only imputation to M, minute-only imputation to S, and date-only records to H.


Required columns for result.csv: dtc, dtm, result
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.

## Input preview

### dtc.tsv
dtc
2020-01-01T12
2020-01-01T12:30
2020-01-01

### dtm.tsv
dtm
2020-01-01 12:00:00
2020-01-01 12:30:00
2020-01-01 00:00:00

### ignore_seconds_flag.tsv
ignore_seconds_flag
FALSE

Reference package function list:
The hidden reference solution's R package function calls are listed below. If the list is empty, the reference solution does not call package functions.
- package_functions: []
```

## Input
#### `dtc.tsv`

```text
dtc
2020-01-01T12
2020-01-01T12:30
2020-01-01
```

#### `dtm.tsv`

```text
dtm
2020-01-01 12:00:00
2020-01-01 12:30:00
2020-01-01 00:00:00
```

#### `ignore_seconds_flag.tsv`

```text
ignore_seconds_flag
FALSE
```

## Code
### Ground Truth Code

```r
suppressPackageStartupMessages(library(admiral))

dtc <- as.character(read.delim(file.path("inputs", "dtc.tsv"), check.names = FALSE, stringsAsFactors = FALSE)[[1]])
dtm_chr <- as.character(read.delim(file.path("inputs", "dtm.tsv"), check.names = FALSE, stringsAsFactors = FALSE)[[1]])
ignore_seconds_flag <- as.logical(read.delim(file.path("inputs", "ignore_seconds_flag.tsv"), check.names = FALSE, stringsAsFactors = FALSE)[[1]][1])

result <- ifelse(grepl("^\\d{4}-\\d{2}-\\d{2}T\\d{2}:\\d{2}:\\d{2}$", dtc), NA_character_,
          ifelse(grepl("^\\d{4}-\\d{2}-\\d{2}T\\d{2}:\\d{2}$", dtc), if (ignore_seconds_flag) NA_character_ else "S",
          ifelse(grepl("^\\d{4}-\\d{2}-\\d{2}T\\d{2}$", dtc), "M",
          ifelse(grepl("^\\d{4}-\\d{2}-\\d{2}$", dtc), "H", NA_character_))))

dir.create("outputs", showWarnings = FALSE)
write.csv(data.frame(dtc = dtc, dtm = dtm_chr, result = result), file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
library(dplyr)
library(readr)

dtc <- read_tsv("inputs/dtc.tsv", col_types = cols(dtc = col_character()))
dtm <- read_tsv("inputs/dtm.tsv", col_types = cols(dtm = col_character()))
ignore_seconds_flag <- read_tsv("inputs/ignore_seconds_flag.tsv", col_types = cols(ignore_seconds_flag = col_logical()))

ignore_seconds <- ignore_seconds_flag$ignore_seconds_flag[1]

# Compute TMF based on DTC format:
# - date-only (YYYY-MM-DD): H (hours imputed)
# - hour-only (YYYY-MM-DDTHH): M (minutes imputed)
# - minute-only (YYYY-MM-DDTHH:MM): S (seconds imputed)
# - full datetime (YYYY-MM-DDTHH:MM:SS): NA
compute_tmf <- function(dtc_val, ignore_sec = FALSE) {
  if (is.na(dtc_val)) return(NA_character_)

  has_time <- grepl("T", dtc_val)
  if (!has_time) {
    return("H")
  }

  time_part <- sub(".*T", "", dtc_val)
  n_colons <- nchar(time_part) - nchar(gsub(":", "", time_part))

  if (n_colons == 0) {
    return("M")
  } else if (n_colons == 1) {
    if (ignore_sec) return(NA_character_)
    return("S")
  } else {
    return(NA_character_)
  }
}

result <- bind_cols(dtc, dtm) %>%
  mutate(result = sapply(dtc, compute_tmf, ignore_sec = ignore_seconds))

dir.create("outputs", showWarnings = FALSE)
write_csv(result, "outputs/result.csv")
```

## Output
### Ground Truth Output
#### `result.csv`

```csv
"dtc","dtm","result"
"2020-01-01T12","2020-01-01 12:00:00","M"
"2020-01-01T12:30","2020-01-01 12:30:00","S"
"2020-01-01","2020-01-01 00:00:00","H"
```

### LLM Output
#### `result.csv`

```csv
dtc,dtm,result
2020-01-01T12,2020-01-01 12:00:00,M
2020-01-01T12:30,2020-01-01 12:30:00,S
2020-01-01,2020-01-01 00:00:00,H
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

Attaching package: ‘dplyr’

The following objects are masked from ‘package:stats’:

    filter, lag

The following objects are masked from ‘package:base’:

    intersect, setdiff, setequal, union
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
library(dplyr)
library(readr)

dtc <- read_tsv("inputs/dtc.tsv", col_types = cols(dtc = col_character()))
dtm <- read_tsv("inputs/dtm.tsv", col_types = cols(dtm = col_character()))
ignore_seconds_flag <- read_tsv("inputs/ignore_seconds_flag.tsv", col_types = cols(ignore_seconds_flag = col_logical()))

ignore_seconds <- ignore_seconds_flag$ignore_seconds_flag[1]

# Compute TMF based on DTC format:
# - date-only (YYYY-MM-DD): H (hours imputed)
# - hour-only (YYYY-MM-DDTHH): M (minutes imputed)
# - minute-only (YYYY-MM-DDTHH:MM): S (seconds imputed)
# - full datetime (YYYY-MM-DDTHH:MM:SS): NA
compute_tmf <- function(dtc_val, ignore_sec = FALSE) {
  if (is.na(dtc_val)) return(NA_character_)

  has_time <- grepl("T", dtc_val)
  if (!has_time) {
    return("H")
  }

  time_part <- sub(".*T", "", dtc_val)
  n_colons <- nchar(time_part) - nchar(gsub(":", "", time_part))

  if (n_colons == 0) {
    return("M")
  } else if (n_colons == 1) {
    if (ignore_sec) return(NA_character_)
    return("S")
  } else {
    return(NA_character_)
  }
}

result <- bind_cols(dtc, dtm) %>%
  mutate(result = sapply(dtc, compute_tmf, ignore_sec = ignore_seconds))

dir.create("outputs", showWarnings = FALSE)
write_csv(result, "outputs/result.csv")
```
