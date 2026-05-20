# Double-quote variable names: `pharmaverse/admiraldev/dquote`

## metadata

- Task: `pharmaverse/admiraldev/dquote`
- Package: `admiraldev`
- Model: `openai/gpt-5.1`
- Status: `PASS`
- Failure stage: `pass`
- Score: `1.0`

## pass/error

```text
PASS
```

## prompt

```text
Write an R script for an R package task: perform dquote using the admiraldev package. Use tools and functions from the admiraldev package to solve the task.

Input: x.tsv
Output: result.csv


Required columns for result.csv: x, result
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## input

### `x.tsv`

```text
x
USUBJID
AVAL
TRT01A
```

## code

### ground truth code

```r
x <- as.character(read.delim(file.path("inputs", "x.tsv"), check.names = FALSE, stringsAsFactors = FALSE)$x)
result_df <- data.frame(x = x, result = paste0('"', x, '"'), stringsAsFactors = FALSE)
dir.create("outputs", showWarnings = FALSE)
write.csv(result_df, file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM code

```r
#!/usr/bin/env Rscript

library(dplyr)
library(readr)
library(admiraldev)

# Create outputs directory if it doesn't exist
if (!dir.exists("outputs")) dir.create("outputs", recursive = TRUE)

# Read input file
x_df <- read_tsv(
  file = file.path("inputs", "x.tsv"),
  col_types = cols(.default = col_character())
)

# Perform dquote using admiraldev clinical task contract
result_df <- x_df %>%
  mutate(result = admiraldev::dquote(x)) %>%
  select(x, result)

# Write output file
write_csv(result_df, file.path("outputs", "result.csv"), na = "")
```

## output

### ground truth output

#### `result.csv`

```csv
"x","result"
"USUBJID","""USUBJID"""
"AVAL","""AVAL"""
"TRT01A","""TRT01A"""
```

### LLM output

#### `result.csv`

```csv
x,result
USUBJID,"""USUBJID"""
AVAL,"""AVAL"""
TRT01A,"""TRT01A"""
```
