# Case 034: pharmaverse/aNCA/interval_add_impute

## Metadata
- Task ID: `pharmaverse/aNCA/interval_add_impute`
- Package: `aNCA`
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

Write an R script for an R package task: perform interval add impute using the aNCA package. Use functions from the aNCA package when suitable.

Input: after.tsv, data.tsv, target_groups.tsv, target_impute.tsv, target_params.tsv
Output: result.csv

Computation: For target group rows and target parameter columns, split interval rows when needed, add the target impute token at the requested position, clear non-target parameter flags on inserted rows, and preserve original ordering with inserted rows immediately after their source row.


Required columns for result.csv: start, end, cmax, auclast, half.life, impute, analyte, period
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.

## Input preview

### after.tsv
after
1

### data.tsv
start	end	cmax	auclast	half.life	impute	analyte	period
0	12	TRUE	TRUE	FALSE	start_predose	DrugA	Single
0	Inf	TRUE	FALSE	TRUE		DrugB	Single
12	24	TRUE	TRUE	TRUE	start_conc0	DrugA	Multiple

### target_groups.tsv
analyte	period
DrugA	Single

### target_impute.tsv
target_impute
start_conc0

### target_params.tsv
target_params
cmax
half.life
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
# solution.R — interval_add_impute (aNCA-compatible implementation)
# Implements the logic from aNCA::interval_add_impute.data.frame

# ── read inputs ────────────────────────────────────────────────────────────────
data          <- read.table("inputs/data.tsv",           header = TRUE, sep = "\t",
                            stringsAsFactors = FALSE, check.names = FALSE,
                            na.strings = "")
after_df      <- read.table("inputs/after.tsv",          header = TRUE, sep = "\t",
                            stringsAsFactors = FALSE)
target_groups <- read.table("inputs/target_groups.tsv",  header = TRUE, sep = "\t",
                            stringsAsFactors = FALSE, check.names = FALSE)
target_impute <- read.table("inputs/target_impute.tsv",  header = TRUE, sep = "\t",
                            stringsAsFactors = FALSE)[[1]]
target_params <- read.table("inputs/target_params.tsv",  header = TRUE, sep = "\t",
                            stringsAsFactors = FALSE)[[1]]

after <- after_df$after[1]   # numeric position (0 = before first, Inf = last)

# ── helpers ────────────────────────────────────────────────────────────────────

# Add an impute token into every value of impute_vals at position `after`.
# Already-present copies are removed first to avoid duplicates.
add_impute_method <- function(impute_vals, target_impute, after) {
  if (length(impute_vals) == 0) return(impute_vals)
  impute_vals <- ifelse(is.na(impute_vals), "", impute_vals)
  vapply(strsplit(impute_vals, "[ ,]+"), function(x) {
    x <- x[nchar(x) > 0]          # drop empty strings from splitting ""
    x <- setdiff(x, target_impute) # remove any existing copy
    x <- append(x, target_impute, after = after)  # insert at position
    paste(x, collapse = ",")
  }, FUN.VALUE = "")
}

# Row-wise group matching: returns logical vector of length nrow(data).
# TRUE when a row matches any row of target_groups exactly.
is_in_target_groups <- function(data, target_groups) {
  group_cols  <- names(target_groups)
  data_keys   <- do.call(paste, c(data[, group_cols, drop = FALSE],   sep = "|||"))
  target_keys <- do.call(paste, c(target_groups,                      sep = "|||"))
  data_keys %in% target_keys
}

# Returns logical vector (TRUE = row NEEDS the impute to be (re-)added).
is_after_needed <- function(impute_col, target_impute, after) {
  vapply(impute_col, function(raw) {
    x   <- if (is.na(raw) || raw == "") character(0)
            else strsplit(raw, "[ ,]+")[[1]]
    x   <- x[nchar(x) > 0]
    pos <- which(x == target_impute)
    if (length(pos) == 0) return(TRUE)          # not present → needs adding
    actual_pos <- if (pos == length(x)) Inf else pos
    actual_pos != after
  }, FUN.VALUE = logical(1))
}

