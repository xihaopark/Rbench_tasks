# Case 180: pharmaverse/gridify/get_layouts

## Metadata
- Task ID: `pharmaverse/gridify/get_layouts`
- Package: `gridify`
- Model: `openai/gpt-5.5`
- Status: `PASS`
- Failure stage: `pass`

## Pass/Error
```text
PASS
```

## Prompt
```text
Write an R script for an R package task: perform get layouts using the gridify package. Use functions from the gridify package when suitable.

Output: result.csv

Computation: Call gridify::get_layouts and normalize the returned layouts to result.csv columns id and result instead of writing the raw object.


Required columns for result.csv: id, result
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
No input files.

## Code
### Ground Truth Code

```r
suppressPackageStartupMessages(library(gridify))

dir.create("outputs", showWarnings = FALSE, recursive = TRUE)

result <- gridify::get_layouts()
result_df <- data.frame(
  id = seq_along(result),
  result = as.character(result),
  stringsAsFactors = FALSE
)

write.csv(result_df, file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

if (!requireNamespace("gridify", quietly = TRUE)) {
  stop("Package 'gridify' is required.", call. = FALSE)
}

dir.create("outputs", showWarnings = FALSE, recursive = TRUE)

layouts <- gridify::get_layouts()

to_text <- function(x) {
  if (is.null(x)) {
    return(NA_character_)
  }
  if (length(x) == 1L && (is.atomic(x) || is.factor(x))) {
    return(as.character(x))
  }
  paste(capture.output(str(x, give.attr = FALSE)), collapse = "\n")
}

if (is.null(layouts)) {
  result_df <- data.frame(
    id = character(),
    result = character(),
    stringsAsFactors = FALSE
  )
} else if (is.data.frame(layouts)) {
  if (all(c("id", "result") %in% names(layouts))) {
    result_df <- layouts[, c("id", "result"), drop = FALSE]
  } else {
    ids <- if (!is.null(rownames(layouts))) rownames(layouts) else seq_len(nrow(layouts))
    result_df <- data.frame(
      id = as.character(ids),
      result = apply(layouts, 1L, function(row) paste(row, collapse = ";")),
      stringsAsFactors = FALSE
    )
  }
} else if (is.atomic(layouts) || is.factor(layouts)) {
  ids <- names(layouts)
  if (is.null(ids)) {
    ids <- seq_along(layouts)
  }
  result_df <- data.frame(
    id = as.character(ids),
    result = as.character(layouts),
    stringsAsFactors = FALSE
  )
} else if (is.list(layouts)) {
  ids <- names(layouts)
  if (is.null(ids) || any(ids == "")) {
    ids <- seq_along(layouts)
  }
  result_df <- data.frame(
    id = as.character(ids),
    result = vapply(layouts, to_text, character(1L)),
    stringsAsFactors = FALSE
  )
} else {
  result_df <- data.frame(
    id = "1",
    result = to_text(layouts),
    stringsAsFactors = FALSE
  )
}

result_df <- result_df[, c("id", "result"), drop = FALSE]

utils::write.csv(
  result_df,
  file = file.path("outputs", "result.csv"),
  row.names = FALSE,
  na = ""
)
```

## Output
### Ground Truth Output
#### `result.csv`

```csv
"id","result"
1,"complex_layout"
2,"simple_layout"
3,"pharma_layout_base"
4,"pharma_layout_A4"
5,"pharma_layout_letter"
```

### LLM Output
#### `result.csv`

```csv
"id","result"
"1","complex_layout"
"2","simple_layout"
"3","pharma_layout_base"
"4","pharma_layout_A4"
"5","pharma_layout_letter"
```
