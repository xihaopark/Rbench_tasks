# Case 071: pharmaverse/admiral/compute_tmf

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
Write an R script for an R package task: perform compute tmf using the admiral package. Use functions from the admiral package when suitable.

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
library(stringr)
library(admiral)

# Create outputs directory if it does not exist
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

ignore_seconds_flag_df <- read_tsv(
  file = file.path("inputs", "ignore_seconds_flag.tsv"),
  show_col_types = FALSE
)

ignore_seconds <- ignore_seconds_flag_df$ignore_seconds_flag[1]

# Prepare data
df <- dtc_df %>%
  bind_cols(dtm_df) %>%
  mutate(
    # Derive analysis datetime from DTC using admiral helper
    ADTM = convert_dtc_to_dtm(
      dtc = dtc,
      highest_imputation = "h",
      include_time = TRUE
    ),
    ADTM = as.POSIXct(ADTM),
    DTM  = as.POSIXct(dtm)
  )

# Compute time imputation flag
# H: date-only (no time in DTC)
# M: hour-only imputation
# S: minute-only imputation
df <- df %>%
  mutate(
    has_time   = str_detect(dtc, "T"),
    has_minute = str_detect(dtc, ":\\d{2}"),
    has_second = str_detect(dtc, ":\\d{2}:\\d{2}$"),
    tmf = case_when(
      !has_time ~ "H",
      has_time & !has_minute ~ "M",
      has_minute & !has_second ~ "S",
      TRUE ~ ""
    )
  )

# Select required columns and write output
result <- df %>%
  select(dtc, dtm, result = tmf)

write_csv(result, file.path("outputs", "result.csv"), na = "")
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
