# Case 085: pharmaverse/admiral/convert_start_patterns

## Metadata
- Task ID: `pharmaverse/admiral/convert_start_patterns`
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
Write an R script to perform convert start patterns using the admiral clinical task contract.

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
FALSE
```

#### `result.tsv`


```text
result
NA
NA
NA
4
```

#### `xxtpt.tsv`


```text
xxtpt
2h prior start of infusion
1h after start of treatment
3h post start of infusion
random label
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

pattern <- regex("^(?<value>\\d+(?:\\.\\d+)?)h\\s+(?<direction>prior|before|post|after)\\s+start\\s+of\\s+(?:infusion|treatment)$", ignore_case = TRUE)
matches <- str_match(xxtpt, pattern)
idx <- !is.na(matches[, 1]) & is.na(result) & !na_idx
if (any(idx)) {
  value <- as.numeric(matches[idx, "value"])
  direction <- tolower(matches[idx, "direction"])
  result[idx] <- ifelse(direction %in% c("prior", "before"), -value, value)
}

dir.create("outputs", showWarnings = FALSE)
write.csv(data.frame(xxtpt = xxtpt, result = result), file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

library(readr)
library(dplyr)
library(stringr)
library(fs)

# Create outputs directory if it does not exist
if (!dir_exists("outputs")) {
  dir_create("outputs")
}

# Read inputs
na_idx   <- read_tsv("inputs/na_idx.tsv", show_col_types = FALSE)
result   <- read_tsv("inputs/result.tsv", show_col_types = FALSE)
xxtpt    <- read_tsv("inputs/xxtpt.tsv", show_col_types = FALSE)

# Ensure logical na_idx
na_idx <- na_idx %>%
  mutate(na_idx = as.logical(na_idx))

# Apply admiral-like clinical task contract:
# - For rows where na_idx is TRUE, result stays NA
# - For rows where na_idx is FALSE, convert start patterns in xxtpt into numeric
#   keeping any pre-existing non-NA result values
#
# Pattern examples to convert:
#   "2h prior start of infusion"  -> -2
#   "1h after start of treatment" ->  1
#   "3h post start of infusion"   ->  3
#
# Any text containing:
#   "prior start" => negative sign
#   "after start" or "post start" => positive sign
# Extract leading hours before 'h'.

derive_time_from_xxtpt <- function(text) {
  if (is.na(text)) return(NA_real_)
  hrs <- str_match(text, "^(\\d+)h")[, 2]
  if (is.na(hrs)) return(NA_real_)
  hrs_num <- as.numeric(hrs)
  if (str_detect(text, "prior start")) {
    return(-hrs_num)
  } else if (str_detect(text, "after start|post start")) {
    return(hrs_num)
  } else {
    return(NA_real_)
  }
}

converted <- tibble(
  xxtpt = xxtpt$xxtpt,
  na_idx = na_idx$na_idx,
  result = result$result
) %>%
  mutate(
    conv = vapply(xxtpt, derive_time_from_xxtpt, numeric(1)),
    result = case_when(
      na_idx ~ NA_real_,
      !is.na(result) ~ result,
      TRUE ~ conv
    )
  ) %>%
  select(xxtpt, result)

# Write output
write_csv(converted, "outputs/result.csv", na = "")
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"xxtpt","result"
"2h prior start of infusion",-2
"1h after start of treatment",1
"3h post start of infusion",3
"random label",4
```

### LLM Output
#### `result.csv`


```csv
xxtpt,result
2h prior start of infusion,-2
1h after start of treatment,1
3h post start of infusion,3
random label,4
```
