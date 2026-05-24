# Case 034: pharmaverse/aNCA/interval_add_impute

## Metadata
- Task ID: `pharmaverse/aNCA/interval_add_impute`
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

Write an R script for an R package task: perform interval add impute using the aNCA package. Use functions from the aNCA package when suitable.

Input: after.tsv, data.tsv, target_groups.tsv, target_impute.tsv, target_params.tsv
Output: result.csv

Computation: For target group rows and target parameter columns, split interval rows when needed, add the target impute token at the requested position, clear non-target parameter flags on inserted rows, and preserve original ordering with inserted rows immediately after their source row.


Required columns for result.csv: start, end, cmax, auclast, half.life, impute, analyte, period
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `after.tsv`

```text
after
1
```

#### `data.tsv`

```text
start	end	cmax	auclast	half.life	impute	analyte	period
0	12	TRUE	TRUE	FALSE	start_predose	DrugA	Single
0	Inf	TRUE	FALSE	TRUE		DrugB	Single
12	24	TRUE	TRUE	TRUE	start_conc0	DrugA	Multiple
```

#### `target_groups.tsv`

```text
analyte	period
DrugA	Single
```

#### `target_impute.tsv`

```text
target_impute
start_conc0
```

#### `target_params.tsv`

```text
target_params
cmax
half.life
```

## Code
### Ground Truth Code

