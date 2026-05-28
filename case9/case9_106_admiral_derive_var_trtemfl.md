# Case 106: pharmaverse/admiral/derive_var_trtemfl

## Metadata
- Task ID: `pharmaverse/admiral/derive_var_trtemfl`
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

Write an R script for an R package task: perform derive var trtemfl using the admiral package. Use functions from the admiral package when suitable.

Input: dataset.tsv
Output: result.csv

Computation: Derive TRTEMFL using the admiral treatment-emergent rule: flag in-window events and ongoing or worsening toxicity grades relative to baseline intensity.


Required columns for result.csv: USUBJID, TRTSDTM, TRTEDTM, ASTDTM, AENDTM, AEITOXGR, AETOXGR, TRTEMFL
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.

## Input preview

### dataset.tsv
USUBJID	TRTSDTM	TRTEDTM	ASTDTM	AENDTM	AEITOXGR	AETOXGR
01	2021-01-01	2021-01-31	2020-12-20	2020-12-21	2	2
01	2021-01-01	2021-01-31	2021-01-15	2021-01-20	1	3
02	2021-02-01	2021-02-28	2021-03-01	2021-03-05	1	2
03	2021-03-01	2021-03-31	2021-02-20	2021-03-02	2	3
```

## Input
#### `dataset.tsv`

```text
USUBJID	TRTSDTM	TRTEDTM	ASTDTM	AENDTM	AEITOXGR	AETOXGR
01	2021-01-01	2021-01-31	2020-12-20	2020-12-21	2	2
01	2021-01-01	2021-01-31	2021-01-15	2021-01-20	1	3
02	2021-02-01	2021-02-28	2021-03-01	2021-03-05	1	2
03	2021-03-01	2021-03-31	2021-02-20	2021-03-02	2	3
```

## Code
### Ground Truth Code

```r
suppressPackageStartupMessages(library(admiral))
suppressPackageStartupMessages(library(dplyr))

dataset <- read.delim(file.path("inputs", "dataset.tsv"), check.names = FALSE, stringsAsFactors = FALSE)
for (col in c("TRTSDTM", "TRTEDTM", "ASTDTM", "AENDTM")) {
  dataset[[col]] <- as.POSIXct(dataset[[col]], tz = "UTC")
}

result <- admiral::derive_var_trtemfl(
  dataset,
  new_var = TRTEMFL,
  trt_end_date = TRTEDTM,
  end_window = 0,
  initial_intensity = AEITOXGR,
  intensity = AETOXGR,
  subject_keys = exprs(USUBJID)
)

dir.create("outputs", showWarnings = FALSE)
write.csv(result, file.path("outputs", "result.csv"), row.names = FALSE, na = "")
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

library(admiral)
library(dplyr)
library(lubridate)

# Create output directory if needed
dir.create("outputs", showWarnings = FALSE, recursive = TRUE)

# Read input data
dataset <- read.table("inputs/dataset.tsv", sep = "\t", header = TRUE,
                      colClasses = "character", stringsAsFactors = FALSE)

# Convert date/datetime columns to POSIXct (admiral requires date/datetime types)
dataset <- dataset %>%
  mutate(
    TRTSDTM  = as.POSIXct(TRTSDTM,  tz = "UTC"),
    TRTEDTM  = as.POSIXct(TRTEDTM,  tz = "UTC"),
    ASTDTM   = as.POSIXct(ASTDTM,   tz = "UTC"),
    AENDTM   = as.POSIXct(AENDTM,   tz = "UTC"),
    AEITOXGR = as.character(AEITOXGR),
    AETOXGR  = as.character(AETOXGR)
  )

# Derive TRTEMFL using admiral::derive_var_trtemfl
# Treatment-emergent rule flags:
#   1. In-window events: ASTDTM >= TRTSDTM and <= TRTEDTM (end_window = 0)
#   2. Pre-existing ongoing/worsening: ASTDTM < TRTSDTM, AENDTM >= TRTSDTM,
#      and AETOXGR > AEITOXGR
result <- tryCatch({
  derive_var_trtemfl(
    dataset,
    new_var           = TRTEMFL,
    trt_start_date    = TRTSDTM,
    trt_end_date      = TRTEDTM,
    end_window        = 0,
    initial_intensity = AEITOXGR,
    intensity         = AETOXGR,
    start_date        = ASTDTM,
    end_date          = AENDTM
  )
}, error = function(e) {
  message("admiral::derive_var_trtemfl failed: ", conditionMessage(e))
  message("Falling back to manual implementation.")
  NULL
})

# Manual fallback implementing the same treatment-emergent rule
if (is.null(result)) {
  result <- dataset %>%
    mutate(
      TRTEMFL = case_when(
        # AE ends before treatment start — not treatment-emergent
        AENDTM < TRTSDTM ~ NA_character_,
        # Event starts on or after treatment start and on or before treatment end
        ASTDTM >= TRTSDTM & (is.na(TRTEDTM) | as.Date(ASTDTM) <= as.Date(TRTEDTM)) ~ "Y",
        # Pre-existing event ongoing at treatment start that worsened
        ASTDTM < TRTSDTM & !is.na(AENDTM) & AENDTM >= TRTSDTM &
          AETOXGR > AEITOXGR ~ "Y",
        TRUE ~ NA_character_
      )
    )
}

