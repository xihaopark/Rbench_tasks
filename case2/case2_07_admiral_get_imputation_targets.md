# Case2 07: pharmaverse/admiral/get_imputation_targets

## Metadata
- Task ID: `pharmaverse/admiral/get_imputation_targets`
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
Write an R script for an R package task: perform get imputation targets using the admiral package. The function(s) you use to solve the task must come from the admiral package.

Input: partial.tsv
Output: result.csv

Computation: For partial date/time targets, year-only dates use month 06 and day 15; missing time components use 00.


Required columns for result.csv: partial, year, month, day, hour, minute, second
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.

## Input preview

### partial.tsv
partial
2020
2020-02
2020-02-20T14
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

library(admiral)
library(dplyr)
library(readr)
library(stringr)

# Create outputs directory if it doesn't exist
if (!dir.exists("outputs")) {
  dir.create("outputs", recursive = TRUE)
}

# Read input
df <- read_tsv(
  file = file.path("inputs", "partial.tsv"),
  col_types = cols(partial = col_character())
)

# Map partial strings to CDISC-style imprecise ISO8601
df_cdisc <- df %>%
  mutate(
    partial_cdisc = case_when(
      str_detect(partial, "^\\d{4}$") ~ paste0(partial, "-XX-XXTXX:XX:XX"),
      str_detect(partial, "^\\d{4}-\\d{2}$") ~ paste0(partial, "-XXTXX:XX:XX"),
      str_detect(partial, "^\\d{4}-\\d{2}-\\d{2}T\\d{2}$") ~ paste0(partial, ":XX:XX"),
      TRUE ~ partial
    )
  )

# Derive imputation targets
df_imp <- df_cdisc %>%
  rowwise() %>%
  mutate(
    impute = list(
      derive_vars_dtm(
        dataset = tibble(ISO8601DATETIME = partial_cdisc),
        dtm = ISO8601DATETIME,
        highest_imputation = "h",
        date_imputation = "mid",
        time_imputation = "FIRST"
      )
    )
  ) %>%
  ungroup() %>%
  select(-partial_cdisc) %>%
  tidyr::unnest(impute)

# Extract components
result <- df_imp %>%
  mutate(
    year   = as.integer(substr(ISO8601DATETIME, 1, 4)),
    month  = as.integer(substr(ISO8601DATETIME, 6, 7)),
    day    = as.integer(substr(ISO8601DATETIME, 9, 10)),
    hour   = as.integer(substr(ISO8601DATETIME, 12, 13)),
    minute = as.integer(substr(ISO8601DATETIME, 15, 16)),
    second = as.integer(substr(ISO8601DATETIME, 18, 19))
  ) %>%
  select(partial, year, month, day, hour, minute, second)

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
No LLM output artifact was produced.
