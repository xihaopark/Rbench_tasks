# Case 164: pharmaverse/admiral/slice_derivation

## Metadata
- Task ID: `pharmaverse/admiral/slice_derivation`
- Package: `admiral`
- Model: `codex/gpt-5.5`
- Agent: `Codex CLI`
- Agent command: `codex exec -C <worktree> --dangerously-bypass-approvals-and-sandbox <prompt> -m gpt-5.5`
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
- Read input files only from `inputs/` using relative paths.
- Write exactly the required output artifact(s): outputs/result.csv.
- Create `outputs/` if needed.
- You may inspect `task.json`, `TASK.md`, and input files.
- Do not modify `inputs/`, `task.json`, `AGENTS.md`, or hidden evaluator metadata.
- Do not use files outside this worktree.
- Do not commit changes.
- Before finishing, run `Rscript solution.R` if R is available and fix any concrete error.

Task prompt:

Write an R script for an R package task: perform slice derivation using the admiral package. Use functions from the admiral package when suitable.

Input: dataset.tsv, filter.tsv
Output: result.csv

Computation: Within each subject, use the filter condition to identify the first selected visit, then keep rows through that selected visit rather than only the matching row.


Required columns for result.csv: USUBJID, AVISITN, AVAL
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.

## Input preview

### dataset.tsv
USUBJID	AVISITN	AVAL
01	1	10
01	2	12
02	1	9
02	2	11

### filter.tsv
filter
AVISITN == 1
```

## Input
#### `dataset.tsv`

```text
USUBJID	AVISITN	AVAL
01	1	10
01	2	12
02	1	9
02	2	11
```

#### `filter.tsv`

```text
filter
AVISITN == 1
```

## Code
### Ground Truth Code

```r
suppressPackageStartupMessages(library(admiral))
suppressPackageStartupMessages(library(dplyr))
suppressPackageStartupMessages(library(rlang))

