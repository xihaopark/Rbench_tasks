# Case 010: pharmaverse/aNCA/clean_deparse

## Metadata
- Task ID: `pharmaverse/aNCA/clean_deparse`
- Package: `aNCA`
- Model: `openai/gpt-5.1`
- Status: `FAIL`
- Failure stage: `value_mismatch`

## Pass/Error
```text
The generated output files were produced, but one or more values differed from the
ground truth.

Main signal:
result.csv: Value mismatch in column: result.
```

## Prompt
```text
Write an R script to perform clean deparse using the aNCA clinical task contract.

Input: indent.tsv, max_per_line.tsv, min_to_rep.tsv, obj.tsv
Output: result.csv


Required columns for result.csv: result, indent
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `indent.tsv`


```text
indent
2
```

#### `max_per_line.tsv`


```text
max_per_line
80
```

#### `min_to_rep.tsv`


```text
min_to_rep
3
```

#### `obj.tsv`


```text
obj
alpha
beta
gamma
```

## Code
### Ground Truth Code

```r
has_aNCA <- requireNamespace("aNCA", quietly = TRUE)

# 1. Read input data
obj_path <- file.path("inputs", "obj.tsv")
if (!file.exists(obj_path)) {
  stop("obj.tsv is required input")
}
obj_df <- read.delim(obj_path, check.names = FALSE, stringsAsFactors = FALSE)
# obj may be any R object; try to parse it
if (ncol(obj_df) == 1 && nrow(obj_df) == 1) {
  obj_str <- obj_df[[1]][1]
  # Try eval parsing
  obj <- tryCatch({
    eval(parse(text = obj_str))
  }, error = function(e) {
    # If parsing fails, use the string value
    obj_str
  })
} else if (ncol(obj_df) == 1) {
  # If this is a single-column multi-row input, use a vector
  obj <- obj_df[[1]]
} else {
  # If this is multi-column data, use a data frame
  obj <- obj_df
}
max_per_line_path <- file.path("inputs", "max_per_line.tsv")
if (!file.exists(max_per_line_path)) {
  stop("max_per_line.tsv is required input")
}
max_per_line_df <- read.delim(max_per_line_path, check.names = FALSE, stringsAsFactors = FALSE)
max_per_line <- suppressWarnings(as.numeric(max_per_line_df[[1]][1]))
if (length(max_per_line) > 1) max_per_line <- max_per_line[1]
if (is.na(max_per_line)) max_per_line <- 80
min_to_rep_path <- file.path("inputs", "min_to_rep.tsv")
if (!file.exists(min_to_rep_path)) {
  stop("min_to_rep.tsv is required input")
}
min_to_rep_df <- read.delim(min_to_rep_path, check.names = FALSE, stringsAsFactors = FALSE)
min_to_rep <- suppressWarnings(as.numeric(min_to_rep_df[[1]][1]))
if (length(min_to_rep) > 1) min_to_rep <- min_to_rep[1]
if (is.na(min_to_rep)) min_to_rep <- 3
indent_path <- file.path("inputs", "indent.tsv")
if (!file.exists(indent_path)) {
  stop("indent.tsv is required input")
}
indent_df <- read.delim(indent_path, check.names = FALSE, stringsAsFactors = FALSE)
indent <- suppressWarnings(as.numeric(indent_df[[1]][1]))
if (length(indent) > 1) indent <- indent[1]
if (is.na(indent)) indent <- 0

# 2. Validate data
# obj may be any R object, so no data frame validation is needed

# 3. Execute function implementation
# Handle tbl_df objects as data.frame
if (inherits(obj, "tbl_df")) obj <- as.data.frame(obj)

# clean_deparse has no method for character values; convert to data.frame or list
if (is.character(obj) && length(obj) > 0) {
  obj <- as.data.frame(matrix(obj, ncol = 1), stringsAsFactors = FALSE)
  colnames(obj) <- "value"
}

# Handle trivial length-0 constructors (character(0), numeric(0), list(), data.frame(), ...)
if (length(obj) == 0 && !is.null(obj)) {
  result <- paste0(class(obj)[1], "()")
} else {
  # clean_deparse is an S3 generic; calling it directly dispatches to the correct method
  result <- tryCatch(
    {
      if (!has_aNCA) stop("aNCA unavailable")
      aNCA:::clean_deparse(obj, max_per_line = max_per_line, min_to_rep = min_to_rep, indent = indent)
    },
    error = function(e) deparse(obj)
  )
}

# 4. Create result dataframe
# Combine function results with input data when possible
if (is.vector(result) || is.numeric(result) || is.character(result)) {
  # result may be a multi-line string; normalize it to one line
  result_str <- paste(as.character(result), collapse = "\n")
  result_df <- data.frame(
    result = result_str,
    indent = indent,
    stringsAsFactors = FALSE)
} else if (is.data.frame(result)) {
  result_df <- result
} else {
  result_df <- data.frame(result = as.character(result), stringsAsFactors = FALSE)
}

