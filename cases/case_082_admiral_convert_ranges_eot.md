# Case 082: pharmaverse/admiral/convert_ranges_eot

## Metadata
- Task ID: `pharmaverse/admiral/convert_ranges_eot`
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
Write an R script to perform convert ranges eot using the admiral clinical task contract.

Input: na_idx.tsv, range_method.tsv, result.tsv, treatment_duration.tsv, xxtpt.tsv
Output: result.csv


Required columns for result.csv: xxtpt, treatment_duration, result
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
9
```

#### `treatment_duration.tsv`


```text
treatment_duration
10
10
20
9
```

#### `xxtpt.tsv`


```text
xxtpt
1-3 h post EOI
2-4 hours after end of infusion
3-5 hr after end of treatment
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
treatment_duration <- read.delim(file.path("inputs", "treatment_duration.tsv"), check.names = FALSE, stringsAsFactors = FALSE)$treatment_duration
treatment_duration <- as.numeric(treatment_duration)

range_value <- function(start, end, method) {
  if (tolower(method) %in% c("midpoint", "mean", "average")) (start + end) / 2 else end
}
pattern <- regex("^(?<start>\\d+(?:\\.\\d+)?)\\s*-\\s*(?<end>\\d+(?:\\.\\d+)?)\\s*h(?:r|our)?s?\\s+(?:(?:post|after)\\s*-?\\s*)?(?:eo[it]|end\\s+of\\s+(?:infusion|treatment)|(?:post|after)\\s*-?\\s*inf(?:usion)?)$", ignore_case = TRUE)
matches <- str_match(xxtpt, pattern)
idx <- !is.na(matches[, 1]) & is.na(result) & !na_idx
if (any(idx)) {
  value <- range_value(as.numeric(matches[idx, "start"]), as.numeric(matches[idx, "end"]), range_method)
  result[idx] <- treatment_duration[idx] + value
}

dir.create("outputs", showWarnings = FALSE)
write.csv(data.frame(xxtpt = xxtpt, treatment_duration = treatment_duration, result = result), file.path("outputs", "result.csv"), row.names = FALSE)
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
na_idx <- read_tsv("inputs/na_idx.tsv", show_col_types = FALSE)
range_method <- read_tsv("inputs/range_method.tsv", show_col_types = FALSE)
result <- read_tsv("inputs/result.tsv", show_col_types = FALSE)
treatment_duration <- read_tsv("inputs/treatment_duration.tsv", show_col_types = FALSE)
xxtpt <- read_tsv("inputs/xxtpt.tsv", show_col_types = FALSE)

# Helper to extract numeric ranges from xxtpt text (in hours)
extract_range <- function(txt) {
  # replace possible time unit words around numbers
  x <- tolower(txt)
  # capture first two numeric values
  nums <- str_extract_all(x, "\\d+\\.*\\d*")[[1]]
  if (length(nums) >= 2) {
    as.numeric(nums[1:2])
  } else if (length(nums) == 1) {
    c(as.numeric(nums[1]), NA_real_)
  } else {
    c(NA_real_, NA_real_)
  }
}

# Apply range extraction
ranges <- lapply(xxtpt$xxtpt, extract_range)
lower <- sapply(ranges, `[[`, 1)
upper <- sapply(ranges, `[[`, 2)

df <- tibble(
  xxtpt = xxtpt$xxtpt,
  treatment_duration = treatment_duration$treatment_duration,
  result = result$result,
  na_idx = na_idx$na_idx,
  lower = lower,
  upper = upper
)

# Only fill where result is NA and na_idx is FALSE
method <- range_method$range_method[1]

df <- df %>%
  mutate(
    calc = case_when(
      is.na(result) & !na_idx & !is.na(lower) & !is.na(upper) & method == "midpoint" ~
        (lower + upper) / 2,
      TRUE ~ as.numeric(result)
    ),
    result = calc
  ) %>%
  select(xxtpt, treatment_duration, result)

# Write output
write_csv(df, "outputs/result.csv", na = "")
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"xxtpt","treatment_duration","result"
"1-3 h post EOI",10,12
"2-4 hours after end of infusion",10,13
"3-5 hr after end of treatment",20,24
"unmatched",9,9
```

### LLM Output
#### `result.csv`


```csv
xxtpt,treatment_duration,result
1-3 h post EOI,10,2
2-4 hours after end of infusion,10,3
3-5 hr after end of treatment,20,4
unmatched,9,9
```