# Keep only required columns
required_cols <- c("USUBJID", "TRTSDTM", "TRTEDTM", "ASTDTM", "AENDTM",
                   "AEITOXGR", "AETOXGR", "TRTEMFL")
result <- result %>% select(all_of(required_cols))

# Write output
write.csv(result, "outputs/result.csv", row.names = FALSE)
message("Done. Output written to outputs/result.csv")
```

## Output
### Ground Truth Output
#### `result.csv`

```csv
"USUBJID","TRTSDTM","TRTEDTM","ASTDTM","AENDTM","AEITOXGR","AETOXGR","TRTEMFL"
1,2021-01-01,2021-01-31,2020-12-20,2020-12-21,2,2,
1,2021-01-01,2021-01-31,2021-01-15,2021-01-20,1,3,"Y"
2,2021-02-01,2021-02-28,2021-03-01,2021-03-05,1,2,
3,2021-03-01,2021-03-31,2021-02-20,2021-03-02,2,3,"Y"
```

### LLM Output
#### `result.csv`

```csv
"USUBJID","TRTSDTM","TRTEDTM","ASTDTM","AENDTM","AEITOXGR","AETOXGR","TRTEMFL"
"01",2021-01-01,2021-01-31,2020-12-20,2020-12-21,"2","2",NA
"01",2021-01-01,2021-01-31,2021-01-15,2021-01-20,"1","3","Y"
"02",2021-02-01,2021-02-28,2021-03-01,2021-03-05,"1","2",NA
"03",2021-03-01,2021-03-31,2021-02-20,2021-03-02,"2","3","Y"
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


Attaching package: ‘lubridate’

The following objects are masked from ‘package:base’:

    date, intersect, setdiff, union

Done. Output written to outputs/result.csv
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

library(admiral)
library(dplyr)
library(lubridate)

# Create output directory if needed
dir.create("outputs", showWarnings = FALSE, recursive = TRUE)

# Read input data
dataset <- read.table("inputs/dataset.tsv", sep = "\t", header = TRUE,
                      colClasses = "character", stringsAsFactors = FALSE)

# Convert date/datetime columns to POSIXct (admiral requires date/datetime types)
dataset <- dataset %>%
  mutate(
    TRTSDTM  = as.POSIXct(TRTSDTM,  tz = "UTC"),
    TRTEDTM  = as.POSIXct(TRTEDTM,  tz = "UTC"),
    ASTDTM   = as.POSIXct(ASTDTM,   tz = "UTC"),
    AENDTM   = as.POSIXct(AENDTM,   tz = "UTC"),
    AEITOXGR = as.character(AEITOXGR),
    AETOXGR  = as.character(AETOXGR)
  )

# Derive TRTEMFL using admiral::derive_var_trtemfl
# Treatment-emergent rule flags:
#   1. In-window events: ASTDTM >= TRTSDTM and <= TRTEDTM (end_window = 0)
#   2. Pre-existing ongoing/worsening: ASTDTM < TRTSDTM, AENDTM >= TRTSDTM,
#      and AETOXGR > AEITOXGR
result <- tryCatch({
  derive_var_trtemfl(
    dataset,
    new_var           = TRTEMFL,
    trt_start_date    = TRTSDTM,
    trt_end_date      = TRTEDTM,
    end_window        = 0,
    initial_intensity = AEITOXGR,
    intensity         = AETOXGR,
    start_date        = ASTDTM,
    end_date          = AENDTM
  )
}, error = function(e) {
  message("admiral::derive_var_trtemfl failed: ", conditionMessage(e))
  message("Falling back to manual implementation.")
  NULL
})

# Manual fallback implementing the same treatment-emergent rule
if (is.null(result)) {
  result <- dataset %>%
    mutate(
      TRTEMFL = case_when(
        # AE ends before treatment start — not treatment-emergent
        AENDTM < TRTSDTM ~ NA_character_,
        # Event starts on or after treatment start and on or before treatment end
        ASTDTM >= TRTSDTM & (is.na(TRTEDTM) | as.Date(ASTDTM) <= as.Date(TRTEDTM)) ~ "Y",
        # Pre-existing event ongoing at treatment start that worsened
        ASTDTM < TRTSDTM & !is.na(AENDTM) & AENDTM >= TRTSDTM &
          AETOXGR > AEITOXGR ~ "Y",
        TRUE ~ NA_character_
      )
    )
}

# Keep only required columns
required_cols <- c("USUBJID", "TRTSDTM", "TRTEDTM", "ASTDTM", "AENDTM",
                   "AEITOXGR", "AETOXGR", "TRTEMFL")
result <- result %>% select(all_of(required_cols))

# Write output
write.csv(result, "outputs/result.csv", row.names = FALSE)
message("Done. Output written to outputs/result.csv")
```
