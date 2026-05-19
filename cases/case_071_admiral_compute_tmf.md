# Case 071: pharmaverse/admiral/compute_tmf

## Metadata
- Task ID: `pharmaverse/admiral/compute_tmf`
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
Write an R script to perform compute tmf using the admiral clinical task contract.

Input: dtc.tsv, dtm.tsv, ignore_seconds_flag.tsv
Output: result.csv


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

suppressPackageStartupMessages({
  library(readr)
  library(dplyr)
  library(stringr)
  library(lubridate)
})

# Create outputs directory if it doesn't exist
if (!dir.exists("outputs")) dir.create("outputs", recursive = TRUE)

# Read inputs
dtc_df <- read_tsv(
  file = file.path("inputs", "dtc.tsv"),
  col_types = cols(.default = col_character())
)

dtm_df <- read_tsv(
  file = file.path("inputs", "dtm.tsv"),
  col_types = cols(.default = col_character())
)

flag_df <- read_tsv(
  file = file.path("inputs", "ignore_seconds_flag.tsv"),
  col_types = cols(ignore_seconds_flag = col_logical())
)

ignore_seconds_flag <- flag_df$ignore_seconds_flag[[1]]

# Function to compute tmf as in admiral derive_vars_dtm
compute_tmf <- function(dtc, dtm, ignore_seconds_flag = FALSE) {
  # If dtc is NA or blank, result is NA
  if (is.na(dtc) || str_trim(dtc) == "") return(NA_character_)
  if (is.na(dtm) || str_trim(dtm) == "") return(NA_character_)

  # Normalize dtc formats (allowed: YYYY-MM-DD, YYYY-MM-DDThh, YYYY-MM-DDThh:mm)
  # Replace 'T' with space for easier handling
  dtc_norm <- str_replace(dtc, "T", " ")

  # Determine dtc precision
  # date only
  if (str_detect(dtc_norm, "^\\d{4}-\\d{2}-\\d{2}$")) {
    prec <- "date"
  } else if (str_detect(dtc_norm, "^\\d{4}-\\d{2}-\\d{2} \\d{2}$")) {
    prec <- "hour"
  } else if (str_detect(dtc_norm, "^\\d{4}-\\d{2}-\\d{2} \\d{2}:\\d{2}$")) {
    prec <- "minute"
  } else {
    # Unsupported or invalid dtc format => NA
    return(NA_character_)
  }

  # Parse dtm as POSIXct with seconds
  dtm_parsed <- suppressWarnings(
    ymd_hms(dtm, quiet = TRUE, tz = "UTC")
  )
  if (is.na(dtm_parsed)) return(NA_character_)

  # Build reference POSIXct from dtc according to its precision
  if (prec == "date") {
    ref <- suppressWarnings(ymd_hms(paste0(dtc_norm, " 00:00:00"), quiet = TRUE, tz = "UTC"))
  } else if (prec == "hour") {
    ref <- suppressWarnings(ymd_hms(paste0(dtc_norm, ":00:00"), quiet = TRUE, tz = "UTC"))
  } else { # minute
    ref <- suppressWarnings(ymd_hms(paste0(dtc_norm, ":00"), quiet = TRUE, tz = "UTC"))
  }
  if (is.na(ref)) return(NA_character_)

  # If ignore_seconds_flag, drop seconds from dtm before comparison
  if (ignore_seconds_flag) {
    dtm_parsed <- floor_date(dtm_parsed, unit = "minute")
  }

  # Extract components for comparison based on precision
  y_dtm <- year(dtm_parsed)
  m_dtm <- month(dtm_parsed)
  d_dtm <- mday(dtm_parsed)
  H_dtm <- hour(dtm_parsed)
  M_dtm <- minute(dtm_parsed)
  S_dtm <- second(dtm_parsed)

  y_ref <- year(ref)
  m_ref <- month(ref)
  d_ref <- mday(ref)
  H_ref <- hour(ref)
  M_ref <- minute(ref)
  S_ref <- second(ref)

  # For "date" precision: all must match (year, month, day, time = 00:00:00)
  if (prec == "date") {
    if (y_dtm == y_ref && m_dtm == m_ref && d_dtm == d_ref &&
        H_dtm == 0 && M_dtm == 0 && S_dtm == 0) {
      return("Y")
    } else {
      return("N")
    }
  }

  # For "hour" precision: match year, month, day, hour; minutes/seconds must be 0
  if (prec == "hour") {
    if (y_dtm == y_ref && m_dtm == m_ref && d_dtm == d_ref &&
        H_dtm == H_ref && M_dtm == 0 && S_dtm == 0) {
      return("Y")
    } else {
      return("N")
    }
  }

  # For "minute" precision: match year, month, day, hour, minute; seconds must be 0 (unless ignored)
  if (prec == "minute") {
    if (y_dtm == y_ref && m_dtm == m_ref && d_dtm == d_ref &&
        H_dtm == H_ref && M_dtm == M_ref &&
        (ignore_seconds_flag || S_dtm == 0)) {
      return("Y")
    } else {
      return("N")
    }
  }

  NA_character_
}

# Vectorized application
res <- mapply(
  compute_tmf,
  dtc = dtc_df$dtc,
  dtm = dtm_df$dtm,
  MoreArgs = list(ignore_seconds_flag = ignore_seconds_flag),
  USE.NAMES = FALSE
)

result_df <- tibble(
  dtc = dtc_df$dtc,
  dtm = dtm_df$dtm,
  result = res
)

write_csv(result_df, file.path("outputs", "result.csv"), na = "")
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
#### `result.csv`


```csv
dtc,dtm,result
2020-01-01T12,2020-01-01 12:00:00,Y
2020-01-01T12:30,2020-01-01 12:30:00,Y
2020-01-01,2020-01-01 00:00:00,Y
```