# ── parameter columns in this data frame ──────────────────────────────────────
# All columns that are not group/metadata columns, start, end, or impute.
non_param_cols <- unique(c("start", "end", "impute", names(target_groups),
                            names(data)[sapply(data, is.character)]))
param_cols     <- setdiff(names(data), non_param_cols)

# ── identify target rows ───────────────────────────────────────────────────────
is_tg  <- is_in_target_groups(data, target_groups)

target_params_data <- data[, target_params, drop = FALSE]
is_tp  <- rowSums(replace(target_params_data, is.na(target_params_data), FALSE)) > 0

is_aft <- is_after_needed(data$impute, target_impute, after)

target_rows <- is_tg & is_tp & is_aft

# ── apply transformations ─────────────────────────────────────────────────────
# Add index to preserve ordering
data$..index.. <- seq_len(nrow(data))

new_intervals <- data[target_rows, , drop = FALSE]

if (nrow(new_intervals) == 0) {
  warning("No intervals found with the specified target parameters, groups, ",
          "and/or after-change needed. No changes made.")
} else {
  # In the new rows: non-target param columns set to FALSE
  non_target_params <- setdiff(param_cols, target_params)
  new_intervals[, non_target_params] <- FALSE

  # Add the impute token at position `after`
  new_intervals$impute <- add_impute_method(new_intervals$impute, target_impute, after)

  # Position new rows immediately after their originals
  new_intervals$..index.. <- new_intervals$..index.. + 0.5

  # Remove target params from the original target rows (they move to new rows)
  data[target_rows, target_params] <- FALSE

  # Combine
  data <- rbind(data, new_intervals)

  # Drop rows where every param column is FALSE or NA
  param_data    <- data[, param_cols, drop = FALSE]
  rows_no_param <- rowSums(replace(param_data, is.na(param_data), FALSE)) == 0
  data <- data[!rows_no_param, , drop = FALSE]

  # Re-sort by index
  data <- data[order(data$..index..), ]
  rownames(data) <- NULL
}

# Remove index helper column
data$..index.. <- NULL

# ── select required output columns ────────────────────────────────────────────
required_cols <- c("start", "end", "cmax", "auclast", "half.life",
                   "impute", "analyte", "period")
result <- data[, required_cols, drop = FALSE]

# ── write output ──────────────────────────────────────────────────────────────
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
No LLM output artifact was produced.

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
# solution.R — interval_add_impute (aNCA-compatible implementation)
# Implements the logic from aNCA::interval_add_impute.data.frame

# ── read inputs ────────────────────────────────────────────────────────────────
data          <- read.table("inputs/data.tsv",           header = TRUE, sep = "\t",
                            stringsAsFactors = FALSE, check.names = FALSE,
                            na.strings = "")
after_df      <- read.table("inputs/after.tsv",          header = TRUE, sep = "\t",
                            stringsAsFactors = FALSE)
target_groups <- read.table("inputs/target_groups.tsv",  header = TRUE, sep = "\t",
                            stringsAsFactors = FALSE, check.names = FALSE)
target_impute <- read.table("inputs/target_impute.tsv",  header = TRUE, sep = "\t",
                            stringsAsFactors = FALSE)[[1]]
target_params <- read.table("inputs/target_params.tsv",  header = TRUE, sep = "\t",
                            stringsAsFactors = FALSE)[[1]]

after <- after_df$after[1]   # numeric position (0 = before first, Inf = last)

# ── helpers ────────────────────────────────────────────────────────────────────

