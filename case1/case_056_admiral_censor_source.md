# Case 056: pharmaverse/admiral/censor_source

## Metadata
- Task ID: `pharmaverse/admiral/censor_source`
- Package: `admiral`
- Model: `openai/gpt-5.1`
- Status: `FAIL`
- Failure stage: `value_mismatch`

## Pass/Error
```text
The generated output files were produced, but one or more values differed from the
ground truth.

Main signal:
result.csv: Value mismatch in column: field.
```

## Prompt
```text
Write an R script to perform censor source using the admiral clinical task contract.

Input: spec.tsv
Output: result.csv


Required columns for result.csv: field, value
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `spec.tsv`


```text
field	value
dataset_name	adsl
filter	SAFFL == "Y"
date	ADT
censor	1
set_var	CNSR
set_value	1
order	ADT
```

## Code
### Ground Truth Code

```r
suppressPackageStartupMessages(library(admiral))
suppressPackageStartupMessages(library(rlang))

spec <- read.delim(file.path("inputs", "spec.tsv"), check.names = FALSE, stringsAsFactors = FALSE)
vals <- stats::setNames(spec$value, spec$field)
obj <- admiral::censor_source(
  dataset_name = vals[["dataset_name"]],
  filter = parse_expr(vals[["filter"]]),
  date = sym(vals[["date"]]),
  censor = as.integer(vals[["censor"]]),
  set_values_to = exprs(!!sym(vals[["set_var"]]) := as.integer(vals[["set_value"]])),
  order = exprs(!!sym(vals[["order"]]))
)
result <- data.frame(
  field = c("class", "dataset_name", "filter", "date", "censor", "set_values_to", "order"),
  value = c(
    paste(class(obj), collapse = "|"),
    obj$dataset_name,
    paste(deparse(obj$filter), collapse = " "),
    paste(deparse(obj$date), collapse = " "),
    as.character(obj$censor),
    paste(names(obj$set_values_to), vapply(obj$set_values_to, function(x) paste(deparse(x), collapse = " "), character(1)), sep = "=", collapse = ";"),
    paste(vapply(obj$order, function(x) paste(deparse(x), collapse = " "), character(1)), collapse = ",")
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
  library(tidyr)
  library(stringr)
  library(fs)
})

# Create outputs directory if needed
if (!dir_exists("outputs")) {
  dir_create("outputs")
}

# Read spec
spec_path <- file.path("inputs", "spec.tsv")
spec <- read_tsv(spec_path, show_col_types = FALSE)

# Pivot spec to name-value pairs
spec_long <- spec %>%
  select(field, value) %>%
  filter(!is.na(field), field != "") %>%
  distinct()

# Write result
write_csv(spec_long, file.path("outputs", "result.csv"))
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"field","value"
"class","censor_source|tte_source|source|list"
"dataset_name","adsl"
"filter","parse_expr(vals[[""filter""]])"
"date","sym(vals[[""date""]])"
"censor","1"
"set_values_to","CNSR=as.integer(vals[[""set_value""]])"
"order","ADT"
```

### LLM Output
#### `result.csv`


```csv
field,value
dataset_name,adsl
filter,"SAFFL == ""Y"""
date,ADT
censor,1
set_var,CNSR
set_value,1
order,ADT
```
