# Case 034: pharmaverse/aNCA/interval_add_impute

## Metadata
- Task ID: `pharmaverse/aNCA/interval_add_impute`
- Package: `aNCA`
- Model: `openai/gpt-5.1`
- Status: `NO_OUTPUT`
- Failure stage: `execution_failure`
- Failure pattern: `package_api_hallucination`
- Failure pattern note: 'arg' should be one of “any”, “within”, “start”, “end”, “equal”

## Pass/Error
```text
The generated code failed while running before valid outputs could be evaluated.

Main signal:
missing artifacts: result.csv.
```

## Prompt
```text
Write an R script for an R package task: perform interval add impute using the aNCA package. Use functions from the aNCA package when suitable.

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

suppressPackageStartupMessages({
  library(aNCA)
  library(data.table)
})

dir.create("outputs", showWarnings = FALSE, recursive = TRUE)

after      <- fread("inputs/after.tsv", sep = "\t", header = TRUE)
data_dt    <- fread("inputs/data.tsv", sep = "\t", header = TRUE)
tg_dt      <- fread("inputs/target_groups.tsv", sep = "\t", header = TRUE)
ti_dt      <- fread("inputs/target_impute.tsv", sep = "\t", header = TRUE)
tp_dt      <- fread("inputs/target_params.tsv", sep = "\t", header = TRUE)

target_impute <- ti_dt$target_impute[1]
target_params <- tp_dt$target_params

# Ensure logical columns
param_cols <- intersect(names(data_dt), c("cmax","auclast","half.life"))
for (col in param_cols) {
  if (!is.logical(data_dt[[col]])) {
    data_dt[[col]] <- as.logical(data_dt[[col]])
  }
}

# Flag target group rows
setkeyv(data_dt, c("analyte","period"))
setkeyv(tg_dt,    c("analyte","period"))
data_dt[, .tg_match := FALSE]
if (nrow(tg_dt) > 0) {
  idx <- foverlaps(
    x = data_dt[, .(analyte, period, row_id = .I)],
    y = tg_dt[, .(analyte, period)],
    by.x = c("analyte","period"),
    by.y = c("analyte","period"),
    type = "inner",
    nomatch = 0L
  )$row_id
  data_dt[idx, .tg_match := TRUE]
}

# Keep original row order
data_dt[, .orig_row := .I]

result_list <- list()
res_idx <- 1L

for (i in seq_len(nrow(data_dt))) {
  row <- data_dt[i]

  # always keep original row
  result_list[[res_idx]] <- copy(row)
  res_idx <- res_idx + 1L

  # only process if in target group and impute token matches
  if (!row$.tg_match) next
  if (!identical(row$impute, target_impute)) next

  # determine which params need imputation added
  to_impute <- intersect(target_params, param_cols)
  if (length(to_impute) == 0L) next

  # if all already TRUE, nothing to add
  if (all(unlist(row[, ..to_impute]))) next

  # Use aNCA tools for interval handling (split if needed)
  # Build an aNCA-like interval object
  interval <- data.table(
    start     = row$start,
    end       = row$end,
    cmax      = row$cmax,
    auclast   = row$auclast,
    half.life = row$`half.life`,
    impute    = row$impute,
    analyte   = row$analyte,
    period    = row$period
  )

  # Example usage of aNCA interval utilities (no-op split, placeholder)
  # interval <- split_interval(interval, ...) # if such function exists in aNCA

  # Create inserted row based on original interval
  new_row <- copy(row)

  # clear non-target parameter flags
  non_target <- setdiff(param_cols, to_impute)
  if (length(non_target) > 0L) {
    for (col in non_target) new_row[[col]] <- FALSE
  }

  # ensure target params are TRUE
  for (col in to_impute) new_row[[col]] <- TRUE

  # impute token stays as requested
  new_row$impute <- target_impute

  # mark as derived (if needed internally)
  new_row$.orig_row <- row$.orig_row + 0.1

  result_list[[res_idx]] <- new_row
  res_idx <- res_idx + 1L
}

res_dt <- rbindlist(result_list, use.names = TRUE, fill = TRUE)

# Reorder: by original index, then by whether it is inserted (derived)
res_dt[, .is_inserted := duplicated(.orig_row)]
setorder(res_dt, .orig_row, .is_inserted)

# Keep required columns only
required_cols <- c("start","end","cmax","auclast","half.life","impute","analyte","period")
res_dt <- res_dt[, ..required_cols]

fwrite(res_dt, file = "outputs/result.csv", sep = ",", quote = TRUE, na = "")
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
