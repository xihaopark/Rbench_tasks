# Case 076: pharmaverse/admiral/convert_min_after_start

## Metadata
- Task ID: `pharmaverse/admiral/convert_min_after_start`
- Package: `admiral`
- Model: `openai/gpt-5.1`
- Status: `FAIL`
- Failure stage: `value_mismatch`

## Pass/Error
```text
The generated output files were produced, but one or more values differed from the
ground truth.

Main signal:
result.csv: Numeric missingness mismatch in column: result.
```

## Prompt
```text
Write an R script to perform convert min after start using the admiral clinical task contract.

Input: na_idx.tsv, result.tsv, xxtpt.tsv
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
TRUE
```

#### `result.tsv`


```text
result
NA
NA
2
NA
```

#### `xxtpt.tsv`


```text
xxtpt
5 min after start inf
30 minutes after start inf
2 hours after start
not applicable
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

pattern <- regex("^(?<value>\\d+(?:\\.\\d+)?)\\s*m(?:in|inute)?s?\\s+after\\s+start\\s+inf$", ignore_case = TRUE)
matches <- str_match(xxtpt, pattern)
idx <- !is.na(matches[, 1]) & is.na(result) & !na_idx
result[idx] <- as.numeric(matches[idx, "value"]) / 60

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
na_idx  <- read_tsv("inputs/na_idx.tsv", show_col_types = FALSE)
result  <- read_tsv("inputs/result.tsv", show_col_types = FALSE)
xxtpt   <- read_tsv("inputs/xxtpt.tsv", show_col_types = FALSE)

# Ensure logical na_idx
na_idx <- na_idx %>%
  mutate(na_idx = as.logical(na_idx))

# Apply ADMIRAL clinical task contract:
# - For rows where na_idx is TRUE, derive result from xxtpt as minutes after start
# - Else keep existing result
derive_mins_after_start <- function(txt) {
  txt <- tolower(txt)

  # pattern components
  num <- "([0-9]+(?:\\.[0-9]+)?)"
  unit <- "(min|mins|minute|minutes|hour|hours|hr|hrs)"
  after_start_pattern <- paste0("^\\s*", num, "\\s*", unit, "s?\\s+after\\s+start")

  out <- rep(NA_real_, length(txt))

  is_match <- str_detect(txt, after_start_pattern)
  matched_txt <- txt[is_match]

  n <- as.numeric(str_match(matched_txt, num)[, 2])
  u <- str_match(matched_txt, unit)[, 2]

  factor_min <- ifelse(str_detect(u, "hour|hr"), 60, 1)
  out[is_match] <- n * factor_min

  out
}

derived_result <- derive_mins_after_start(xxtpt$xxtpt)

final_result <- result %>%
  mutate(
    result = if_else(na_idx$na_idx, derived_result, as.numeric(result))
  ) %>%
  select(result = result) %>%
  bind_cols(xxtpt %>% select(xxtpt)) %>%
  select(xxtpt, result)

# Write output
write_csv(final_result, "outputs/result.csv", na = "")
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"xxtpt","result"
"5 min after start inf",0.0833333333333333
"30 minutes after start inf",0.5
"2 hours after start",2
"not applicable",NA
```

### LLM Output
#### `result.csv`


```csv
xxtpt,result
5 min after start inf,
30 minutes after start inf,
2 hours after start,2
not applicable,
```
