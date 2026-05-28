# Case 016: pharmaverse/aNCA/detect_study_types

## Metadata
- Task ID: `pharmaverse/aNCA/detect_study_types`
- Package: `aNCA`
- Model: `claude-code/claude-sonnet-4-6`
- Agent: `Claude Code`
- Status: `FAIL`
- Failure stage: `schema_mismatch`
- Pattern: ``

## Pass/Error
```text
FAIL
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

Write an R script for an R package task: perform detect study types using the aNCA package. Use functions from the aNCA package when suitable.

Input: data.tsv, groups.tsv, metabfl_column.tsv, route_column.tsv, volume_column.tsv
Output: result.csv

Computation: Classify rows by route/evidence: oral rows as oral, IV infusion rows as iv_infusion, and excretion rows as excretion; preserve one output row per input row.


Required columns for result.csv: subject, study_type
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.

## Input preview

### data.tsv
subject	period	route	urine_volume	metabfl
101	1	oral	0	parent
101	2	oral	0	parent
102	1	iv infusion	0	parent
103	1	urine	150	metabolite

### groups.tsv
groups
subject

### metabfl_column.tsv
metabfl_column
metabfl

### route_column.tsv
route_column
route

### volume_column.tsv
volume_column
urine_volume
```

## Input
#### `data.tsv`

```text
subject	period	route	urine_volume	metabfl
101	1	oral	0	parent
101	2	oral	0	parent
102	1	iv infusion	0	parent
103	1	urine	150	metabolite
```

#### `groups.tsv`

```text
groups
subject
```

#### `metabfl_column.tsv`

```text
metabfl_column
metabfl
```

#### `route_column.tsv`

```text
route_column
route
```

#### `volume_column.tsv`

```text
volume_column
urine_volume
```

## Code
### Ground Truth Code

```r
read_table <- function(name) {
  read.delim(file.path("inputs", name), check.names = FALSE, stringsAsFactors = FALSE)
}

first_value <- function(df, default = "") {
  if (nrow(df) == 0 || ncol(df) == 0 || is.na(df[[1]][[1]])) return(default)
  as.character(df[[1]][[1]])
}

is_extravascular <- function(route) {
  grepl("^(extravascular|oral|po|subcutaneous|sc|intramuscular|im)$", tolower(route))
}

classify_group <- function(df, route_col, volume_col) {
  if (volume_col %in% names(df)) {
    volume <- suppressWarnings(as.numeric(df[[volume_col]]))
    if (any(!is.na(volume) & volume > 0)) return("Excretion Data")
  }
  route <- if (route_col %in% names(df)) df[[route_col]][[1]] else NA_character_
  multiple <- ("TRTRINT" %in% names(df) && any(!is.na(df$TRTRINT))) || ("DOSNO" %in% names(df) && length(unique(df$DOSNO)) > 1)
  bolus <- "ADOSEDUR" %in% names(df) && suppressWarnings(as.numeric(df$ADOSEDUR[[1]])) == 0
  if (is.na(route)) return("Unknown")
  if (is_extravascular(route)) return(ifelse(multiple, "Multiple Extravascular Doses", "Single Extravascular Dose"))
  if (bolus) return(ifelse(multiple, "Multiple IV Bolus Doses", "Single IV Bolus Dose"))
  ifelse(multiple, "Multiple IV Infusion Doses", "Single IV Infusion Dose")
}

data <- read_table("data.tsv")
groups <- as.character(read_table("groups.tsv")[[1]])
groups <- groups[groups %in% names(data)]
route_col <- first_value(read_table("route_column.tsv"), "route")
volume_col <- first_value(read_table("volume_column.tsv"), "volume")

if (!(route_col %in% names(data)) && !(volume_col %in% names(data))) {
  result <- data.frame(study_type = "unknown", stringsAsFactors = FALSE)
} else if (length(groups) == 0) {
  result <- data.frame(study_type = classify_group(data, route_col, volume_col), stringsAsFactors = FALSE)
} else {
  keys <- unique(data[groups])
  study_type <- character(nrow(keys))
  for (i in seq_len(nrow(keys))) {
    keep <- rep(TRUE, nrow(data))
    for (col in groups) keep <- keep & data[[col]] == keys[[col]][[i]]
    study_type[[i]] <- classify_group(data[keep, , drop = FALSE], route_col, volume_col)
  }
  result <- cbind(keys, study_type = study_type)
}

dir.create("outputs", showWarnings = FALSE, recursive = TRUE)
write.csv(result, file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

dir.create('outputs', showWarnings = FALSE, recursive = TRUE)

library(dplyr)

data        <- read.delim("inputs/data.tsv",           stringsAsFactors = FALSE, check.names = FALSE)
route_col   <- read.delim("inputs/route_column.tsv",   stringsAsFactors = FALSE, check.names = FALSE)
volume_col  <- read.delim("inputs/volume_column.tsv",  stringsAsFactors = FALSE, check.names = FALSE)

route_column  <- route_col[[1]][1]
volume_column <- volume_col[[1]][1]

detect_study_type <- function(route, volume) {
  r <- tolower(trimws(route))
  if (grepl("^oral$", r)) {
    "oral"
  } else if (grepl("iv", r) && grepl("infusion", r)) {
    "iv_infusion"
  } else if (grepl("urine|excretion|feces|faeces|bile", r) || (!is.na(volume) && volume > 0)) {
    "excretion"
  } else {
    NA_character_
  }
}

data$study_type <- mapply(detect_study_type, data[[route_column]], data[[volume_column]])

result <- data[, c("subject", "study_type")]

write.csv(result, "outputs/result.csv", row.names = FALSE)
cat("Done. Rows:", nrow(result), "\n")
print(result)
```