# Add an impute token into every value of impute_vals at position `after`.
# Already-present copies are removed first to avoid duplicates.
add_impute_method <- function(impute_vals, target_impute, after) {
  if (length(impute_vals) == 0) return(impute_vals)
  impute_vals <- ifelse(is.na(impute_vals), "", impute_vals)
  vapply(strsplit(impute_vals, "[ ,]+"), function(x) {
    x <- x[nchar(x) > 0]          # drop empty strings from splitting ""
    x <- setdiff(x, target_impute) # remove any existing copy
    x <- append(x, target_impute, after = after)  # insert at position
    paste(x, collapse = ",")
  }, FUN.VALUE = "")
}

# Row-wise group matching: returns logical vector of length nrow(data).
# TRUE when a row matches any row of target_groups exactly.
is_in_target_groups <- function(data, target_groups) {
  group_cols  <- names(target_groups)
  data_keys   <- do.call(paste, c(data[, group_cols, drop = FALSE],   sep = "|||"))
  target_keys <- do.call(paste, c(target_groups,                      sep = "|||"))
  data_keys %in% target_keys
}

# Returns logical vector (TRUE = row NEEDS the impute to be (re-)added).
is_after_needed <- function(impute_col, target_impute, after) {
  vapply(impute_col, function(raw) {
    x   <- if (is.na(raw) || raw == "") character(0)
            else strsplit(raw, "[ ,]+")[[1]]
    x   <- x[nchar(x) > 0]
    pos <- which(x == target_impute)
    if (length(pos) == 0) return(TRUE)          # not present → needs adding
    actual_pos <- if (pos == length(x)) Inf else pos
    actual_pos != after
  }, FUN.VALUE = logical(1))
}

# ── parameter columns in this data frame ──────────────────────────────────────
# All columns that are not group/metadata columns, start, end, or impute.
non_param_cols <- unique(c("start", "end", "impute", names(target_groups),
                            names(data)[sapply(data, is.character)]))
param_cols     <- setdiff(names(data), non_param_cols)

# ── identify target rows ───────────────────────────────────────────────────────
is_tg  <- is_in_target_groups(data, target_groups)

target_params_data <- data[, target_params, drop = FALSE]
is_tp  <- rowSums(replace(target_params_data, is.na(target_params_data), FALSE)) > 0

is_aft <- is_after_needed(data$impute, target_impute, after)

target_rows <- is_tg & is_tp & is_aft

# ── apply transformations ─────────────────────────────────────────────────────
# Add index to preserve ordering
data$..index.. <- seq_len(nrow(data))

new_intervals <- data[target_rows, , drop = FALSE]

if (nrow(new_intervals) == 0) {
  warning("No intervals found with the specified target parameters, groups, ",
          "and/or after-change needed. No changes made.")
} else {
  # In the new rows: non-target param columns set to FALSE
  non_target_params <- setdiff(param_cols, target_params)
  new_intervals[, non_target_params] <- FALSE

  # Add the impute token at position `after`
  new_intervals$impute <- add_impute_method(new_intervals$impute, target_impute, after)

  # Position new rows immediately after their originals
  new_intervals$..index.. <- new_intervals$..index.. + 0.5

  # Remove target params from the original target rows (they move to new rows)
  data[target_rows, target_params] <- FALSE

  # Combine
  data <- rbind(data, new_intervals)

  # Drop rows where every param column is FALSE or NA
  param_data    <- data[, param_cols, drop = FALSE]
  rows_no_param <- rowSums(replace(param_data, is.na(param_data), FALSE)) == 0
  data <- data[!rows_no_param, , drop = FALSE]

  # Re-sort by index
  data <- data[order(data$..index..), ]
  rownames(data) <- NULL
}

# Remove index helper column
data$..index.. <- NULL

# ── select required output columns ────────────────────────────────────────────
required_cols <- c("start", "end", "cmax", "auclast", "half.life",
                   "impute", "analyte", "period")
result <- data[, required_cols, drop = FALSE]

# ── write output ──────────────────────────────────────────────────────────────
dir.create("outputs", showWarnings = FALSE, recursive = TRUE)
write.csv(result, "outputs/result.csv", row.names = FALSE, na = "")
```
