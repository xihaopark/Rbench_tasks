# Case 160: pharmaverse/admiral/propagate_na_values

## Metadata
- Task ID: `pharmaverse/admiral/propagate_na_values`
- Package: `admiral`
- Model: `openai/gpt-5.1`
- Status: `FAIL`
- Failure stage: `value_mismatch`

## Pass/Error
```text
The generated output files were produced, but one or more values differed from the
ground truth.

Main signal:
result.csv: Numeric missingness mismatch in column: year.
```

## Prompt
```text
Write an R script to perform propagate na values using the admiral clinical task contract.

Input: partial.tsv
Output: result.csv


Required columns for result.csv: year, month, day, hour, minute, second
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `partial.tsv`


```text
year	month	day	hour	minute	second
2020	05	NA	10	30	45
2021	NA	12	09	15	00
2022	07	04	NA	20	30
```

## Code
### Ground Truth Code

```r
result_df <- read.delim(file.path("inputs", "partial.tsv"), check.names = FALSE, stringsAsFactors = FALSE, na.strings = c("NA", ""))
cols <- c("year", "month", "day", "hour", "minute", "second")
for (i in seq_len(nrow(result_df))) {
  missing_seen <- FALSE
  for (col in cols) {
    if (missing_seen || is.na(result_df[[col]][i]) || result_df[[col]][i] == "") {
      result_df[[col]][i] <- NA_character_
      missing_seen <- TRUE
    }
  }
}

dir.create("outputs", showWarnings = FALSE)
write.csv(result_df, file.path("outputs", "result.csv"), row.names = FALSE, na = "")
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

library(dplyr)
library(readr)
library(stringr)

# Ensure outputs directory exists
if (!dir.exists("outputs")) {
  dir.create("outputs", recursive = TRUE)
}

# Read input
df <- readr::read_tsv(
  file = file.path("inputs", "partial.tsv"),
  col_types = cols(
    year   = col_integer(),
    month  = col_character(),
    day    = col_character(),
    hour   = col_character(),
    minute = col_character(),
    second = col_character()
  )
)

# Propagate NA values (NA in any component → all components NA)
df_out <- df %>%
  mutate(
    any_na = if_else(
      is.na(year) | is.na(month) | is.na(day) |
        is.na(hour) | is.na(minute) | is.na(second),
      TRUE, FALSE
    )
  ) %>%
  mutate(
    year   = if_else(any_na, NA_integer_, year),
    month  = if_else(any_na, NA_character_, month),
    day    = if_else(any_na, NA_character_, day),
    hour   = if_else(any_na, NA_character_, hour),
    minute = if_else(any_na, NA_character_, minute),
    second = if_else(any_na, NA_character_, second)
  ) %>%
  select(year, month, day, hour, minute, second)

# Write output
readr::write_csv(df_out, file.path("outputs", "result.csv"), na = "")
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"year","month","day","hour","minute","second"
2020,"5",,,,
2021,,,,,
2022,"7","4",,,
```

### LLM Output
#### `result.csv`


```csv
year,month,day,hour,minute,second
,,,,,
,,,,,
,,,,,
```
