# Case 147: pharmaverse/admiral/get_imputation_targets

## Metadata
- Task ID: `pharmaverse/admiral/get_imputation_targets`
- Package: `admiral`
- Model: `openai/gpt-5.1`
- Status: `FAIL`
- Failure stage: `value_mismatch`

## Pass/Error
```text
The generated output files were produced, but one or more values differed from the
ground truth.

Main signal:
result.csv: Numeric missingness mismatch in column: month.
```

## Prompt
```text
Write an R script to perform get imputation targets using the admiral clinical task contract.

Input: partial.tsv
Output: result.csv


Required columns for result.csv: partial, year, month, day, hour, minute, second
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `partial.tsv`


```text
partial
2020
2020-02
2020-02-20T14
```

## Code
### Ground Truth Code

```r
partial <- as.character(read.delim(file.path("inputs", "partial.tsv"), check.names = FALSE, stringsAsFactors = FALSE)$partial)
year <- substr(partial, 1, 4)
month <- ifelse(nchar(partial) >= 7, substr(partial, 6, 7), "06")
day <- ifelse(nchar(partial) >= 10, substr(partial, 9, 10), "15")
hour <- ifelse(nchar(partial) >= 13, substr(partial, 12, 13), "00")
minute <- ifelse(nchar(partial) >= 16, substr(partial, 15, 16), "00")
second <- ifelse(nchar(partial) >= 19, substr(partial, 18, 19), "00")
result_df <- data.frame(partial = partial, year = year, month = month, day = day, hour = hour, minute = minute, second = second, stringsAsFactors = FALSE)

dir.create("outputs", showWarnings = FALSE)
write.csv(result_df, file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

suppressPackageStartupMessages({
  library(dplyr)
  library(readr)
  library(stringr)
  library(lubridate)
})

# Ensure outputs directory exists
if (!dir.exists("outputs")) {
  dir.create("outputs", recursive = TRUE)
}

# Read input
partial_df <- readr::read_tsv(
  file = file.path("inputs", "partial.tsv"),
  col_types = cols(
    partial = col_character()
  )
)

# Helper to parse partial datetime strings
parse_partial <- function(x) {
  # Initialize all as NA
  year   <- rep(NA_integer_, length(x))
  month  <- rep(NA_integer_, length(x))
  day    <- rep(NA_integer_, length(x))
  hour   <- rep(NA_integer_, length(x))
  minute <- rep(NA_integer_, length(x))
  second <- rep(NA_integer_, length(x))

  # Detect patterns using regex
  # 1) Year only: YYYY
  idx_year <- str_detect(x, "^\\d{4}$")
  year[idx_year] <- as.integer(x[idx_year])

  # 2) Year-month: YYYY-MM
  idx_ym <- str_detect(x, "^\\d{4}-\\d{2}$")
  year[idx_ym]  <- as.integer(str_sub(x[idx_ym], 1, 4))
  month[idx_ym] <- as.integer(str_sub(x[idx_ym], 6, 7))

  # 3) Year-month-dayThour: YYYY-MM-DDThh
  idx_ymdh <- str_detect(x, "^\\d{4}-\\d{2}-\\d{2}T\\d{2}$")
  year[idx_ymdh]  <- as.integer(str_sub(x[idx_ymdh], 1, 4))
  month[idx_ymdh] <- as.integer(str_sub(x[idx_ymdh], 6, 7))
  day[idx_ymdh]   <- as.integer(str_sub(x[idx_ymdh], 9, 10))
  hour[idx_ymdh]  <- as.integer(str_sub(x[idx_ymdh], 12, 13))

  tibble(
    year   = year,
    month  = month,
    day    = day,
    hour   = hour,
    minute = minute,
    second = second
  )
}

result <- partial_df %>%
  mutate(
    parse_partial(partial)
  ) %>%
  select(partial, year, month, day, hour, minute, second)

# Write output
readr::write_csv(result, file.path("outputs", "result.csv"), na = "")
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"partial","year","month","day","hour","minute","second"
"2020","2020","06","15","00","00","00"
"2020-02","2020","02","15","00","00","00"
"2020-02-20T14","2020","02","20","14","00","00"
```

### LLM Output
#### `result.csv`


```csv
partial,year,month,day,hour,minute,second
2020,2020,,,,,
2020-02,2020,2,,,,
2020-02-20T14,2020,2,20,14,,
```
