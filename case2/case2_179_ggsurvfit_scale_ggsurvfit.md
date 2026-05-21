# Case 179: pharmaverse/ggsurvfit/scale_ggsurvfit

## Metadata
- Task ID: `pharmaverse/ggsurvfit/scale_ggsurvfit`
- Package: `ggsurvfit`
- Model: `openai/gpt-5.1`
- Status: `PASS`
- Failure stage: `pass`

## Pass/Error
```text
PASS
```

## Prompt
```text
Write an R script for an R package task: perform scale ggsurvfit using the ggsurvfit package. Use functions from the ggsurvfit package when suitable.

Output: result.csv

Computation: Parse the x_scales and y_scales list expressions, call ggsurvfit::scale_ggsurvfit, and write a one-row class/summary table rather than the raw object.


Required columns for result.csv: scale_classes
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `x_scales.tsv`

```text
expr
list(expand = c(0.015, 0), n.breaks = 8)
```

#### `y_scales.tsv`

```text
expr
list(expand = c(0.025, 0), limits = c(0, 1))
```

## Code
### Ground Truth Code

```r
suppressPackageStartupMessages(library(ggsurvfit))

parse_scale_list <- function(fname) {
  df <- read.delim(file.path("inputs", fname), check.names = FALSE, stringsAsFactors = FALSE)
  txt <- trimws(as.character(df[[1]][1]))
  value <- eval(parse(text = txt))
  if (!is.list(value)) {
    stop(fname, " must contain an expression that evaluates to a list")
  }
  value
}

x_scales <- parse_scale_list("x_scales.tsv")
y_scales <- parse_scale_list("y_scales.tsv")
result <- ggsurvfit::scale_ggsurvfit(x_scales = x_scales, y_scales = y_scales)

result_df <- data.frame(
  scale_classes = paste(class(result), collapse = "|"),
  stringsAsFactors = FALSE
)

dir.create("outputs", showWarnings = FALSE)
write.csv(result_df, file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

suppressPackageStartupMessages({
  library(ggsurvfit)
  library(readr)
  library(dplyr)
})

# ensure outputs directory exists
if (!dir.exists("outputs")) dir.create("outputs", recursive = TRUE)

# read inputs
x_tbl <- readr::read_tsv("inputs/x_scales.tsv", show_col_types = FALSE)
y_tbl <- readr::read_tsv("inputs/y_scales.tsv", show_col_types = FALSE)

# parse list expressions safely
parse_list_expr <- function(expr_chr) {
  expr_parsed <- parse(text = expr_chr)[[1L]]
  val <- eval(expr_parsed, envir = baseenv())
  if (!is.list(val)) {
    stop("Parsed expression is not a list: ", expr_chr)
  }
  val
}

x_scales <- parse_list_expr(x_tbl$expr[[1L]])
y_scales <- parse_list_expr(y_tbl$expr[[1L]])

# call scale_ggsurvfit
scaled_obj <- ggsurvfit::scale_ggsurvfit(x = x_scales, y = y_scales)

# build one-row class/summary table
scale_classes <- paste(class(scaled_obj), collapse = ";")

result <- tibble::tibble(scale_classes = scale_classes)

# write result
readr::write_csv(result, "outputs/result.csv")
```

## Output
### Ground Truth Output
#### `result.csv`

```csv
"scale_classes"
"scale_ggsurvfit"
```

### LLM Output
#### `result.csv`

```csv
scale_classes
scale_ggsurvfit
```
