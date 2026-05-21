# Case 180: pharmaverse/gridify/get_layouts

## Metadata
- Task ID: `pharmaverse/gridify/get_layouts`
- Package: `gridify`
- Model: `openai/gpt-5.1`
- Status: `FAIL`
- Failure stage: `schema_mismatch`

## Pass/Error
```text
The generated output files were produced, but the schema or output shape differed from the ground truth.

Main signal:
result.csv: Shape mismatch: ref=(5, 2) vs llm=(1, 2)
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

suppressPackageStartupMessages({
  library(gridify)
})

# Ensure outputs directory exists
if (!dir.exists("outputs")) {
  dir.create("outputs", recursive = TRUE)
}

# Call gridify::get_layouts with default behavior
layouts <- gridify::get_layouts()

# Normalize to required columns: id, result
# Handle different possible structures of the returned object
normalize_layouts <- function(x) {
  if (is.data.frame(x)) {
    df <- x
    if (!"id" %in% names(df)) {
      df$id <- seq_len(nrow(df))
    }
    df$result <- apply(df, 1, function(row) {
      paste(names(row), as.character(row), sep = "=", collapse = ";")
    })
    out <- df[, c("id", "result")]
    out$id <- as.character(out$id)
    return(out)
  }
  
  if (is.list(x) && !is.data.frame(x)) {
    ids <- names(x)
    if (is.null(ids) || any(ids == "")) {
      ids <- seq_along(x)
    }
    res <- vapply(
      x,
      function(el) {
        if (is.null(el)) {
          return("NULL")
        }
        if (is.atomic(el) && length(el) == 1) {
          return(as.character(el))
        }
        if (is.atomic(el)) {
          return(paste(as.character(el), collapse = ","))
        }
        if (is.list(el) || is.data.frame(el)) {
          return(paste(capture.output(str(el, give.attr = FALSE)), collapse = " "))
        }
        paste(capture.output(print(el)), collapse = " ")
      },
      character(1)
    )
    return(data.frame(id = as.character(ids), result = res, stringsAsFactors = FALSE))
  }
  
  # Fallback: treat as single result
  data.frame(
    id = "1",
    result = paste(capture.output(str(x, give.attr = FALSE)), collapse = " "),
    stringsAsFactors = FALSE
  )
}

result_df <- normalize_layouts(layouts)

# Coerce to required structure and types
result_df <- result_df[, c("id", "result")]
result_df$id <- as.character(result_df$id)
result_df$result <- as.character(result_df$result)

# Write output
output_path <- file.path("outputs", "result.csv")
write.csv(result_df, file = output_path, row.names = FALSE, quote = TRUE)
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
"1"," chr [1:5] ""complex_layout"" ""simple_layout"" ""pharma_layout_base"" ..."
```
