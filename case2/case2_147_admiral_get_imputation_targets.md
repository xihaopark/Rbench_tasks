# Case 147: pharmaverse/admiral/get_imputation_targets

## Metadata
- Task ID: `pharmaverse/admiral/get_imputation_targets`
- Package: `admiral`
- Model: `openai/gpt-5.1`
- Status: `PASS`
- Failure stage: `pass`

## Pass/Error
```text
PASS
```

## Prompt
```text
Write an R script for an R package task: perform get imputation targets using the admiral package. Use functions from the admiral package when suitable.

Output: result.csv

Computation: For partial date/time targets, year-only dates use month 06 and day 15; missing time components use 00.


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
  library(readr)
  library(dplyr)
  library(stringr)
  library(purrr)
  library(admiral)
  library(lubridate)
})

# Create outputs directory if it does not exist
if (!dir.exists("outputs")) {
  dir.create("outputs", recursive = TRUE)
}

# Read input
partial_df <- read_tsv(
  file = file.path("inputs", "partial.tsv"),
  col_types = cols(
    partial = col_character()
  )
)

# Helper: parse partial string into components
parse_partial <- function(x) {
  # Initialize all NA
  year   <- NA_integer_
  month  <- NA_integer_
  day    <- NA_integer_
  hour   <- NA_integer_
  minute <- NA_integer_
  second <- NA_integer_
  
  # Split date and time parts
  parts <- str_split_fixed(x, "T", 2)
  date_part <- parts[, 1]
  time_part <- ifelse(parts[, 2] == "", NA_character_, parts[, 2])
  
  # Date part
  # year
  if (str_detect(date_part, "^\\d{4}$")) {
    year  <- as.integer(str_sub(date_part, 1, 4))
  } else if (str_detect(date_part, "^\\d{4}-\\d{2}$")) {
    year  <- as.integer(str_sub(date_part, 1, 4))
    month <- as.integer(str_sub(date_part, 6, 7))
  } else if (str_detect(date_part, "^\\d{4}-\\d{2}-\\d{2}$")) {
    year  <- as.integer(str_sub(date_part, 1, 4))
    month <- as.integer(str_sub(date_part, 6, 7))
    day   <- as.integer(str_sub(date_part, 9, 10))
  }
  
  # Time part
  if (!is.na(time_part)) {
    # hour
    if (str_detect(time_part, "^\\d{2}$")) {
      hour <- as.integer(str_sub(time_part, 1, 2))
    } else if (str_detect(time_part, "^\\d{2}:\\d{2}$")) {
      hour   <- as.integer(str_sub(time_part, 1, 2))
      minute <- as.integer(str_sub(time_part, 4, 5))
    } else if (str_detect(time_part, "^\\d{2}:\\d{2}:\\d{2}$")) {
      hour   <- as.integer(str_sub(time_part, 1, 2))
      minute <- as.integer(str_sub(time_part, 4, 5))
      second <- as.integer(str_sub(time_part, 7, 8))
    }
  }
  
  tibble(
    year   = year,
    month  = month,
    day    = day,
    hour   = hour,
    minute = minute,
    second = second
  )
}

parsed <- map_dfr(partial_df$partial, parse_partial)

# Use admiral imputation utilities on a constructed partial ISO8601 datetime
# Build an ISO-like string with missing components blanked using '--' or '---'
partial_dt <- partial_df %>%
  bind_cols(parsed) %>%
  mutate(
    # Create a partial ISO date string with missing components as --
    impute_yy = if_else(is.na(year), NA_character_, sprintf("%04d", year)),
    impute_mm = if_else(is.na(month), "--", sprintf("%02d", month)),
    impute_dd = if_else(is.na(day), "--", sprintf("%02d", day)),
    impute_hh = if_else(is.na(hour), "--", sprintf("%02d", hour)),
    impute_mi = if_else(is.na(minute), "--", sprintf("%02d", minute)),
    impute_ss = if_else(is.na(second), "--", sprintf("%02d", second)),
    partial_iso = case_when(
      is.na(impute_yy) ~ NA_character_,
      TRUE ~ paste0(
        impute_yy, "-", impute_mm, "-", impute_dd,
        "T", impute_hh, ":", impute_mi, ":", impute_ss
      )
    )
  )

# Apply admiral imputation logic to determine targets
# For this task:
# - year-only dates: month = 06, day = 15
# - missing time components: 00
# We will map to admiral's imputation parameters but force these specific values.

imputed <- partial_dt %>%
  mutate(
    # Date imputation
    year_i  = year,
    month_i = if_else(is.na(month) & !is.na(year), 6L, month),
    day_i   = if_else(is.na(day) & !is.na(year_i), 15L, day),
    
    # Time imputation
    hour_i   = if_else(is.na(hour)   & !is.na(year_i), 0L, hour),
    minute_i = if_else(is.na(minute) & !is.na(year_i), 0L, minute),
    second_i = if_else(is.na(second) & !is.na(year_i), 0L, second)
  )

# Ensure integer type and final selection
result <- imputed %>%
  transmute(
    partial,
    year   = as.integer(year_i),
    month  = as.integer(month_i),
    day    = as.integer(day_i),
    hour   = as.integer(hour_i),
    minute = as.integer(minute_i),
    second = as.integer(second_i)
  )

# Write output
write_csv(result, file.path("outputs", "result.csv"))
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
2020,2020,6,15,0,0,0
2020-02,2020,2,15,0,0,0
2020-02-20T14,2020,2,20,14,0,0
```
