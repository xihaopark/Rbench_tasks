# Case2 30: pharmaverse/gridify/get_layouts

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
Write an R script for an R package task: perform get layouts using the gridify package. The function(s) you use to solve the task must come from the gridify package.

Input: none
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

# Call gridify::get_layouts (no inputs specified)
layouts <- gridify::get_layouts()

# Normalize to id and result columns
# Convert the layouts object to a data.frame-like structure
# and store a compact representation in 'result'
norm_df <- NULL

if (is.data.frame(layouts)) {
  layouts$result <- apply(layouts, 1, function(row) {
    paste(names(row), as.character(row), sep = "=", collapse = ";")
  })
  layouts$id <- seq_len(nrow(layouts))
  norm_df <- layouts[, c("id", "result")]
} else if (is.list(layouts)) {
  norm_df <- data.frame(
    id = seq_along(layouts),
    result = vapply(
      layouts,
      function(x) {
        if (is.data.frame(x) || is.matrix(x)) {
          paste(
            apply(as.data.frame(x), 1, function(row) {
              paste(names(row), as.character(row), sep = "=", collapse = ",")
            }),
            collapse = "|"
          )
        } else if (is.list(x)) {
          paste(
            paste(names(x), vapply(x, toString, character(1)), sep = "=", collapse = ";"),
            collapse = ";"
          )
        } else {
          toString(x)
        }
      },
      character(1)
    ),
    stringsAsFactors = FALSE
  )
} else {
  norm_df <- data.frame(
    id = 1L,
    result = toString(layouts),
    stringsAsFactors = FALSE
  )
}

# Coerce column types
norm_df$id <- as.integer(norm_df$id)
norm_df$result <- as.character(norm_df$result)

# Write result.csv
write.csv(norm_df[, c("id", "result")],
          file = "outputs/result.csv",
          row.names = FALSE)
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
1,"complex_layout, simple_layout, pharma_layout_base, pharma_layout_A4, pharma_layout_letter"
```
