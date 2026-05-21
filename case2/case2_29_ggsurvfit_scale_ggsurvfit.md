# Case2 29: pharmaverse/ggsurvfit/scale_ggsurvfit

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
Write an R script for an R package task: perform scale ggsurvfit using the ggsurvfit package. The function(s) you use to solve the task must come from the ggsurvfit package.

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
})

# ensure outputs directory exists
if (!dir.exists("outputs")) {
  dir.create("outputs", recursive = TRUE)
}

# read inputs
x_tbl <- read.delim("inputs/x_scales.tsv", stringsAsFactors = FALSE)
y_tbl <- read.delim("inputs/y_scales.tsv", stringsAsFactors = FALSE)

# parse list expressions
x_expr <- eval(parse(text = x_tbl$expr[[1]]))
y_expr <- eval(parse(text = y_tbl$expr[[1]]))

# call scale_ggsurvfit
scales_obj <- ggsurvfit::scale_ggsurvfit(x = x_expr, y = y_expr)

# create one-row class/summary table
scale_classes <- paste(class(scales_obj), collapse = ";")
res <- data.frame(scale_classes = scale_classes, stringsAsFactors = FALSE)

# write result
write.csv(res, file = "outputs/result.csv", row.names = FALSE)
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
"scale_classes"
"scale_ggsurvfit"
```
