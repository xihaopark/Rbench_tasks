# Case 103: pharmaverse/admiral/derive_var_ontrtfl

## Metadata
- Task ID: `pharmaverse/admiral/derive_var_ontrtfl`
- Package: `admiral`
- Model: `openai/gpt-5.1`
- Status: `PASS`
- Failure stage: `pass`

## Pass/Error
```text
pass
```

## Prompt
```text
Write an R script to perform derive var ontrtfl using the admiral clinical task contract.

Input: dataset.tsv
Output: result.csv


Required columns for result.csv: USUBJID, ADT, TRTSDT, TRTEDT, ONTRTFL
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `dataset.tsv`


```text
USUBJID	ADT	TRTSDT	TRTEDT
P01	2020-02-24	2020-01-01	2020-03-01
P02	2020-01-01	2020-01-01	2020-03-01
P03	2019-12-31	2020-01-01	2020-03-01
P04	2020-03-05	2020-01-01	2020-03-01
```

## Code
### Ground Truth Code

```r
suppressPackageStartupMessages(library(admiral))

dataset <- read.delim(file.path("inputs", "dataset.tsv"), check.names = FALSE, stringsAsFactors = FALSE)
for (col in c("ADT", "TRTSDT", "TRTEDT")) {
  dataset[[col]] <- as.Date(dataset[[col]])
}

result <- admiral::derive_var_ontrtfl(
  dataset,
  start_date = ADT,
  ref_start_date = TRTSDT,
  ref_end_date = TRTEDT
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

# Ensure outputs directory exists
if (!dir.exists("outputs")) dir.create("outputs", recursive = TRUE)

# Read input
adsl <- read.delim(
  file = file.path("inputs", "dataset.tsv"),
  sep = "\t",
  header = TRUE,
  stringsAsFactors = FALSE
)

# Convert dates
adsl <- adsl %>%
  mutate(
    ADT    = ymd(ADT),
    TRTSDT = ymd(TRTSDT),
    TRTEDT = ymd(TRTEDT)
  )

# Derive ONTRTFL per admiral logic (ADT between TRTSDT and TRTEDT inclusive)
result <- adsl %>%
  mutate(
    ONTRTFL = if_else(!is.na(ADT) & !is.na(TRTSDT) & !is.na(TRTEDT) &
                        ADT >= TRTSDT & ADT <= TRTEDT,
                      "Y", NA_character_)
  ) %>%
  select(USUBJID, ADT, TRTSDT, TRTEDT, ONTRTFL)

# Write output
write_csv(result, file.path("outputs", "result.csv"), na = "")
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"USUBJID","ADT","TRTSDT","TRTEDT","ONTRTFL"
"P01",2020-02-24,2020-01-01,2020-03-01,"Y"
"P02",2020-01-01,2020-01-01,2020-03-01,"Y"
"P03",2019-12-31,2020-01-01,2020-03-01,
"P04",2020-03-05,2020-01-01,2020-03-01,
```

### LLM Output
#### `result.csv`


```csv
USUBJID,ADT,TRTSDT,TRTEDT,ONTRTFL
P01,2020-02-24,2020-01-01,2020-03-01,Y
P02,2020-01-01,2020-01-01,2020-03-01,Y
P03,2019-12-31,2020-01-01,2020-03-01,
P04,2020-03-05,2020-01-01,2020-03-01,
```
