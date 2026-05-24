# Case 115: pharmaverse/admiral/derive_vars_dtm_to_tm

## Metadata
- Task ID: `pharmaverse/admiral/derive_vars_dtm_to_tm`
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
Write an R script to perform derive vars dtm to tm using the admiral clinical task contract.

Input: dataset.tsv
Output: result.csv


Required columns for result.csv: USUBJID, ADTM, ATM
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `dataset.tsv`


```text
USUBJID	ADTM
01	2020-01-15 08:30:00
02	2020-02-20 14:45:30
03	NA
```

## Code
### Ground Truth Code

```r
dataset <- read.delim(file.path("inputs", "dataset.tsv"), check.names = FALSE, stringsAsFactors = FALSE)
adtm <- as.POSIXct(ifelse(toupper(dataset$ADTM) == "NA", NA_character_, dataset$ADTM), tz = "UTC")
atm <- ifelse(is.na(adtm), NA_character_, format(adtm, "%H:%M:%S", tz = "UTC"))
result_df <- data.frame(USUBJID = dataset$USUBJID, ADTM = adtm, ATM = atm, stringsAsFactors = FALSE)

dir.create("outputs", showWarnings = FALSE)
write.csv(result_df, file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

# Load required packages
suppressPackageStartupMessages({
  library(readr)
  library(dplyr)
  library(stringr)
  library(lubridate)
})

# Create outputs directory if it doesn't exist
if (!dir.exists("outputs")) {
  dir.create("outputs", recursive = TRUE)
}

# Read input
input_path <- file.path("inputs", "dataset.tsv")
dat <- read_tsv(
  input_path,
  col_types = cols(
    USUBJID = col_character(),
    ADTM = col_character()
  )
)

# Derive ADTM as POSIXct and ATM as time (HH:MM:SS) from ADTM
dat_derived <- dat %>%
  mutate(
    ADTM = if_else(
      is.na(ADTM) | ADTM %in% c("", "NA"),
      NA_character_,
      ADTM
    ),
    ADTM_DT = suppressWarnings(ymd_hms(ADTM, quiet = TRUE)),
    ATM = if_else(
      is.na(ADTM_DT),
      NA_character_,
      format(ADTM_DT, "%H:%M:%S")
    )
  ) %>%
  transmute(
    USUBJID,
    ADTM = ADTM_DT,
    ATM
  )

# Write output
output_path <- file.path("outputs", "result.csv")
write_csv(dat_derived, output_path, na = "")
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"USUBJID","ADTM","ATM"
1,2020-01-15 08:30:00,"08:30:00"
2,2020-02-20 14:45:30,"14:45:30"
3,NA,NA
```

### LLM Output
#### `result.csv`


```csv
USUBJID,ADTM,ATM
01,2020-01-15T08:30:00Z,08:30:00
02,2020-02-20T14:45:30Z,14:45:30
03,,
```
