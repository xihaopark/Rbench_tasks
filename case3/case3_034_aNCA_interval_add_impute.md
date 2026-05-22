# Case 034: pharmaverse/aNCA/interval_add_impute

## Metadata
- Task ID: `pharmaverse/aNCA/interval_add_impute`
- Package: `aNCA`
- Model: `openai/gpt-5.5`
- Status: `FAIL`
- Failure stage: `value_mismatch`

## Pass/Error
```text
The generated output files were produced, but one or more values differed from the ground truth.

Main signal:
result.csv: Numeric mismatch in column: half.life
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
suppressPackageStartupMessages({
  if (requireNamespace("aNCA", quietly = TRUE)) {
    library(aNCA)
  }
})

dir.create("outputs", showWarnings = FALSE, recursive = TRUE)

read_tsv <- function(path) {
  read.delim(
    path,
    header = TRUE,
    sep = "\t",
    stringsAsFactors = FALSE,
    check.names = FALSE,
    na.strings = "NA"
  )
}

after_df <- read_tsv(file.path("inputs", "after.tsv"))
data <- read_tsv(file.path("inputs", "data.tsv"))
target_groups <- read_tsv(file.path("inputs", "target_groups.tsv"))
target_impute_df <- read_tsv(file.path("inputs", "target_impute.tsv"))
target_params_df <- read_tsv(file.path("inputs", "target_params.tsv"))

if ("impute" %in% names(data)) {
  data$impute[is.na(data$impute)] <- ""
}

target_impute <- trimws(as.character(target_impute_df[[1]][1]))
target_params <- trimws(as.character(target_params_df[[1]]))
target_params <- target_params[nzchar(target_params)]

after_pos <- suppressWarnings(as.numeric(as.character(after_df[[1]][1])))

group_cols <- names(target_groups)
parameter_cols <- setdiff(names(data), c("start", "end", "impute", group_cols))
target_parameter_cols <- intersect(target_params, names(data))
non_target_parameter_cols <- setdiff(parameter_cols, target_parameter_cols)

make_key <- function(df, cols) {
  if (length(cols) == 0) {
    return(rep("", nrow(df)))
  }
  vals <- lapply(df[cols], function(x) {
    y <- as.character(x)
    y[is.na(y)] <- "<NA>"
    y
  })
  do.call(paste, c(vals, sep = "\r"))
}

target_keys <- make_key(target_groups, group_cols)
data_keys <- make_key(data, group_cols)
is_target_group <- data_keys %in% target_keys

as_flag <- function(x) {
  if (is.logical(x)) {
    return(!is.na(x) & x)
  }
  y <- tolower(trimws(as.character(x)))
  !is.na(y) & y %in% c("true", "t", "1", "yes", "y")
}

has_true_flag <- function(row, cols) {
  if (length(cols) == 0) {
    return(FALSE)
  }
  any(unlist(lapply(row[cols], as_flag)), na.rm = TRUE)
}

set_flags <- function(row, cols, value) {
  if (length(cols) > 0) {
    for (col in cols) {
      row[[col]] <- value
    }
  }
  row
}

split_impute <- function(x) {
  x <- trimws(as.character(x))
  if (length(x) == 0 || is.na(x) || !nzchar(x)) {
    return(character(0))
  }
  parts <- unlist(strsplit(x, "\\s*[,;|]\\s*"))
  parts[nzchar(parts)]
}

add_impute_token <- function(x, token, after) {
  if (is.na(token) || !nzchar(token)) {
    return(as.character(x))
  }

  tokens <- split_impute(x)
  tokens <- tokens[tokens != token]

  if (is.na(after)) {
    pos <- length(tokens)
  } else if (is.infinite(after)) {
    pos <- length(tokens)
  } else {
    pos <- floor(after)
    pos <- max(0, min(pos, length(tokens)))
  }

  tokens <- append(tokens, token, after = pos)
  paste(tokens, collapse = ",")
}

result_rows <- list()

for (i in seq_len(nrow(data))) {
  source_row <- data[i, , drop = FALSE]

  if (isTRUE(is_target_group[i])) {
    needs_split <- has_true_flag(source_row, non_target_parameter_cols)

    if (needs_split) {
      retained_row <- set_flags(source_row, target_parameter_cols, FALSE)

      inserted_row <- source_row
      inserted_row <- set_flags(inserted_row, target_parameter_cols, TRUE)
      inserted_row <- set_flags(inserted_row, non_target_parameter_cols, FALSE)
      inserted_row$impute <- add_impute_token(inserted_row$impute, target_impute, after_pos)

      result_rows[[length(result_rows) + 1]] <- retained_row
      result_rows[[length(result_rows) + 1]] <- inserted_row
    } else {
      source_row <- set_flags(source_row, target_parameter_cols, TRUE)
      source_row$impute <- add_impute_token(source_row$impute, target_impute, after_pos)
      result_rows[[length(result_rows) + 1]] <- source_row
    }
  } else {
    result_rows[[length(result_rows) + 1]] <- source_row
  }
}

result <- if (length(result_rows) > 0) {
  do.call(rbind, result_rows)
} else {
  data[0, , drop = FALSE]
}

required_cols <- c("start", "end", "cmax", "auclast", "half.life", "impute", "analyte", "period")
result <- result[, required_cols, drop = FALSE]

write.csv(
  result,
  file = file.path("outputs", "result.csv"),
  row.names = FALSE,
  na = ""
)
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
0,12,TRUE,FALSE,TRUE,"start_predose,start_conc0","DrugA","Single"
0,Inf,TRUE,FALSE,TRUE,"","DrugB","Single"
12,24,TRUE,TRUE,TRUE,"start_conc0","DrugA","Multiple"
```
