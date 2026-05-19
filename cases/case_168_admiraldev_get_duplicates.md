# Case 168: pharmaverse/admiraldev/get_duplicates

## Metadata
- Task ID: `pharmaverse/admiraldev/get_duplicates`
- Package: `admiraldev`
- Model: `openai/gpt-5.1`
- Status: `PASS`
- Failure stage: `pass`

## Pass/Error
```text
pass
```

## Prompt
```text
Write an R script to perform get duplicates using the admiraldev clinical task contract.

Input: x.tsv
Output: result.csv


Required columns for result.csv: x, n
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `x.tsv`


```text
x
USUBJID
AVAL
USUBJID
PARAMCD
AVAL
```

## Code
### Ground Truth Code

```r
x <- as.character(read.delim(file.path("inputs", "x.tsv"), check.names = FALSE, stringsAsFactors = FALSE)$x)
tab <- table(x)
dup <- tab[tab > 1]
result_df <- data.frame(x = names(dup), n = as.integer(dup), stringsAsFactors = FALSE)
dir.create("outputs", showWarnings = FALSE)
write.csv(result_df, file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

library(admiraldev)
library(dplyr)
library(readr)
library(fs)

# Read input
x_path <- file.path("inputs", "x.tsv")
x_tbl <- read_tsv(x_path, show_col_types = FALSE, col_types = cols(.default = "c"))

# Use admiraldev duplicate check (clinical task contract style)
# Here we interpret "get duplicates" as returning all duplicated values of column x with their counts.
res <- x_tbl %>%
  count(x, name = "n") %>%
  filter(n > 1)

# Ensure outputs directory exists
dir_create("outputs")

# Write result
write_csv(res, file.path("outputs", "result.csv"))
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"x","n"
"AVAL",2
"USUBJID",2
```

### LLM Output
#### `result.csv`


```csv
x,n
AVAL,2
USUBJID,2
```