```r
read_tsv <- function(name) {
  read.delim(file.path("inputs", name), check.names = FALSE, stringsAsFactors = FALSE)
}

as_logical_df <- function(df) {
  as.data.frame(lapply(df, function(x) {
    if (is.logical(x)) {
      return(x)
    }
    ifelse(is.na(x), FALSE, tolower(as.character(x)) %in% c("true", "t", "1", "yes", "y"))
  }), check.names = FALSE)
}

add_impute_method <- function(impute_vals, target_impute, after) {
  if (length(impute_vals) == 0) {
    return(impute_vals)
  }
  vapply(impute_vals, function(value) {
    parts <- strsplit(ifelse(is.na(value), "", value), "[ ,]+")[[1]]
    parts <- parts[nzchar(parts)]
    parts <- setdiff(parts, target_impute)
    insert_after <- if (is.infinite(after)) length(parts) else min(max(after, 0), length(parts))
    paste(append(parts, target_impute, after = insert_after), collapse = ",")
  }, FUN.VALUE = character(1))
}

data <- read_tsv("data.tsv")
target_impute <- as.character(read_tsv("target_impute.tsv")[[1]][1])
after <- suppressWarnings(as.numeric(read_tsv("after.tsv")[[1]][1]))
if (is.na(after)) {
  after <- Inf
}
target_params <- as.character(read_tsv("target_params.tsv")[[1]])
target_groups <- read_tsv("target_groups.tsv")

required_group_cols <- names(target_groups)
missing_group_cols <- setdiff(required_group_cols, names(data))
if (length(missing_group_cols) > 0) {
  stop("target_groups columns are missing from data: ", paste(missing_group_cols, collapse = ", "))
}
missing_params <- setdiff(target_params, names(data))
if (length(missing_params) > 0) {
  stop("target_params columns are missing from data: ", paste(missing_params, collapse = ", "))
}
if (!"impute" %in% names(data)) {
  data$impute <- NA_character_
}
param_cols <- names(data)[vapply(data, is.logical, logical(1))]

index_col <- make.unique(c("index", names(data)))[1]
data[[index_col]] <- seq_len(nrow(data))
group_key <- do.call(paste, c(data[required_group_cols], sep = "\r"))
target_group_key <- do.call(paste, c(target_groups[required_group_cols], sep = "\r"))
is_target_group <- group_key %in% target_group_key
is_target_param <- rowSums(as_logical_df(data[target_params])) > 0

after_vals <- vapply(strsplit(ifelse(is.na(data$impute), "", data$impute), "[ ,]+"), function(parts) {
  parts <- parts[nzchar(parts)]
  pos <- which(parts == target_impute)
  if (length(pos) == 0) {
    return(NA_real_)
  }
  if (pos == length(parts)) {
    return(Inf)
  }
  pos
}, FUN.VALUE = numeric(1))
target_rows <- is_target_group & is_target_param & (is.na(after_vals) | after_vals != after)

new_intervals <- data[target_rows, , drop = FALSE]
if (nrow(new_intervals) > 0) {
  new_intervals[setdiff(param_cols, target_params)] <- FALSE
  new_intervals[target_params] <- data[target_rows, target_params, drop = FALSE]
  new_intervals$impute <- add_impute_method(new_intervals$impute, target_impute, after)
  new_intervals[[index_col]] <- new_intervals[[index_col]] + 0.5
  data[target_rows, target_params] <- FALSE
  data <- rbind(data, new_intervals)
}

param_state <- as_logical_df(data[param_cols])
data <- data[rowSums(param_state) > 0, , drop = FALSE]
result <- data[order(data[[index_col]]), setdiff(names(data), index_col), drop = FALSE]
rownames(result) <- NULL

dir.create("outputs", showWarnings = FALSE, recursive = TRUE)
unlink(file.path("outputs", c("summary.csv", "result.rds")))
write.csv(result, file.path("outputs", "result.csv"), row.names = FALSE, na = "")
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

if (requireNamespace("aNCA", quietly = TRUE)) {
  suppressPackageStartupMessages(library(aNCA))
}

read_tsv <- function(path, ...) {
  read.delim(
    path,
    sep = "\t",
    header = TRUE,
    stringsAsFactors = FALSE,
    na.strings = character(),
    check.names = FALSE,
    ...
  )
}

parse_logical_columns <- function(data, cols) {
  for (col in cols) {
    data[[col]] <- as.logical(data[[col]])
  }
  data
}

add_impute_method_local <- function(impute_vals, target_impute, after) {
  vapply(impute_vals, function(value) {
    if (is.na(value) || value == "") {
      methods <- character()
    } else {
      methods <- unlist(strsplit(value, "[ ,]+"))
      methods <- methods[nzchar(methods)]
    }
    methods <- setdiff(methods, target_impute)
    paste(append(methods, values = target_impute, after = after), collapse = ",")
  }, character(1))
}

target_group_rows <- function(data, target_groups) {
  if (is.null(target_groups) || nrow(target_groups) == 0) {
    return(rep(TRUE, nrow(data)))
  }
  group_cols <- names(target_groups)
  if (!all(group_cols %in% names(data))) {
    stop("target_groups contains columns absent from data: ",
         paste(setdiff(group_cols, names(data)), collapse = ", "))
  }
  apply(data[, group_cols, drop = FALSE], 1, function(data_row) {
    any(apply(target_groups[, group_cols, drop = FALSE], 1, function(group_row) {
      identical(as.character(data_row), as.character(group_row))
    }))
  })
}

target_after_rows <- function(impute_vals, target_impute, after) {
  vapply(impute_vals, function(value) {
    methods <- if (is.na(value) || value == "") character() else unlist(strsplit(value, "[ ,]+"))
    methods <- methods[nzchar(methods)]
    hit <- which(methods == target_impute)
    if (length(hit) == 0) {
      return(TRUE)
    }
    current_after <- if (hit[1] == length(methods)) Inf else hit[1]
    is.na(current_after) || current_after != after
  }, logical(1))
}

interval_add_impute_local <- function(data, target_impute, after, target_params, target_groups) {
  param_cols <- setdiff(names(data), c("start", "end", "impute", names(target_groups)))
  param_cols <- param_cols[vapply(data[param_cols], is.logical, logical(1))]
  if (!all(target_params %in% param_cols)) {
    stop("target_params contains columns absent from interval parameter columns: ",
         paste(setdiff(target_params, param_cols), collapse = ", "))
  }

  index_col <- make.unique(c("index", names(data)))[1]
  data[[index_col]] <- seq_len(nrow(data))

  target_param_data <- data[, target_params, drop = FALSE]
  target_rows <- target_group_rows(data, target_groups) &
    rowSums(replace(target_param_data, is.na(target_param_data), FALSE)) > 0 &
    target_after_rows(data$impute, target_impute, after)

  new_intervals <- data[target_rows, , drop = FALSE]
  if (nrow(new_intervals) == 0) {
    return(data[, !names(data) %in% index_col, drop = FALSE])
  }

  new_intervals[, setdiff(param_cols, target_params)] <- FALSE
  new_intervals[["impute"]] <- add_impute_method_local(new_intervals[["impute"]], target_impute, after)
  new_intervals[[index_col]] <- new_intervals[[index_col]] + 0.5

  data[target_rows, target_params] <- FALSE
  result <- rbind(data, new_intervals)

  param_data <- result[, param_cols, drop = FALSE]
  rows_no_params <- rowSums(replace(param_data, is.na(param_data), FALSE)) == 0
  result <- result[!rows_no_params, , drop = FALSE]
  result <- result[order(result[[index_col]]), , drop = FALSE]
  rownames(result) <- NULL
  result[, !names(result) %in% index_col, drop = FALSE]
}

data <- read_tsv("inputs/data.tsv", colClasses = c(start = "character", end = "character"))
target_groups <- read_tsv("inputs/target_groups.tsv")
target_impute <- read_tsv("inputs/target_impute.tsv")$target_impute[1]
after <- read_tsv("inputs/after.tsv")$after[1]
target_params <- read_tsv("inputs/target_params.tsv")$target_params

data$start <- as.numeric(data$start)
data$end <- as.numeric(data$end)
data <- parse_logical_columns(data, intersect(names(data), target_params))
other_flag_cols <- setdiff(names(data), c("start", "end", "impute", names(target_groups), target_params))
data <- parse_logical_columns(data, other_flag_cols[
  vapply(data[other_flag_cols], function(x) all(x %in% c("TRUE", "FALSE", TRUE, FALSE, NA)), logical(1))
])
data$impute[is.na(data$impute)] <- ""

use_anca <- requireNamespace("aNCA", quietly = TRUE) && ncol(target_groups) <= 1

result <- if (use_anca) {
  tryCatch(
    interval_add_impute(
      data = data,
      target_impute = target_impute,
      after = after,
      target_params = target_params,
      target_groups = target_groups
    ),
    error = function(e) interval_add_impute_local(data, target_impute, after, target_params, target_groups)
  )
} else {
  interval_add_impute_local(data, target_impute, after, target_params, target_groups)
}

required_cols <- c("start", "end", "cmax", "auclast", "half.life", "impute", "analyte", "period")
result <- result[, required_cols, drop = FALSE]

dir.create("outputs", showWarnings = FALSE, recursive = TRUE)
write.csv(result, "outputs/result.csv", row.names = FALSE, na = "")
```

