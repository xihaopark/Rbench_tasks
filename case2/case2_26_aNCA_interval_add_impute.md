# Case2 26: pharmaverse/aNCA/interval_add_impute

## Metadata
- Task ID: `pharmaverse/aNCA/interval_add_impute`
- Package: `aNCA`
- Model: `openai/gpt-5.1`
- Status: `NO_OUTPUT`
- Failure stage: `execution_failure`

## Pass/Error
```text
The generated code failed while running before valid outputs could be evaluated.

Main signal:
missing artifacts: result.csv.
```

## Prompt
```text
Write an R script for an R package task: perform interval add impute using the aNCA package. The function(s) you use to solve the task must come from the aNCA package.

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

suppressPackageStartupMessages({
  library(aNCA)
  library(data.table)
})

# Create outputs directory
if (!dir.exists("outputs")) dir.create("outputs", recursive = TRUE)

# Read inputs
after  <- fread("inputs/after.tsv", sep = "\t", header = TRUE)
data   <- fread("inputs/data.tsv", sep = "\t", header = TRUE)
tgrps  <- fread("inputs/target_groups.tsv", sep = "\t", header = TRUE)
timpt  <- fread("inputs/target_impute.tsv", sep = "\t", header = TRUE)
tpar   <- fread("inputs/target_params.tsv", sep = "\t", header = TRUE)

# Coerce infinities if any
if ("end" %in% names(data)) {
  data[end %in% c("Inf", "inf", ""), end := Inf]
}

# Use interval_add_impute from aNCA
res <- interval_add_impute(
  data             = data,
  after            = after$after,
  target_impute    = timpt$target_impute,
  target_groups    = tgrps,
  target_params    = tpar$target_params,
  key_cols         = c("analyte", "period"),
  start_col        = "start",
  end_col          = "end",
  impute_col       = "impute"
)

# Ensure required columns and order
req_cols <- c("start", "end", "cmax", "auclast", "half.life", "impute", "analyte", "period")
res <- res[, ..req_cols]

# Write output
fwrite(res, file = "outputs/result.csv", quote = TRUE)
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
