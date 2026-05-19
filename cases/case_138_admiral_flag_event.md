# Case 138: pharmaverse/admiral/flag_event

## Metadata
- Task ID: `pharmaverse/admiral/flag_event`
- Package: `admiral`
- Model: `openai/gpt-5.1`
- Status: `FAIL`
- Failure stage: `schema_mismatch`

## Pass/Error
```text
The generated output file schema did not match the ground truth schema.

Main signal:
result.csv: Shape mismatch: ref=(4, 2) vs llm=(3, 2).
```

## Prompt
```text
Write an R script to perform flag event using the admiral clinical task contract.

Input: spec.tsv
Output: result.csv


Required columns for result.csv: field, value
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `spec.tsv`


```text
field	value
dataset_name	adlb
condition	AVAL > 10
by_vars	USUBJID,PARAMCD
```

## Code
### Ground Truth Code

```r
suppressPackageStartupMessages(library(admiral))
suppressPackageStartupMessages(library(rlang))

spec <- read.delim(file.path("inputs", "spec.tsv"), check.names = FALSE, stringsAsFactors = FALSE)
vals <- stats::setNames(spec$value, spec$field)
by_syms <- strsplit(vals[["by_vars"]], ",", fixed = TRUE)[[1]]
obj <- admiral::flag_event(
  dataset_name = vals[["dataset_name"]],
  condition = parse_expr(vals[["condition"]]),
  by_vars = as.list(lapply(by_syms, sym))
)
result <- data.frame(
  field = c("class", "dataset_name", "condition", "by_vars"),
  value = c(
    paste(class(obj), collapse = "|"),
    obj$dataset_name,
    paste(deparse(obj$condition), collapse = " "),
    paste(vapply(obj$by_vars, function(x) paste(deparse(x), collapse = " "), character(1)), collapse = ",")
  ),
  stringsAsFactors = FALSE
)
dir.create("outputs", showWarnings = FALSE)
write.csv(result, file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

# Load required packages
suppressPackageStartupMessages({
  library(readr)
  library(dplyr)
  library(stringr)
  library(purrr)
})

# Ensure outputs directory exists
if (!dir.exists("outputs")) dir.create("outputs", recursive = TRUE)

# Read spec
spec <- read_tsv(
  file = file.path("inputs", "spec.tsv"),
  show_col_types = FALSE
)

# Convert spec to named vector for easy access
spec_list <- setNames(spec$value, spec$field)

dataset_name <- spec_list[["dataset_name"]]
condition    <- spec_list[["condition"]]
by_vars_raw  <- spec_list[["by_vars"]]

# Parse by_vars into character vector
by_vars <- if (!is.na(by_vars_raw) && nzchar(by_vars_raw)) {
  str_split(by_vars_raw, ",")[[1]] |> str_trim()
} else {
  character(0)
}

# Prepare output
result <- tribble(
  ~field,          ~value,
  "dataset_name",  dataset_name,
  "condition",     condition,
  "by_vars",       paste(by_vars, collapse = ",")
)

# Write result
write_csv(result, file.path("outputs", "result.csv"))
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"field","value"
"class","flag_event|source|list"
"dataset_name","adlb"
"condition","parse_expr(vals[[""condition""]])"
"by_vars","USUBJID,PARAMCD"
```

### LLM Output
#### `result.csv`


```csv
field,value
dataset_name,adlb
condition,AVAL > 10
by_vars,"USUBJID,PARAMCD"
```
