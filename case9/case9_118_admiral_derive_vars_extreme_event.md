# Case 118: pharmaverse/admiral/derive_vars_extreme_event

## Metadata
- Task ID: `pharmaverse/admiral/derive_vars_extreme_event`
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

Write an R script for an R package task: perform derive vars extreme event using the admiral package. Use functions from the admiral package when suitable.

Input: dataset.tsv, mode.tsv, source_datasets.tsv
Output: result.csv

Computation: Use source adverse events with AETOXGR >= 2, order by descending AETOXGR then ADT and AESEQ, and return one selected extreme event per STUDYID/USUBJID according to mode.


Required columns for result.csv: STUDYID, USUBJID, TRTSDT, EXTDT, EXTTERM, EXTGR
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.

## Input preview

### dataset.tsv
STUDYID	USUBJID	TRTSDT
S1	01	2020-01-01
S1	02	2020-01-05
S1	03	2020-01-10

### mode.tsv
mode
first

### source_datasets.tsv
STUDYID	USUBJID	ADT	AESEQ	AETERM	AETOXGR
S1	01	2020-01-03	1	HEADACHE	1
S1	01	2020-01-08	2	NAUSEA	3
S1	02	2020-01-07	1	RASH	2
S1	02	2020-01-08	2	FEVER	1
S1	03	2020-01-15	1	COUGH	2
```

## Input
#### `dataset.tsv`

```text
STUDYID	USUBJID	TRTSDT
S1	01	2020-01-01
S1	02	2020-01-05
S1	03	2020-01-10
```

#### `mode.tsv`

```text
mode
first
```

#### `source_datasets.tsv`

```text
STUDYID	USUBJID	ADT	AESEQ	AETERM	AETOXGR
S1	01	2020-01-03	1	HEADACHE	1
S1	01	2020-01-08	2	NAUSEA	3
S1	02	2020-01-07	1	RASH	2
S1	02	2020-01-08	2	FEVER	1
S1	03	2020-01-15	1	COUGH	2
```

## Code
### Ground Truth Code

```r
suppressPackageStartupMessages(library(admiral))
suppressPackageStartupMessages(library(dplyr))
suppressPackageStartupMessages(library(rlang))

dataset <- read.delim(file.path("inputs", "dataset.tsv"), check.names = FALSE, stringsAsFactors = FALSE)
ae <- read.delim(file.path("inputs", "source_datasets.tsv"), check.names = FALSE, stringsAsFactors = FALSE)
mode <- read.delim(file.path("inputs", "mode.tsv"), check.names = FALSE, stringsAsFactors = FALSE)$mode[1]

dataset$TRTSDT <- as.Date(dataset$TRTSDT)
ae$ADT <- as.Date(ae$ADT)

grade2_event <- admiral::event(
  dataset_name = "ae",
  condition = AETOXGR >= 2,
  set_values_to = rlang::exprs(EVNTDESC = "Grade 2 or higher adverse event")
)

result <- admiral::derive_vars_extreme_event(
  dataset = dataset,
  by_vars = rlang::exprs(STUDYID, USUBJID),
  events = list(grade2_event),
  source_datasets = list(ae = ae),
  order = rlang::exprs(dplyr::desc(AETOXGR), ADT, AESEQ),
  mode = mode,
  new_vars = rlang::exprs(EXTDT = ADT, EXTTERM = AETERM, EXTGR = AETOXGR)
)

