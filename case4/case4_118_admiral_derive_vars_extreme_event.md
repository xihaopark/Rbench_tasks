# Case 118: pharmaverse/admiral/derive_vars_extreme_event

## Metadata
- Task ID: `pharmaverse/admiral/derive_vars_extreme_event`
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

Write an R script for an R package task: perform derive vars extreme event using the admiral package. Use functions from the admiral package when suitable.

Input: dataset.tsv, mode.tsv, source_datasets.tsv
Output: result.csv

Computation: Use source adverse events with AETOXGR >= 2, order by descending AETOXGR then ADT and AESEQ, and return one selected extreme event per STUDYID/USUBJID according to mode.


Required columns for result.csv: STUDYID, USUBJID, TRTSDT, EXTDT, EXTTERM, EXTGR
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
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
#!/usr/bin/env Rscript

read_tsv_chars <- function(path) {
  read.delim(
    path,
    sep = "\t",
    header = TRUE,
    stringsAsFactors = FALSE,
    colClasses = "character",
    check.names = FALSE
  )
}

dataset <- read_tsv_chars(file.path("inputs", "dataset.tsv"))
mode_data <- read_tsv_chars(file.path("inputs", "mode.tsv"))
source <- read_tsv_chars(file.path("inputs", "source_datasets.tsv"))

mode <- mode_data$mode[[1]]
if (!mode %in% c("first", "last")) {
  stop("Unsupported mode: ", mode)
}

source$ADT_sort <- as.Date(source$ADT)
source$AESEQ_sort <- suppressWarnings(as.numeric(source$AESEQ))
source$AETOXGR_sort <- suppressWarnings(as.numeric(source$AETOXGR))

eligible <- source[!is.na(source$AETOXGR_sort) & source$AETOXGR_sort >= 2, , drop = FALSE]

if (nrow(eligible) > 0) {
  eligible <- eligible[order(
    eligible$STUDYID,
    eligible$USUBJID,
    -eligible$AETOXGR_sort,
    eligible$ADT_sort,
    eligible$AESEQ_sort,
    na.last = TRUE
  ), , drop = FALSE]

  selected_index <- unlist(
    tapply(
      seq_len(nrow(eligible)),
      interaction(eligible$STUDYID, eligible$USUBJID, drop = TRUE, lex.order = TRUE),
      function(index) {
        if (mode == "first") {
          index[[1]]
        } else {
          index[[length(index)]]
        }
      }
    ),
    use.names = FALSE
  )
  selected <- eligible[selected_index, c("STUDYID", "USUBJID", "ADT", "AETERM", "AETOXGR"), drop = FALSE]
  names(selected) <- c("STUDYID", "USUBJID", "EXTDT", "EXTTERM", "EXTGR")
} else {
  selected <- data.frame(
    STUDYID = character(),
    USUBJID = character(),
    EXTDT = character(),
    EXTTERM = character(),
    EXTGR = character(),
    stringsAsFactors = FALSE
  )
}

result <- merge(
  dataset,
  selected,
  by = c("STUDYID", "USUBJID"),
  all.x = TRUE,
  sort = FALSE
)

result <- result[, c("STUDYID", "USUBJID", "TRTSDT", "EXTDT", "EXTTERM", "EXTGR"), drop = FALSE]

dir.create("outputs", showWarnings = FALSE, recursive = TRUE)
write.csv(result, file.path("outputs", "result.csv"), row.names = FALSE, na = "")
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
"S1","01","2020-01-01","2020-01-08","NAUSEA","3"
"S1","02","2020-01-05","2020-01-07","RASH","2"
"S1","03","2020-01-10","2020-01-15","COUGH","2"
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

read_tsv_chars <- function(path) {
  read.delim(
    path,
    sep = "\t",
    header = TRUE,
    stringsAsFactors = FALSE,
    colClasses = "character",
    check.names = FALSE
  )
}

dataset <- read_tsv_chars(file.path("inputs", "dataset.tsv"))
mode_data <- read_tsv_chars(file.path("inputs", "mode.tsv"))
source <- read_tsv_chars(file.path("inputs", "source_datasets.tsv"))

mode <- mode_data$mode[[1]]
if (!mode %in% c("first", "last")) {
  stop("Unsupported mode: ", mode)
}

source$ADT_sort <- as.Date(source$ADT)
source$AESEQ_sort <- suppressWarnings(as.numeric(source$AESEQ))
source$AETOXGR_sort <- suppressWarnings(as.numeric(source$AETOXGR))

eligible <- source[!is.na(source$AETOXGR_sort) & source$AETOXGR_sort >= 2, , drop = FALSE]

if (nrow(eligible) > 0) {
  eligible <- eligible[order(
    eligible$STUDYID,
    eligible$USUBJID,
    -eligible$AETOXGR_sort,
    eligible$ADT_sort,
    eligible$AESEQ_sort,
    na.last = TRUE
  ), , drop = FALSE]

  selected_index <- unlist(
    tapply(
      seq_len(nrow(eligible)),
      interaction(eligible$STUDYID, eligible$USUBJID, drop = TRUE, lex.order = TRUE),
      function(index) {
        if (mode == "first") {
          index[[1]]
        } else {
          index[[length(index)]]
        }
      }
    ),
    use.names = FALSE
  )
  selected <- eligible[selected_index, c("STUDYID", "USUBJID", "ADT", "AETERM", "AETOXGR"), drop = FALSE]
  names(selected) <- c("STUDYID", "USUBJID", "EXTDT", "EXTTERM", "EXTGR")
} else {
  selected <- data.frame(
    STUDYID = character(),
    USUBJID = character(),
    EXTDT = character(),
    EXTTERM = character(),
    EXTGR = character(),
    stringsAsFactors = FALSE
  )
}

result <- merge(
  dataset,
  selected,
  by = c("STUDYID", "USUBJID"),
  all.x = TRUE,
  sort = FALSE
)

result <- result[, c("STUDYID", "USUBJID", "TRTSDT", "EXTDT", "EXTTERM", "EXTGR"), drop = FALSE]

dir.create("outputs", showWarnings = FALSE, recursive = TRUE)
write.csv(result, file.path("outputs", "result.csv"), row.names = FALSE, na = "")
```
