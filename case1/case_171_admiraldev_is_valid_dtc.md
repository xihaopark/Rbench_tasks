# Case 171: pharmaverse/admiraldev/is_valid_dtc

## Metadata
- Task ID: `pharmaverse/admiraldev/is_valid_dtc`
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
Write an R script to perform is valid dtc using the admiraldev clinical task contract.

Input: arg.tsv
Output: result.csv


Required columns for result.csv: arg, result
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `arg.tsv`


```text
arg
2020
2020-05
2020-05-20
2020-05-20T12:30:45
20-MAY-2020
```

## Code
### Ground Truth Code

```r
arg <- as.character(read.delim(file.path("inputs", "arg.tsv"), check.names = FALSE, stringsAsFactors = FALSE)$arg)
pattern <- "^[0-9]{4}(-[0-9]{2}(-[0-9]{2}(T[0-9]{2}(:[0-9]{2}(:[0-9]{2})?)?)?)?)?$"
result_df <- data.frame(arg = arg, result = grepl(pattern, arg), stringsAsFactors = FALSE)
dir.create("outputs", showWarnings = FALSE)
write.csv(result_df, file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

suppressPackageStartupMessages({
  library(readr)
  library(dplyr)
  library(admiraldev)
})

# Input / output paths
in_file  <- file.path("inputs", "arg.tsv")
out_dir  <- "outputs"
out_file <- file.path(out_dir, "result.csv")

if (!dir.exists(out_dir)) {
  dir.create(out_dir, recursive = TRUE)
}

# Read input
arg_df <- read_tsv(in_file, show_col_types = FALSE)

# Validate DTC using admiraldev clinical task contract
res_df <- arg_df %>%
  mutate(
    result = purrr::map_chr(
      arg,
      ~ {
        val <- tryCatch(
          {
            admiraldev::is_valid_dtc(.x)
          },
          error = function(e) NA
        )
        if (is.na(val)) {
          "ERROR"
        } else if (isTRUE(val)) {
          "TRUE"
        } else {
          "FALSE"
        }
      }
    )
  ) %>%
  select(arg, result)

# Write output
write_csv(res_df, out_file)
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"arg","result"
"2020",TRUE
"2020-05",TRUE
"2020-05-20",TRUE
"2020-05-20T12:30:45",TRUE
"20-MAY-2020",FALSE
```

### LLM Output
#### `result.csv`


```csv
arg,result
2020,TRUE
2020-05,TRUE
2020-05-20,TRUE
2020-05-20T12:30:45,TRUE
20-MAY-2020,FALSE
```