# 6. Generate summary statistics
if (is.data.frame(result_df)) {
  summary_stats <- data.frame(
    n_rows = nrow(result_df),
    n_cols = ncol(result_df),
    col_names = paste(colnames(result_df), collapse = ", "),
    stringsAsFactors = FALSE)
  } else if (is.vector(result) || is.numeric(result)) {
  summary_stats <- data.frame(
    n_values = length(result),
    mean_value = mean(result, na.rm = TRUE),
    min_value = min(result, na.rm = TRUE),
    max_value = max(result, na.rm = TRUE),
    median_value = median(result, na.rm = TRUE),
    stringsAsFactors = FALSE)
  } else {
  summary_stats <- data.frame(
    result_type = class(result)[1],
    stringsAsFactors = FALSE
  )
}

# 7. Save outputs
outputs_dir <- "outputs"
dir.create(outputs_dir, showWarnings = FALSE)
unlink(file.path(outputs_dir, c("result.rds", "summary.csv")))
if (exists("result_df") && is.data.frame(result_df)) {
  write.csv(result_df, file.path(outputs_dir, "result.csv"), row.names = FALSE)
  } else if (exists("result")) {
  if (is.data.frame(result)) {
    write.csv(result, file.path(outputs_dir, "result.csv"), row.names = FALSE)
  } else if (is.list(result)) {
    saveRDS(result, file.path(outputs_dir, "result.rds"))
  } else if (is.vector(result) || is.numeric(result) || is.character(result)) {
    result_df <- data.frame(result = result, stringsAsFactors = FALSE)
    write.csv(result_df, file.path(outputs_dir, "result.csv"), row.names = FALSE)
  } else {
    saveRDS(result, file.path(outputs_dir, "result.rds"))
  }
} else {
  stop("No result variable found. Function implementation may not have returned a value.")
}
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

suppressPackageStartupMessages({
  # no extra packages needed
})

# Create outputs directory if it doesn't exist
if (!dir.exists("outputs")) dir.create("outputs", recursive = TRUE)

# Read inputs
indent_df      <- read.table("inputs/indent.tsv", header = TRUE, sep = "\t", stringsAsFactors = FALSE, quote = "")
max_per_line_df<- read.table("inputs/max_per_line.tsv", header = TRUE, sep = "\t", stringsAsFactors = FALSE, quote = "")
min_to_rep_df  <- read.table("inputs/min_to_rep.tsv", header = TRUE, sep = "\t", stringsAsFactors = FALSE, quote = "")
obj_df         <- read.table("inputs/obj.tsv", header = TRUE, sep = "\t", stringsAsFactors = FALSE, quote = "")

indent      <- as.integer(indent_df$indent[1])
max_per_line<- as.integer(max_per_line_df$max_per_line[1])
min_to_rep  <- as.integer(min_to_rep_df$min_to_rep[1])

# Construct the object to deparse
obj <- obj_df[[1]]

# Clean deparse function following aNCA-style clinical task contract ideas:
# - Respect indent
# - Do not exceed max characters per line when possible
# - Only introduce repetition constructs (rep()) when length >= min_to_rep
clean_deparse <- function(x, indent = 2, max_per_line = 80, min_to_rep = 3) {
  base <- deparse(x, width.cutoff = max_per_line)

  # If no repetition or length too small, just re-indent
  if (length(x) < min_to_rep) {
    ind_str <- paste0(rep(" ", indent), collapse = "")
    if (length(base) == 1L) {
      return(list(result = base, indent = as.integer(indent)))
    } else {
      base[2:length(base)] <- paste0(ind_str, base[2:length(base)])
      return(list(result = base, indent = as.integer(indent)))
    }
  }

  # Detect simple repetition patterns; if all elements identical, use rep()
  all_same <- length(unique(x)) == 1L
  if (all_same && length(x) >= min_to_rep) {
    val <- deparse(x[1], width.cutoff = max_per_line)
    if (length(val) > 1L) {
      val <- paste(val, collapse = " ")
    }
    rep_call <- paste0("rep(", val, ", ", length(x), ")")
    return(list(result = rep_call, indent = as.integer(indent)))
  }

  # Fallback to standard deparse with indent fix
  ind_str <- paste0(rep(" ", indent), collapse = "")
  if (length(base) > 1L) {
    base[2:length(base)] <- paste0(ind_str, base[2:length(base)])
  }
  list(result = base, indent = as.integer(indent))
}

res <- clean_deparse(obj, indent = indent, max_per_line = max_per_line, min_to_rep = min_to_rep)

# Normalize result to a single string (join lines with newline if multiple)
result_str <- if (length(res$result) == 1L) res$result else paste(res$result, collapse = "\n")

out_df <- data.frame(
  result = result_str,
  indent = res$indent,
  stringsAsFactors = FALSE
)

write.csv(out_df, file = "outputs/result.csv", row.names = FALSE, quote = TRUE)
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"result","indent"
"structure(list(value = c(""alpha"", ""beta"", ""gamma"")), class = ""data.frame"", row.names = c(NA,
-3L))",2
```

### LLM Output
#### `result.csv`


```csv
"result","indent"
"c(""alpha"", ""beta"", ""gamma"")",2
```
