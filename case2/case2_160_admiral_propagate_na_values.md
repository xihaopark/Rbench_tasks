# Case 160: pharmaverse/admiral/propagate_na_values

## Metadata
- Task ID: `pharmaverse/admiral/propagate_na_values`
- Package: `admiral`
- Model: `openai/gpt-5.1`
- Status: `NO_OUTPUT`
- Failure stage: `execution_failure`

## Pass/Error
```text
The generated code failed while running before valid outputs could be evaluated.

Main signal:
missing artifacts: result.csv.
```

## Prompt
```text
Write an R script for an R package task: perform propagate na values using the admiral package. Use functions from the admiral package when suitable.

Output: result.csv

Computation: For ordered datetime components, keep components before the first missing component and blank all later components.


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

library(admiral)
library(readr)
library(dplyr)

# Create outputs directory if it doesn't exist
if (!dir.exists("outputs")) {
  dir.create("outputs", recursive = TRUE)
}

# Read input
df <- read_tsv(
  file = file.path("inputs", "partial.tsv"),
  col_types = cols(
    year = col_character(),
    month = col_character(),
    day = col_character(),
    hour = col_character(),
    minute = col_character(),
    second = col_character()
  )
)

# Propagate missing date-time components using admiral
df_res <- df %>%
  derive_vars_dt(
    dtc = paste0(
      year, "-",
      month, "-",
      day, "T",
      hour, ":",
      minute, ":",
      second
    ),
    flag_imputation = FALSE
  ) %>%
  mutate(
    year   = if_else(is.na(year)   | is.na(dtc), NA_character_, year),
    month  = if_else(is.na(month)  | is.na(dtc), NA_character_, month),
    day    = if_else(is.na(day)    | is.na(dtc), NA_character_, day),
    hour   = if_else(is.na(hour)   | is.na(dtc), NA_character_, hour),
    minute = if_else(is.na(minute) | is.na(dtc), NA_character_, minute),
    second = if_else(is.na(second) | is.na(dtc), NA_character_, second)
  ) %>%
  select(year, month, day, hour, minute, second)

# Write output
write_csv(df_res, file.path("outputs", "result.csv"), na = "")
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
No LLM output artifact was produced.