dir.create("outputs", showWarnings = FALSE)
write.csv(result, file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
library(admiral)
library(dplyr)

# Read inputs
dataset <- read.delim("inputs/dataset.tsv",          stringsAsFactors = FALSE)
mode_df <- read.delim("inputs/mode.tsv",             stringsAsFactors = FALSE)
ae_data <- read.delim("inputs/source_datasets.tsv",  stringsAsFactors = FALSE)

# Extract mode string ("first" or "last")
mode_val <- mode_df$mode[1]

# Convert date columns to Date class
dataset <- dataset %>%
  mutate(TRTSDT = as.Date(TRTSDT))

ae_data <- ae_data %>%
  mutate(ADT = as.Date(ADT))

# Derive extreme-event variables using admiral::derive_vars_extreme_event
#
# Logic:
#   - Source: ae_data, filtered to AETOXGR >= 2
#   - Order within each STUDYID/USUBJID: desc(AETOXGR), then ADT, then AESEQ
#   - Select the first/last record per group after ordering (from mode_val)
#   - Map source variables -> new output variables:
#       ADT     -> EXTDT
#       AETERM  -> EXTTERM
#       AETOXGR -> EXTGR

result <- derive_vars_extreme_event(
  dataset,
  by_vars = exprs(STUDYID, USUBJID),
  events = list(
    event(
      dataset_name = "ae",
      condition    = AETOXGR >= 2,
      set_values_to = exprs(
        EXTDT   = ADT,
        EXTTERM = AETERM,
        EXTGR   = AETOXGR
      )
    )
  ),
  order           = exprs(desc(AETOXGR), ADT, AESEQ),
  mode            = mode_val,
  source_datasets = list(ae = ae_data),
  new_vars        = exprs(EXTDT, EXTTERM, EXTGR)
)

# Keep only required output columns
result_out <- result %>%
  select(STUDYID, USUBJID, TRTSDT, EXTDT, EXTTERM, EXTGR)

# Write output
dir.create("outputs", showWarnings = FALSE, recursive = TRUE)
write.csv(result_out, "outputs/result.csv", row.names = FALSE)
```

## Output
### Ground Truth Output
#### `result.csv`

```csv
"STUDYID","USUBJID","TRTSDT","EXTDT","EXTTERM","EXTGR"
"S1",1,2020-01-01,2020-01-08,"NAUSEA",3
"S1",2,2020-01-05,2020-01-07,"RASH",2
"S1",3,2020-01-10,2020-01-15,"COUGH",2
```

### LLM Output
#### `result.csv`

```csv
"STUDYID","USUBJID","TRTSDT","EXTDT","EXTTERM","EXTGR"
"S1",1,2020-01-01,2020-01-08,"NAUSEA",3
"S1",2,2020-01-05,2020-01-07,"RASH",2
"S1",3,2020-01-10,2020-01-15,"COUGH",2
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
library(admiral)
library(dplyr)

# Read inputs
dataset <- read.delim("inputs/dataset.tsv",          stringsAsFactors = FALSE)
mode_df <- read.delim("inputs/mode.tsv",             stringsAsFactors = FALSE)
ae_data <- read.delim("inputs/source_datasets.tsv",  stringsAsFactors = FALSE)

# Extract mode string ("first" or "last")
mode_val <- mode_df$mode[1]

# Convert date columns to Date class
dataset <- dataset %>%
  mutate(TRTSDT = as.Date(TRTSDT))

ae_data <- ae_data %>%
  mutate(ADT = as.Date(ADT))

# Derive extreme-event variables using admiral::derive_vars_extreme_event
#
# Logic:
#   - Source: ae_data, filtered to AETOXGR >= 2
#   - Order within each STUDYID/USUBJID: desc(AETOXGR), then ADT, then AESEQ
#   - Select the first/last record per group after ordering (from mode_val)
#   - Map source variables -> new output variables:
#       ADT     -> EXTDT
#       AETERM  -> EXTTERM
#       AETOXGR -> EXTGR

result <- derive_vars_extreme_event(
  dataset,
  by_vars = exprs(STUDYID, USUBJID),
  events = list(
    event(
      dataset_name = "ae",
      condition    = AETOXGR >= 2,
      set_values_to = exprs(
        EXTDT   = ADT,
        EXTTERM = AETERM,
        EXTGR   = AETOXGR
      )
    )
  ),
  order           = exprs(desc(AETOXGR), ADT, AESEQ),
  mode            = mode_val,
  source_datasets = list(ae = ae_data),
  new_vars        = exprs(EXTDT, EXTTERM, EXTGR)
)

# Keep only required output columns
result_out <- result %>%
  select(STUDYID, USUBJID, TRTSDT, EXTDT, EXTTERM, EXTGR)

# Write output
dir.create("outputs", showWarnings = FALSE, recursive = TRUE)
write.csv(result_out, "outputs/result.csv", row.names = FALSE)
```