dataset <- read.delim(file.path("inputs", "dataset.tsv"), check.names = FALSE, stringsAsFactors = FALSE)
filter_expr <- read.delim(file.path("inputs", "filter.tsv"), check.names = FALSE, stringsAsFactors = FALSE)$filter[1]
slice <- admiral::derivation_slice(!!parse_expr(filter_expr))
result <- admiral::slice_derivation(dataset, derivation = dplyr::filter, slice)
dir.create("outputs", showWarnings = FALSE)
write.csv(result, file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

library(admiral)

dir.create("outputs", showWarnings = FALSE, recursive = TRUE)

dataset <- read.delim(
  file = file.path("inputs", "dataset.tsv"),
  sep = "\t",
  header = TRUE,
  stringsAsFactors = FALSE,
  colClasses = c(USUBJID = "character")
)

filter_data <- read.delim(
  file = file.path("inputs", "filter.tsv"),
  sep = "\t",
  header = TRUE,
  stringsAsFactors = FALSE,
  check.names = FALSE
)

required_columns <- c("USUBJID", "AVISITN", "AVAL")
missing_columns <- setdiff(required_columns, names(dataset))
if (length(missing_columns) > 0) {
  stop("dataset.tsv is missing required columns: ", paste(missing_columns, collapse = ", "))
}

if (!"filter" %in% names(filter_data) || nrow(filter_data) < 1 || is.na(filter_data$filter[1])) {
  stop("filter.tsv must contain a non-missing filter column.")
}

filter_text <- trimws(filter_data$filter[1])
if (!nzchar(filter_text)) {
  stop("The filter expression is empty.")
}
filter_expr <- parse(text = filter_text)[[1]]

keep_through_first_selected_visit <- function(dataset, filter_expr) {
  if (nrow(dataset) == 0) {
    return(dataset)
  }

  selected <- eval(filter_expr, envir = dataset, enclos = parent.frame())
  selected[is.na(selected)] <- FALSE
  if (!is.logical(selected) || length(selected) != nrow(dataset)) {
    stop("The filter condition must evaluate to one logical value per input row.")
  }

  dataset$..selected_filter <- selected
  dataset$..input_order <- seq_len(nrow(dataset))
  dataset <- dataset[order(dataset$USUBJID, dataset$AVISITN, dataset$..input_order), , drop = FALSE]

  pieces <- split(dataset, dataset$USUBJID, drop = TRUE)
  kept <- lapply(pieces, function(subject_data) {
    selected_rows <- which(subject_data$..selected_filter)
    if (length(selected_rows) == 0) {
      return(subject_data[0, , drop = FALSE])
    }

    first_selected_visit <- subject_data$AVISITN[selected_rows[1]]
    subject_data[subject_data$AVISITN <= first_selected_visit, , drop = FALSE]
  })

  result <- do.call(rbind, kept)
  result <- result[order(result$..input_order), , drop = FALSE]
  result$..selected_filter <- NULL
  result$..input_order <- NULL
  row.names(result) <- NULL
  result
}

result <- slice_derivation(
  dataset = dataset,
  derivation = keep_through_first_selected_visit,
  derivation_slice(filter = TRUE),
  args = params(filter_expr = filter_expr)
)

result <- result[, required_columns, drop = FALSE]

write.csv(result, file.path("outputs", "result.csv"), row.names = FALSE, na = "")
```

## Output
### Ground Truth Output
#### `result.csv`

```csv
"USUBJID","AVISITN","AVAL"
1,1,10
2,1,9
1,2,12
2,2,11
```

### LLM Output
#### `result.csv`

```csv
"USUBJID","AVISITN","AVAL"
"01",1,10
"02",1,9
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

library(admiral)

dir.create("outputs", showWarnings = FALSE, recursive = TRUE)

dataset <- read.delim(
  file = file.path("inputs", "dataset.tsv"),
  sep = "\t",
  header = TRUE,
  stringsAsFactors = FALSE,
  colClasses = c(USUBJID = "character")
)

filter_data <- read.delim(
  file = file.path("inputs", "filter.tsv"),
  sep = "\t",
  header = TRUE,
  stringsAsFactors = FALSE,
  check.names = FALSE
)

required_columns <- c("USUBJID", "AVISITN", "AVAL")
missing_columns <- setdiff(required_columns, names(dataset))
if (length(missing_columns) > 0) {
  stop("dataset.tsv is missing required columns: ", paste(missing_columns, collapse = ", "))
}

if (!"filter" %in% names(filter_data) || nrow(filter_data) < 1 || is.na(filter_data$filter[1])) {
  stop("filter.tsv must contain a non-missing filter column.")
}

filter_text <- trimws(filter_data$filter[1])
if (!nzchar(filter_text)) {
  stop("The filter expression is empty.")
}
filter_expr <- parse(text = filter_text)[[1]]

keep_through_first_selected_visit <- function(dataset, filter_expr) {
  if (nrow(dataset) == 0) {
    return(dataset)
  }

  selected <- eval(filter_expr, envir = dataset, enclos = parent.frame())
  selected[is.na(selected)] <- FALSE
  if (!is.logical(selected) || length(selected) != nrow(dataset)) {
    stop("The filter condition must evaluate to one logical value per input row.")
  }

  dataset$..selected_filter <- selected
  dataset$..input_order <- seq_len(nrow(dataset))
  dataset <- dataset[order(dataset$USUBJID, dataset$AVISITN, dataset$..input_order), , drop = FALSE]

  pieces <- split(dataset, dataset$USUBJID, drop = TRUE)
  kept <- lapply(pieces, function(subject_data) {
    selected_rows <- which(subject_data$..selected_filter)
    if (length(selected_rows) == 0) {
      return(subject_data[0, , drop = FALSE])
    }

    first_selected_visit <- subject_data$AVISITN[selected_rows[1]]
    subject_data[subject_data$AVISITN <= first_selected_visit, , drop = FALSE]
  })

  result <- do.call(rbind, kept)
  result <- result[order(result$..input_order), , drop = FALSE]
  result$..selected_filter <- NULL
  result$..input_order <- NULL
  row.names(result) <- NULL
  result
}

result <- slice_derivation(
  dataset = dataset,
  derivation = keep_through_first_selected_visit,
  derivation_slice(filter = TRUE),
  args = params(filter_expr = filter_expr)
)

result <- result[, required_columns, drop = FALSE]

write.csv(result, file.path("outputs", "result.csv"), row.names = FALSE, na = "")
```
