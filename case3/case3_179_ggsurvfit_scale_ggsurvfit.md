# Case 179: pharmaverse/ggsurvfit/scale_ggsurvfit

## Metadata
- Task ID: `pharmaverse/ggsurvfit/scale_ggsurvfit`
- Package: `ggsurvfit`
- Model: `openai/gpt-5.5`
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

dir.create("outputs", showWarnings = FALSE, recursive = TRUE)

read_list_expr <- function(path) {
  dat <- utils::read.delim(
    path,
    sep = "\t",
    header = TRUE,
    stringsAsFactors = FALSE,
    check.names = FALSE,
    quote = "",
    comment.char = ""
  )

  if (!"expr" %in% names(dat) || nrow(dat) < 1L) {
    return(list())
  }

  expr_txt <- dat$expr[[1]]
  if (is.na(expr_txt) || !nzchar(trimws(expr_txt))) {
    return(list())
  }

  val <- eval(parse(text = expr_txt), envir = baseenv())

  if (!is.list(val)) {
    stop("Parsed expression in ", path, " must evaluate to a list.")
  }

  val
}

x_scales <- read_list_expr(file.path("inputs", "x_scales.tsv"))
y_scales <- read_list_expr(file.path("inputs", "y_scales.tsv"))

scale_obj <- ggsurvfit::scale_ggsurvfit(
  x_scales = x_scales,
  y_scales = y_scales
)

class_summary <- function(x) {
  top_class <- paste(class(x), collapse = "/")

  if (is.list(x) && length(x) > 0L) {
    element_classes <- vapply(seq_along(x), function(i) {
      nm <- names(x)[[i]]
      cls <- paste(class(x[[i]]), collapse = "/")
      if (!is.null(nm) && nzchar(nm)) {
        paste0(nm, ":", cls)
      } else {
        cls
      }
    }, character(1))

    paste(c(top_class, element_classes), collapse = ";")
  } else {
    top_class
  }
}

result <- data.frame(
  scale_classes = class_summary(scale_obj),
  stringsAsFactors = FALSE
)

utils::write.csv(
  result,
  file = file.path("outputs", "result.csv"),
  row.names = FALSE
)
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
