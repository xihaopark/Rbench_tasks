# Case 081: pharmaverse/admiral/convert_ranges

## Metadata
- Task ID: `pharmaverse/admiral/convert_ranges`
- Package: `admiral`
- Model: `openai/gpt-5.1`
- Status: `FAIL`
- Failure stage: `value_mismatch`

## Pass/Error
```text
The generated output files were produced, but one or more values differed from the
ground truth.

Main signal:
result.csv: Numeric mismatch in column: result.
```

## Prompt
```text
Write an R script to perform convert ranges using the admiral clinical task contract.

Input: na_idx.tsv, range_method.tsv, result.tsv, xxtpt.tsv
Output: result.csv


Required columns for result.csv: xxtpt, result
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `na_idx.tsv`


```text
na_idx
FALSE
FALSE
FALSE
FALSE
```

#### `range_method.tsv`


```text
range_method
midpoint
```

#### `result.tsv`


```text
result
NA
NA
NA
5
```

#### `xxtpt.tsv`


```text
xxtpt
1-3 h prior start
2-4 hours after start
6-8 hr post dose
unmatched
```

## Code
### Ground Truth Code

```r
suppressPackageStartupMessages(library(stringr))

xxtpt <- read.delim(file.path("inputs", "xxtpt.tsv"), check.names = FALSE, stringsAsFactors = FALSE)$xxtpt
result <- read.delim(file.path("inputs", "result.tsv"), check.names = FALSE, stringsAsFactors = FALSE)$result
result <- suppressWarnings(as.numeric(result))
na_idx <- read.delim(file.path("inputs", "na_idx.tsv"), check.names = FALSE, stringsAsFactors = FALSE)$na_idx
na_idx <- as.logical(na_idx)
range_method <- read.delim(file.path("inputs", "range_method.tsv"), check.names = FALSE, stringsAsFactors = FALSE)[[1]][1]

range_value <- function(start, end, method) {
  if (tolower(method) %in% c("midpoint", "mean", "average")) (start + end) / 2 else end
}
pattern <- regex("^(?<start>\\d+(?:\\.\\d+)?)\\s*-\\s*(?<end>\\d+(?:\\.\\d+)?)\\s*h(?:r|our)?s?(?:\\s+(?<direction>prior|before|post|after)(?:\\s+(?:start|end|dose))?)?$", ignore_case = TRUE)
matches <- str_match(xxtpt, pattern)
idx <- !is.na(matches[, 1]) & is.na(result) & !na_idx
if (any(idx)) {
  value <- range_value(as.numeric(matches[idx, "start"]), as.numeric(matches[idx, "end"]), range_method)
  direction <- tolower(matches[idx, "direction"])
  direction[is.na(direction)] <- "after"
  result[idx] <- ifelse(direction %in% c("prior", "before"), -value, value)
}

dir.create("outputs", showWarnings = FALSE)
write.csv(data.frame(xxtpt = xxtpt, result = result), file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

suppressPackageStartupMessages({
  library(readr)
  library(dplyr)
  library(stringr)
})

# Create outputs directory if it doesn't exist
if (!dir.exists("outputs")) dir.create("outputs", recursive = TRUE)

# Read inputs
na_idx <- read_tsv(
  file = file.path("inputs", "na_idx.tsv"),
  show_col_types = FALSE
) %>%
  mutate(row = row_number())

range_method <- read_tsv(
  file = file.path("inputs", "range_method.tsv"),
  show_col_types = FALSE
)

result <- read_tsv(
  file = file.path("inputs", "result.tsv"),
  show_col_types = FALSE
) %>%
  mutate(row = row_number())

xxtpt <- read_tsv(
  file = file.path("inputs", "xxtpt.tsv"),
  show_col_types = FALSE
) %>%
  mutate(row = row_number())

# Helper: get midpoint from a label containing a numeric range
get_midpoint <- function(label) {
  # Extract all numbers (integer or decimal, including possible leading minus)
  nums <- str_extract_all(label, "-?\\d*\\.?\\d+")[[1]]
  # Keep only first two numeric values as range bounds
  if (length(nums) < 2) return(NA_real_)
  vals <- as.numeric(nums[1:2])
  mean(vals)
}

# Apply range conversion only where:
# - result is NA
# - na_idx is FALSE
# - range_method is "midpoint"
res <- result %>%
  left_join(na_idx, by = "row", suffix = c("", "_naidx")) %>%
  left_join(xxtpt, by = "row", suffix = c("", "_xxtpt")) %>%
  mutate(
    result = case_when(
      is.na(result) &
        !na_idx &
        range_method$range_method[1] == "midpoint" ~
          vapply(xxtpt, get_midpoint, numeric(1)),
      TRUE ~ result
    )
  ) %>%
  select(xxtpt, result)

# Write output
write_csv(res, file.path("outputs", "result.csv"), na = "")
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"xxtpt","result"
"1-3 h prior start",-2
"2-4 hours after start",3
"6-8 hr post dose",7
"unmatched",5
```

### LLM Output
#### `result.csv`


```csv
xxtpt,result
1-3 h prior start,-1
2-4 hours after start,-1
6-8 hr post dose,-1
unmatched,5
```
