# Case 151: pharmaverse/admiral/get_unified_time_unit

## Metadata
- Task ID: `pharmaverse/admiral/get_unified_time_unit`
- Package: `admiral`
- Model: `openai/gpt-5.1`
- Status: `FAIL`
- Failure stage: `value_mismatch`

## Pass/Error
```text
The generated output files were produced, but one or more values differed from the
ground truth.

Main signal:
result.csv: Value mismatch in column: result.
```

## Prompt
```text
Write an R script to perform get unified time unit using the admiral clinical task contract.

Input: time_unit.tsv
Output: result.csv


Required columns for result.csv: time_unit, result
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `time_unit.tsv`


```text
time_unit
HOURS
hour
hr
DAYS
day
min
minutes
seconds
```

## Code
### Ground Truth Code

```r
time_unit <- as.character(read.delim(file.path("inputs", "time_unit.tsv"), check.names = FALSE, stringsAsFactors = FALSE)$time_unit)
key <- tolower(trimws(time_unit))
result <- ifelse(key %in% c("h", "hr", "hrs", "hour", "hours"), "hours",
  ifelse(key %in% c("d", "day", "days"), "days",
    ifelse(key %in% c("m", "min", "mins", "minute", "minutes"), "minutes",
      ifelse(key %in% c("s", "sec", "secs", "second", "seconds"), "seconds", NA_character_))))

dir.create("outputs", showWarnings = FALSE)
write.csv(data.frame(time_unit = time_unit, result = result), file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

suppressPackageStartupMessages({
  library(dplyr)
  library(readr)
  library(stringr)
})

# Create outputs directory if it doesn't exist
if (!dir.exists("outputs")) {
  dir.create("outputs", recursive = TRUE)
}

# Read input
time_unit_df <- readr::read_tsv(
  file = file.path("inputs", "time_unit.tsv"),
  show_col_types = FALSE
)

# Function to map to unified time unit (based on admiral conventions)
map_time_unit <- function(x) {
  x_std <- str_trim(toupper(x))

  case_when(
    x_std %in% c("H", "HR", "HOUR", "HOURS", "HRS") ~ "HOURS",
    x_std %in% c("D", "DAY", "DAYS") ~ "DAYS",
    x_std %in% c("W", "WK", "WEEK", "WEEKS") ~ "WEEKS",
    x_std %in% c("MON", "MONTH", "MONTHS", "MO") ~ "MONTHS",
    x_std %in% c("Y", "YR", "YEAR", "YEARS") ~ "YEARS",
    TRUE ~ NA_character_
  )
}

result_df <- time_unit_df %>%
  mutate(result = map_time_unit(time_unit)) %>%
  select(time_unit, result)

# Write output
readr::write_csv(result_df, file.path("outputs", "result.csv"), na = "")
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"time_unit","result"
"HOURS","hours"
"hour","hours"
"hr","hours"
"DAYS","days"
"day","days"
"min","minutes"
"minutes","minutes"
"seconds","seconds"
```

### LLM Output
#### `result.csv`


```csv
time_unit,result
HOURS,HOURS
hour,HOURS
hr,HOURS
DAYS,DAYS
day,DAYS
min,
minutes,
seconds,
```
