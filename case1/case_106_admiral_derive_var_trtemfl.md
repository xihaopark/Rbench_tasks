# Case 106: pharmaverse/admiral/derive_var_trtemfl

## Metadata
- Task ID: `pharmaverse/admiral/derive_var_trtemfl`
- Package: `admiral`
- Model: `openai/gpt-5.1`
- Status: `FAIL`
- Failure stage: `value_mismatch`

## Pass/Error
```text
The generated output files were produced, but one or more values differed from the
ground truth.

Main signal:
result.csv: Value mismatch in column: TRTEMFL.
```

## Prompt
```text
Write an R script to perform derive var trtemfl using the admiral clinical task contract.

Input: dataset.tsv
Output: result.csv


Required columns for result.csv: USUBJID, TRTSDTM, TRTEDTM, ASTDTM, AENDTM, AEITOXGR, AETOXGR, TRTEMFL
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `dataset.tsv`


```text
USUBJID	TRTSDTM	TRTEDTM	ASTDTM	AENDTM	AEITOXGR	AETOXGR
01	2021-01-01	2021-01-31	2020-12-20	2020-12-21	2	2
01	2021-01-01	2021-01-31	2021-01-15	2021-01-20	1	3
02	2021-02-01	2021-02-28	2021-03-01	2021-03-05	1	2
03	2021-03-01	2021-03-31	2021-02-20	2021-03-02	2	3
```

## Code
### Ground Truth Code

```r
suppressPackageStartupMessages(library(admiral))
suppressPackageStartupMessages(library(dplyr))

dataset <- read.delim(file.path("inputs", "dataset.tsv"), check.names = FALSE, stringsAsFactors = FALSE)
for (col in c("TRTSDTM", "TRTEDTM", "ASTDTM", "AENDTM")) {
  dataset[[col]] <- as.POSIXct(dataset[[col]], tz = "UTC")
}

result <- admiral::derive_var_trtemfl(
  dataset,
  new_var = TRTEMFL,
  trt_end_date = TRTEDTM,
  end_window = 0,
  initial_intensity = AEITOXGR,
  intensity = AETOXGR,
  subject_keys = exprs(USUBJID)
)

dir.create("outputs", showWarnings = FALSE)
write.csv(result, file.path("outputs", "result.csv"), row.names = FALSE, na = "")
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

library(dplyr)
library(readr)
library(lubridate)

# Create outputs directory if it doesn't exist
if (!dir.exists("outputs")) {
  dir.create("outputs", recursive = TRUE)
}

# Read input
adsl <- read_tsv(
  file = file.path("inputs", "dataset.tsv"),
  col_types = cols(
    USUBJID  = col_character(),
    TRTSDTM  = col_character(),
    TRTEDTM  = col_character(),
    ASTDTM   = col_character(),
    AENDTM   = col_character(),
    AEITOXGR = col_double(),
    AETOXGR  = col_double()
  )
)

# Convert to datetime (assume dates, set time to 00:00:00)
adsl <- adsl %>%
  mutate(
    TRTSDTM = ymd(TRTSDTM),
    TRTEDTM = ymd(TRTEDTM),
    ASTDTM  = ymd(ASTDTM),
    AENDTM  = ymd(AENDTM)
  )

# Derive TRTEMFL per admiral logic:
# Treatment-emergent if AE start is on/after treatment start
#   AND (AE start is on/before treatment end OR AE end is on/after treatment start)
adsl <- adsl %>%
  mutate(
    TRTEMFL = case_when(
      !is.na(TRTSDTM) &
        !is.na(ASTDTM) &
        ASTDTM >= TRTSDTM &
        (
          (!is.na(TRTEDTM) & ASTDTM <= TRTEDTM) |
            (!is.na(AENDTM) & AENDTM >= TRTSDTM)
        ) ~ "Y",
      TRUE ~ NA_character_
    )
  )

# Select required columns
result <- adsl %>%
  select(USUBJID, TRTSDTM, TRTEDTM, ASTDTM, AENDTM, AEITOXGR, AETOXGR, TRTEMFL)

# Write output
write_csv(result, file.path("outputs", "result.csv"), na = "")
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"USUBJID","TRTSDTM","TRTEDTM","ASTDTM","AENDTM","AEITOXGR","AETOXGR","TRTEMFL"
1,2021-01-01,2021-01-31,2020-12-20,2020-12-21,2,2,
1,2021-01-01,2021-01-31,2021-01-15,2021-01-20,1,3,"Y"
2,2021-02-01,2021-02-28,2021-03-01,2021-03-05,1,2,
3,2021-03-01,2021-03-31,2021-02-20,2021-03-02,2,3,"Y"
```

### LLM Output
#### `result.csv`


```csv
USUBJID,TRTSDTM,TRTEDTM,ASTDTM,AENDTM,AEITOXGR,AETOXGR,TRTEMFL
01,2021-01-01,2021-01-31,2020-12-20,2020-12-21,2,2,
01,2021-01-01,2021-01-31,2021-01-15,2021-01-20,1,3,Y
02,2021-02-01,2021-02-28,2021-03-01,2021-03-05,1,2,Y
03,2021-03-01,2021-03-31,2021-02-20,2021-03-02,2,3,
```
