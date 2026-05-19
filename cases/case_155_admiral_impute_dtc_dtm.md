# Case 155: pharmaverse/admiral/impute_dtc_dtm

## Metadata
- Task ID: `pharmaverse/admiral/impute_dtc_dtm`
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
Write an R script to perform impute dtc dtm using the admiral clinical task contract.

Input: date_imputation.tsv, dtc.tsv, time_imputation.tsv
Output: result.csv


Required columns for result.csv: dtc, result
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `date_imputation.tsv`


```text
date_imputation
first
```

#### `dtc.tsv`


```text
dtc
2020-06-15T12:30:00
2020-06
2020
NA
```

#### `time_imputation.tsv`


```text
time_imputation
first
```

## Code
### Ground Truth Code

```r
dtc <- as.character(read.delim(file.path("inputs", "dtc.tsv"), check.names = FALSE, stringsAsFactors = FALSE)$dtc)
missing <- is.na(dtc) | toupper(dtc) == "NA" | dtc == ""
result_chr <- ifelse(grepl("^[0-9]{4}$", dtc), paste0(dtc, "-01-01 00:00:00"),
  ifelse(grepl("^[0-9]{4}-[0-9]{2}$", dtc), paste0(dtc, "-01 00:00:00"),
    ifelse(grepl("^[0-9]{4}-[0-9]{2}-[0-9]{2}T[0-9]{2}:[0-9]{2}:[0-9]{2}$", dtc), sub("T", " ", dtc), NA_character_)))
result <- as.POSIXct(ifelse(missing, NA_character_, result_chr), tz = "UTC")

dir.create("outputs", showWarnings = FALSE)
write.csv(data.frame(dtc = dtc, result = result), file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

# Load required packages
suppressPackageStartupMessages({
  library(readr)
  library(dplyr)
  library(stringr)
})

# Create outputs directory if it does not exist
if (!dir.exists("outputs")) {
  dir.create("outputs", recursive = TRUE)
}

# Read inputs
date_imp <- read_tsv(
  file = file.path("inputs", "date_imputation.tsv"),
  col_types = cols(.default = col_character())
) %>%
  pull(date_imputation) %>%
  tolower()

time_imp <- read_tsv(
  file = file.path("inputs", "time_imputation.tsv"),
  col_types = cols(.default = col_character())
) %>%
  pull(time_imputation) %>%
  tolower()

dtc_df <- read_tsv(
  file = file.path("inputs", "dtc.tsv"),
  col_types = cols(.default = col_character())
)

# Helper functions for imputation
impute_date <- function(dtc, method) {
  # dtc may be "YYYY", "YYYY-MM", "YYYY-MM-DD", or NA
  if (is.na(dtc) || dtc == "") return(NA_character_)

  # Split date and time if present
  parts <- str_split_fixed(dtc, "T", 2)
  date_part <- parts[, 1]

  # Detect granularity
  n_dash <- str_count(date_part, "-")

  if (n_dash == 0) {
    # Only year
    yyyy <- date_part
    if (method == "first") {
      return(paste0(yyyy, "-01-01"))
    } else if (method == "last") {
      return(paste0(yyyy, "-12-31"))
    } else {
      return(paste0(yyyy, "-06-30"))
    }
  } else if (n_dash == 1) {
    # Year and month
    yyyy_mm <- date_part
    yyyy <- str_sub(yyyy_mm, 1, 4)
    mm <- str_sub(yyyy_mm, 6, 7)
    if (method == "first") {
      return(paste0(yyyy, "-", mm, "-01"))
    } else if (method == "last") {
      # Last day of month
      last_day <- as.integer(format(
        as.Date(paste0(yyyy, "-", mm, "-01")) + months(1) - 1,
        "%d"
      ))
      return(sprintf("%s-%s-%02d", yyyy, mm, last_day))
    } else {
      # Mid-month (15th)
      return(paste0(yyyy, "-", mm, "-15"))
    }
  } else {
    # Full date present; return as is
    return(date_part)
  }
}

impute_time <- function(dtc, method) {
  # dtc may have time already or not, may be NA
  if (is.na(dtc) || dtc == "") return(NA_character_)

  parts <- str_split_fixed(dtc, "T", 2)
  if (parts[, 2] != "") {
    # Already has time; keep time
    return(parts[, 2])
  }

  if (method == "first") {
    return("00:00:00")
  } else if (method == "last") {
    return("23:59:59")
  } else {
    return("12:00:00")
  }
}

# Apply imputation following admiral-like logic
result_df <- dtc_df %>%
  mutate(
    imputed_date = vapply(dtc, impute_date, character(1), method = date_imp),
    imputed_time = vapply(dtc, impute_time, character(1), method = time_imp),
    result = if_else(
      is.na(imputed_date),
      NA_character_,
      paste0(imputed_date, "T", imputed_time)
    )
  ) %>%
  select(dtc, result)

# Write output
write.csv(result_df, file = file.path("outputs", "result.csv"), row.names = FALSE, na = "")
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"dtc","result"
"2020-06-15T12:30:00",2020-06-15 12:30:00
"2020-06",2020-06-01
"2020",2020-01-01
NA,NA
```

### LLM Output
#### `result.csv`


```csv
"dtc","result"
"2020-06-15T12:30:00","2020-06-15T12:30:00"
"2020-06","2020-06-01T00:00:00"
"2020","2020-01-01T00:00:00"
,
```
