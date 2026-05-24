# Case 114: pharmaverse/admiral/derive_vars_dtm_to_dt

## Metadata
- Task ID: `pharmaverse/admiral/derive_vars_dtm_to_dt`
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
Write an R script to perform derive vars dtm to dt using the admiral clinical task contract.

Input: dataset.tsv
Output: result.csv


Required columns for result.csv: USUBJID, ADTM, ADT
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `dataset.tsv`


```text
USUBJID	ADTM
01	2020-01-15 08:30:00
02	2020-02-20 14:45:00
03	NA
```

## Code
### Ground Truth Code

```r
dataset <- read.delim(file.path("inputs", "dataset.tsv"), check.names = FALSE, stringsAsFactors = FALSE)
adtm <- as.POSIXct(ifelse(toupper(dataset$ADTM) == "NA", NA_character_, dataset$ADTM), tz = "UTC")
adt <- as.Date(adtm)
result_df <- data.frame(USUBJID = dataset$USUBJID, ADTM = adtm, ADT = adt, stringsAsFactors = FALSE)

dir.create("outputs", showWarnings = FALSE)
write.csv(result_df, file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

library(admiral)
library(dplyr)
library(readr)
library(lubridate)

# Create outputs directory if it doesn't exist
if (!dir.exists("outputs")) dir.create("outputs", recursive = TRUE)

# Read input
dt <- read_tsv(
  file = file.path("inputs", "dataset.tsv"),
  col_types = cols(
    USUBJID = col_character(),
    ADTM    = col_character()
  )
)

# Convert ADTM to datetime (POSIXct); keep NA as is
dt <- dt %>%
  mutate(
    ADTM = parse_datetime(ADTM, format = "%Y-%m-%d %H:%M:%S")
  )

# Derive ADT from ADTM using admiral
dt <- dt %>%
  derive_vars_dtm_to_dt(
    dtm = ADTM,
    new_vars_prefix = "AD"
  )

# Keep only required columns
result <- dt %>%
  select(USUBJID, ADTM, ADT)

# Write output
write_csv(result, file.path("outputs", "result.csv"), na = "")
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"USUBJID","ADTM","ADT"
1,2020-01-15 08:30:00,2020-01-15
2,2020-02-20 14:45:00,2020-02-20
3,NA,NA
```

### LLM Output
No LLM output artifact was produced.