## Output
### Ground Truth Output
#### `result.csv`

```csv
"subject","study_type"
101,"Single Extravascular Dose"
102,"Single IV Infusion Dose"
103,"Excretion Data"
```

### LLM Output
#### `result.csv`

```csv
"subject","study_type"
101,"oral"
101,"oral"
102,"iv_infusion"
103,"excretion"
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
  "status": "FAIL",
  "tier": "output_bad",
  "failure_stage": "schema_mismatch",
  "pass": false,
  "score": 0.0,
  "message": "Failed at case_embedded",
  "validation_diagnostics": {
    "failure_stage": "schema_mismatch",
    "tier": "output_bad",
    "case_count": 1,
    "failed_case": "case_embedded"
  }
}
```

## Raw Agent Response
```text
#!/usr/bin/env Rscript

dir.create('outputs', showWarnings = FALSE, recursive = TRUE)

library(dplyr)

data        <- read.delim("inputs/data.tsv",           stringsAsFactors = FALSE, check.names = FALSE)
route_col   <- read.delim("inputs/route_column.tsv",   stringsAsFactors = FALSE, check.names = FALSE)
volume_col  <- read.delim("inputs/volume_column.tsv",  stringsAsFactors = FALSE, check.names = FALSE)

route_column  <- route_col[[1]][1]
volume_column <- volume_col[[1]][1]

detect_study_type <- function(route, volume) {
  r <- tolower(trimws(route))
  if (grepl("^oral$", r)) {
    "oral"
  } else if (grepl("iv", r) && grepl("infusion", r)) {
    "iv_infusion"
  } else if (grepl("urine|excretion|feces|faeces|bile", r) || (!is.na(volume) && volume > 0)) {
    "excretion"
  } else {
    NA_character_
  }
}

data$study_type <- mapply(detect_study_type, data[[route_column]], data[[volume_column]])

result <- data[, c("subject", "study_type")]

write.csv(result, "outputs/result.csv", row.names = FALSE)
cat("Done. Rows:", nrow(result), "\n")
print(result)
```
