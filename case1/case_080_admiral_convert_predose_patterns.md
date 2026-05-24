# Case 080: pharmaverse/admiral/convert_predose_patterns

## Metadata
- Task ID: `pharmaverse/admiral/convert_predose_patterns`
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
Write an R script to perform convert predose patterns using the admiral clinical task contract.

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
1
NA
```

#### `xxtpt.tsv`


```text
xxtpt
15 min predose
2 hr before
1 hour postdose
UNSCHEDULED
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

pattern <- regex("^(?<value>\\d+(?:\\.\\d+)?)\\s*(?<unit>m(?:in|inute)?|h(?:r|our)?)s?\\s+(?:pre-?dose|before)$", ignore_case = TRUE)
matches <- str_match(xxtpt, pattern)
idx <- !is.na(matches[, 1]) & is.na(result) & !na_idx
if (any(idx)) {
  value <- as.numeric(matches[idx, "value"])
  unit <- tolower(matches[idx, "unit"])
  result[idx] <- ifelse(substr(unit, 1, 1) == "m", -value / 60, -value)
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
})

# Create outputs directory if it doesn't exist
if (!dir.exists("outputs")) dir.create("outputs", recursive = TRUE)

# Read inputs
na_idx  <- read_tsv("inputs/na_idx.tsv", show_col_types = FALSE)
result  <- read_tsv("inputs/result.tsv", show_col_types = FALSE)
xxtpt   <- read_tsv("inputs/xxtpt.tsv", show_col_types = FALSE)

# Convert logical index if needed
na_logical <- na_idx[[1]]

# Apply NA index to result
result_vec <- result[[1]]
result_vec[na_logical] <- NA

# Prepare output
out <- tibble(
  xxtpt  = xxtpt[[1]],
  result = result_vec
)

# Write output
write_csv(out, "outputs/result.csv")
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"xxtpt","result"
"15 min predose",-0.25
"2 hr before",-2
"1 hour postdose",1
"UNSCHEDULED",NA
```

### LLM Output
#### `result.csv`


```csv
xxtpt,result
15 min predose,NA
2 hr before,NA
1 hour postdose,1
UNSCHEDULED,NA
```
