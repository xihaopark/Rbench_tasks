# Case 054: pharmaverse/admiral/calculate_range_value

## Metadata
- Task ID: `pharmaverse/admiral/calculate_range_value`
- Package: `admiral`
- Model: `openai/gpt-5.1`
- Status: `PASS`
- Failure stage: `pass`

## Pass/Error
```text
pass
```

## Prompt
```text
Write an R script to perform calculate range value using the admiral clinical task contract.

Input: end_val.tsv, range_method.tsv, start_val.tsv
Output: result.csv


Required columns for result.csv: start_val, end_val, range_method, result
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `end_val.tsv`


```text
end_val
10
6
8
```

#### `range_method.tsv`


```text
range_method
midpoint
```

#### `start_val.tsv`


```text
start_val
0
2
4
```

## Code
### Ground Truth Code

```r
suppressPackageStartupMessages(library(admiral))

start_val <- as.numeric(read.delim(file.path("inputs", "start_val.tsv"), check.names = FALSE, stringsAsFactors = FALSE)[[1]])
end_val <- as.numeric(read.delim(file.path("inputs", "end_val.tsv"), check.names = FALSE, stringsAsFactors = FALSE)[[1]])
range_method <- as.character(read.delim(file.path("inputs", "range_method.tsv"), check.names = FALSE, stringsAsFactors = FALSE)[[1]][1])
result <- switch(range_method, start = start_val, end = end_val, midpoint = (start_val + end_val) / 2, (start_val + end_val) / 2)

dir.create("outputs", showWarnings = FALSE)
write.csv(data.frame(start_val = start_val, end_val = end_val, range_method = range_method, result = result), file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

library(readr)
library(dplyr)
library(stringr)
library(tidyr)
library(purrr)
library(fs)

# Create outputs directory if it does not exist
if (!dir_exists("outputs")) {
  dir_create("outputs")
}

# Read inputs
end_val <- read_tsv("inputs/end_val.tsv", show_col_types = FALSE)
start_val <- read_tsv("inputs/start_val.tsv", show_col_types = FALSE)
range_method <- read_tsv("inputs/range_method.tsv", show_col_types = FALSE)

# Basic checks
if (!"end_val" %in% names(end_val)) stop("Missing end_val column")
if (!"start_val" %in% names(start_val)) stop("Missing start_val column")
if (!"range_method" %in% names(range_method)) stop("Missing range_method column")

# Expand inputs to same length (assume row-wise pairing)
n <- max(nrow(start_val), nrow(end_val))
if (nrow(start_val) != n) {
  start_val <- start_val %>%
    slice(rep(1:nrow(.), length.out = n))
}
if (nrow(end_val) != n) {
  end_val <- end_val %>%
    slice(rep(1:nrow(.), length.out = n))
}

# Use single method for all rows (per admiral-style parameterization)
method <- range_method$range_method[1]

res <- tibble(
  start_val = start_val$start_val,
  end_val   = end_val$end_val
) %>%
  mutate(
    range_method = method,
    result = case_when(
      str_to_lower(range_method) == "midpoint" ~ (start_val + end_val) / 2,
      TRUE ~ NA_real_
    )
  )

write_csv(res, "outputs/result.csv")
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"start_val","end_val","range_method","result"
0,10,"midpoint",5
2,6,"midpoint",4
4,8,"midpoint",6
```

### LLM Output
#### `result.csv`


```csv
start_val,end_val,range_method,result
0,10,midpoint,5
2,6,midpoint,4
4,8,midpoint,6
```
