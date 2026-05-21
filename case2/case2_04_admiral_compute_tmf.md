# Case2 04: pharmaverse/admiral/compute_tmf

## Metadata
- Task ID: `pharmaverse/admiral/compute_tmf`
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
Write an R script for an R package task: perform compute tmf using the admiral package. The function(s) you use to solve the task must come from the admiral package.

Output: result.csv

Computation: For this fixture's partial-time flags, map hour-only imputation to M, minute-only imputation to S, and date-only records to H.


Required columns for result.csv: dtc, dtm, result
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `dtc.tsv`

```text
dtc
2020-01-01T12
2020-01-01T12:30
2020-01-01
```

#### `dtm.tsv`

```text
dtm
2020-01-01 12:00:00
2020-01-01 12:30:00
2020-01-01 00:00:00
```

#### `ignore_seconds_flag.tsv`

```text
ignore_seconds_flag
FALSE
```

## Code
### Ground Truth Code

```r
suppressPackageStartupMessages(library(admiral))

dtc <- as.character(read.delim(file.path("inputs", "dtc.tsv"), check.names = FALSE, stringsAsFactors = FALSE)[[1]])
dtm_chr <- as.character(read.delim(file.path("inputs", "dtm.tsv"), check.names = FALSE, stringsAsFactors = FALSE)[[1]])
ignore_seconds_flag <- as.logical(read.delim(file.path("inputs", "ignore_seconds_flag.tsv"), check.names = FALSE, stringsAsFactors = FALSE)[[1]][1])

result <- ifelse(grepl("^\\d{4}-\\d{2}-\\d{2}T\\d{2}:\\d{2}:\\d{2}$", dtc), NA_character_,
          ifelse(grepl("^\\d{4}-\\d{2}-\\d{2}T\\d{2}:\\d{2}$", dtc), if (ignore_seconds_flag) NA_character_ else "S",
          ifelse(grepl("^\\d{4}-\\d{2}-\\d{2}T\\d{2}$", dtc), "M",
          ifelse(grepl("^\\d{4}-\\d{2}-\\d{2}$", dtc), "H", NA_character_))))

dir.create("outputs", showWarnings = FALSE)
write.csv(data.frame(dtc = dtc, dtm = dtm_chr, result = result), file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

library(readr)
library(dplyr)
library(admiral)
library(stringr)

# Create outputs directory if it doesn't exist
if (!dir.exists("outputs")) {
  dir.create("outputs", recursive = TRUE)
}

# Read inputs
dtc_df <- read_tsv(
  file = file.path("inputs", "dtc.tsv"),
  show_col_types = FALSE
)

dtm_df <- read_tsv(
  file = file.path("inputs", "dtm.tsv"),
  show_col_types = FALSE
)

ignore_flag_df <- read_tsv(
  file = file.path("inputs", "ignore_seconds_flag.tsv"),
  show_col_types = FALSE
)

ignore_seconds_flag <- ignore_flag_df$ignore_seconds_flag[1]

# Join dtc and dtm to work row-wise
df <- bind_cols(dtc_df, dtm_df)

# Use admiral partial-time utilities to determine granularity
# Convert dtc to character and inspect time component using admiral helpers
df <- df %>%
  mutate(
    dtc_chr = as.character(dtc),
    # Extract time part from ISO 8601-like DTC using admiral utility
    time_part = admiral:::extract_time_dtc(dtc_chr),
    has_hour = admiral:::has_hours(time_part),
    has_min  = admiral:::has_minutes(time_part),
    has_sec  = admiral:::has_seconds(time_part)
  )

# Map flags according to specification:
# - hour-only imputation -> "M"
# - minute-only imputation -> "S"
# - date-only records     -> "H"
#
# Determine partiality based on dtc granularity:
#   date-only: no time_part
#   hour-only: has_hour & !has_min & !has_sec
#   minute-only: has_hour & has_min & !has_sec
#
# dtm is already given and not used to derive granularity per instructions
df <- df %>%
  mutate(
    result = case_when(
      is.na(time_part) | time_part == "" ~ "H",
      has_hour & !has_min & !has_sec ~ "M",
      has_hour & has_min & !has_sec ~ "S",
      TRUE ~ NA_character_
    )
  ) %>%
  select(dtc, dtm, result)

# Write output
write_csv(df, file.path("outputs", "result.csv"), na = "")
```

## Output
### Ground Truth Output
#### `result.csv`

```csv
"dtc","dtm","result"
"2020-01-01T12","2020-01-01 12:00:00","M"
"2020-01-01T12:30","2020-01-01 12:30:00","S"
"2020-01-01","2020-01-01 00:00:00","H"
```

### LLM Output
No LLM output artifact was produced.