## Output
### Ground Truth Output
#### `result.csv`

```csv
"start","end","cmax","auclast","half.life","impute","analyte","period"
0,12,FALSE,TRUE,FALSE,"start_predose","DrugA","Single"
0,12,TRUE,FALSE,FALSE,"start_predose,start_conc0","DrugA","Single"
0,Inf,TRUE,FALSE,TRUE,"","DrugB","Single"
12,24,TRUE,TRUE,TRUE,"start_conc0","DrugA","Multiple"
```

### LLM Output
#### `result.csv`

```csv
"start","end","cmax","auclast","half.life","impute","analyte","period"
0,12,FALSE,TRUE,FALSE,"start_predose","DrugA","Single"
0,12,TRUE,FALSE,FALSE,"start_predose,start_conc0","DrugA","Single"
0,Inf,TRUE,FALSE,TRUE,"","DrugB","Single"
12,24,TRUE,TRUE,TRUE,"start_conc0","DrugA","Multiple"
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

if (requireNamespace("aNCA", quietly = TRUE)) {
  suppressPackageStartupMessages(library(aNCA))
}

read_tsv <- function(path, ...) {
  read.delim(
    path,
    sep = "\t",
    header = TRUE,
    stringsAsFactors = FALSE,
    na.strings = character(),
    check.names = FALSE,
    ...
  )
}

parse_logical_columns <- function(data, cols) {
  for (col in cols) {
    data[[col]] <- as.logical(data[[col]])
  }
  data
}

add_impute_method_local <- function(impute_vals, target_impute, after) {
  vapply(impute_vals, function(value) {
    if (is.na(value) || value == "") {
      methods <- character()
    } else {
      methods <- unlist(strsplit(value, "[ ,]+"))
      methods <- methods[nzchar(methods)]
    }
    methods <- setdiff(methods, target_impute)
    paste(append(methods, values = target_impute, after = after), collapse = ",")
  }, character(1))
}

target_group_rows <- function(data, target_groups) {
  if (is.null(target_groups) || nrow(target_groups) == 0) {
    return(rep(TRUE, nrow(data)))
  }
  group_cols <- names(target_groups)
  if (!all(group_cols %in% names(data))) {
    stop("target_groups contains columns absent from data: ",
         paste(setdiff(group_cols, names(data)), collapse = ", "))
  }
  apply(data[, group_cols, drop = FALSE], 1, function(data_row) {
    any(apply(target_groups[, group_cols, drop = FALSE], 1, function(group_row) {
      identical(as.character(data_row), as.character(group_row))
    }))
  })
}

target_after_rows <- function(impute_vals, target_impute, after) {
  vapply(impute_vals, function(value) {
    methods <- if (is.na(value) || value == "") character() else unlist(strsplit(value, "[ ,]+"))
    methods <- methods[nzchar(methods)]
    hit <- which(methods == target_impute)
    if (length(hit) == 0) {
      return(TRUE)
    }
    current_after <- if (hit[1] == length(methods)) Inf else hit[1]
    is.na(current_after) || current_after != after
  }, logical(1))
}

interval_add_impute_local <- function(data, target_impute, after, target_params, target_groups) {
  param_cols <- setdiff(names(data), c("start", "end", "impute", names(target_groups)))
  param_cols <- param_cols[vapply(data[param_cols], is.logical, logical(1))]
  if (!all(target_params %in% param_cols)) {
    stop("target_params contains columns absent from interval parameter columns: ",
         paste(setdiff(target_params, param_cols), collapse = ", "))
  }

  index_col <- make.unique(c("index", names(data)))[1]
  data[[index_col]] <- seq_len(nrow(data))

  target_param_data <- data[, target_params, drop = FALSE]
  target_rows <- target_group_rows(data, target_groups) &
    rowSums(replace(target_param_data, is.na(target_param_data), FALSE)) > 0 &
    target_after_rows(data$impute, target_impute, after)

  new_intervals <- data[target_rows, , drop = FALSE]
  if (nrow(new_intervals) == 0) {
    return(data[, !names(data) %in% index_col, drop = FALSE])
  }

  new_intervals[, setdiff(param_cols, target_params)] <- FALSE
  new_intervals[["impute"]] <- add_impute_method_local(new_intervals[["impute"]], target_impute, after)
  new_intervals[[index_col]] <- new_intervals[[index_col]] + 0.5

  data[target_rows, target_params] <- FALSE
  result <- rbind(data, new_intervals)

  param_data <- result[, param_cols, drop = FALSE]
  rows_no_params <- rowSums(replace(param_data, is.na(param_data), FALSE)) == 0
  result <- result[!rows_no_params, , drop = FALSE]
  result <- result[order(result[[index_col]]), , drop = FALSE]
  rownames(result) <- NULL
  result[, !names(result) %in% index_col, drop = FALSE]
}

data <- read_tsv("inputs/data.tsv", colClasses = c(start = "character", end = "character"))
target_groups <- read_tsv("inputs/target_groups.tsv")
target_impute <- read_tsv("inputs/target_impute.tsv")$target_impute[1]
after <- read_tsv("inputs/after.tsv")$after[1]
target_params <- read_tsv("inputs/target_params.tsv")$target_params

data$start <- as.numeric(data$start)
data$end <- as.numeric(data$end)
data <- parse_logical_columns(data, intersect(names(data), target_params))
other_flag_cols <- setdiff(names(data), c("start", "end", "impute", names(target_groups), target_params))
data <- parse_logical_columns(data, other_flag_cols[
  vapply(data[other_flag_cols], function(x) all(x %in% c("TRUE", "FALSE", TRUE, FALSE, NA)), logical(1))
])
data$impute[is.na(data$impute)] <- ""

use_anca <- requireNamespace("aNCA", quietly = TRUE) && ncol(target_groups) <= 1

result <- if (use_anca) {
  tryCatch(
    interval_add_impute(
      data = data,
      target_impute = target_impute,
      after = after,
      target_params = target_params,
      target_groups = target_groups
    ),
    error = function(e) interval_add_impute_local(data, target_impute, after, target_params, target_groups)
  )
} else {
  interval_add_impute_local(data, target_impute, after, target_params, target_groups)
}

required_cols <- c("start", "end", "cmax", "auclast", "half.life", "impute", "analyte", "period")
result <- result[, required_cols, drop = FALSE]

dir.create("outputs", showWarnings = FALSE, recursive = TRUE)
write.csv(result, "outputs/result.csv", row.names = FALSE, na = "")
```
