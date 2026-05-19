# Case 113: pharmaverse/admiral/derive_vars_dtm

## Metadata
- Task ID: `pharmaverse/admiral/derive_vars_dtm`
- Package: `admiral`
- Model: `openai/gpt-5.1`
- Status: `FAIL`
- Failure stage: `value_mismatch`

## Pass/Error
```text
The generated output files were produced, but one or more values differed from the
ground truth.

Main signal:
result.csv: Value mismatch in column: ADTM.
```

## Prompt
```text
Write an R script to perform derive vars dtm using the admiral clinical task contract.

Input: dataset.tsv
Output: result.csv


Required columns for result.csv: USUBJID, DTC, ADTM
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `dataset.tsv`


```text
USUBJID	DTC
01	2020-01-15T08:30:00
02	2020-02-20T14:45:00
03	2020-03
04	NA
```

## Code
### Ground Truth Code

```r
dataset <- read.delim(file.path("inputs", "dataset.tsv"), check.names = FALSE, stringsAsFactors = FALSE)
dtc <- as.character(dataset$DTC)
complete_dtm <- grepl("^[0-9]{4}-[0-9]{2}-[0-9]{2}T[0-9]{2}:[0-9]{2}:[0-9]{2}$", dtc)
adtm <- as.POSIXct(ifelse(complete_dtm, sub("T", " ", dtc), NA_character_), tz = "UTC")
result_df <- data.frame(USUBJID = dataset$USUBJID, DTC = dtc, ADTM = adtm, stringsAsFactors = FALSE)

dir.create("outputs", showWarnings = FALSE)
write.csv(result_df, file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

library(readr)
library(dplyr)
library(stringr)
library(lubridate)

# Create outputs directory if it doesn't exist
if (!dir.exists("outputs")) {
  dir.create("outputs", recursive = TRUE)
}

# Read input
df <- read_tsv(
  file = file.path("inputs", "dataset.tsv"),
  col_types = cols(
    USUBJID = col_character(),
    DTC = col_character()
  )
)

# Helper to parse partial ISO8601-like dates
parse_partial_iso <- function(x) {
  if (is.na(x) || x == "") return(NA_character_)

  # Full datetime with time and 'T'
  if (str_detect(x, "^\\d{4}-\\d{2}-\\d{2}T\\d{2}:\\d{2}:\\d{2}$")) {
    dt <- ymd_hms(x, quiet = TRUE)
    return(format(dt, "%Y-%m-%dT%H:%M:%S"))
  }

  # Date only (YYYY-MM-DD)
  if (str_detect(x, "^\\d{4}-\\d{2}-\\d{2}$")) {
    dt <- ymd(x, quiet = TRUE)
    return(if (!is.na(dt)) paste0(format(dt, "%Y-%m-%d"), "T00:00:00") else NA_character_)
  }

  # Year-month only (YYYY-MM)
  if (str_detect(x, "^\\d{4}-\\d{2}$")) {
    dt <- ymd(paste0(x, "-01"), quiet = TRUE)
    return(if (!is.na(dt)) paste0(format(dt, "%Y-%m-%d"), "T00:00:00") else NA_character_)
  }

  # Year only (YYYY)
  if (str_detect(x, "^\\d{4}$")) {
    dt <- ymd(paste0(x, "-01-01"), quiet = TRUE)
    return(if (!is.na(dt)) paste0(format(dt, "%Y-%m-%d"), "T00:00:00") else NA_character_)
  }

  NA_character_
}

# Derive ADTM from DTC per admiral-like logic (assuming local time, no TZ)
result <- df %>%
  mutate(
    DTC = na_if(DTC, "NA"),
    ADTM = vapply(DTC, parse_partial_iso, character(1))
  ) %>%
  select(USUBJID, DTC, ADTM)

# Write output
write_csv(result, file.path("outputs", "result.csv"), na = "")
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"USUBJID","DTC","ADTM"
1,"2020-01-15T08:30:00",2020-01-15 08:30:00
2,"2020-02-20T14:45:00",2020-02-20 14:45:00
3,"2020-03",NA
4,NA,NA
```

### LLM Output
#### `result.csv`


```csv
USUBJID,DTC,ADTM
01,2020-01-15T08:30:00,2020-01-15T08:30:00
02,2020-02-20T14:45:00,2020-02-20T14:45:00
03,2020-03,2020-03-01T00:00:00
04,,
```
