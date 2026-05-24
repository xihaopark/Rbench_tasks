# Case 105: pharmaverse/admiral/derive_var_trtdurd

## Metadata
- Task ID: `pharmaverse/admiral/derive_var_trtdurd`
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
Write an R script to perform derive var trtdurd using the admiral clinical task contract.

Input: dataset.tsv
Output: result.csv


Required columns for result.csv: USUBJID, TRTSDT, TRTEDT, TRTDURD
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `dataset.tsv`


```text
USUBJID	TRTSDT	TRTEDT
01	2020-01-01	2020-01-10
02	2020-02-01	2020-02-01
03	2020-03-01	NA
```

## Code
### Ground Truth Code

```r
dataset <- read.delim(file.path("inputs", "dataset.tsv"), check.names = FALSE, stringsAsFactors = FALSE)
trtsdt <- as.Date(dataset$TRTSDT)
trtedt <- as.Date(ifelse(toupper(dataset$TRTEDT) == "NA", NA_character_, dataset$TRTEDT))
trtdurd <- ifelse(is.na(trtsdt) | is.na(trtedt), NA_integer_, as.integer(trtedt - trtsdt) + 1L)
result_df <- data.frame(USUBJID = dataset$USUBJID, TRTSDT = trtsdt, TRTEDT = trtedt, TRTDURD = trtdurd, stringsAsFactors = FALSE)

dir.create("outputs", showWarnings = FALSE)
write.csv(result_df, file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

suppressPackageStartupMessages({
  library(readr)
  library(dplyr)
  library(lubridate)
})

# Create outputs directory if it does not exist
if (!dir.exists("outputs")) {
  dir.create("outputs", recursive = TRUE)
}

# Read input
adsl <- read_tsv(
  file = file.path("inputs", "dataset.tsv"),
  col_types = cols(
    USUBJID = col_character(),
    TRTSDT  = col_character(),
    TRTEDT  = col_character()
  )
)

# Derive TRTSDT and TRTEDT as Date
adsl_der <- adsl %>%
  mutate(
    TRTSDT = ymd(TRTSDT),
    TRTEDT = ymd(TRTEDT)
  ) %>%
  mutate(
    TRTDURD = if_else(
      !is.na(TRTSDT) & !is.na(TRTEDT),
      as.integer(TRTEDT - TRTSDT) + 1L,
      NA_integer_
    )
  ) %>%
  select(USUBJID, TRTSDT, TRTEDT, TRTDURD)

# Write output
write_csv(adsl_der, file.path("outputs", "result.csv"), na = "")
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"USUBJID","TRTSDT","TRTEDT","TRTDURD"
1,2020-01-01,2020-01-10,10
2,2020-02-01,2020-02-01,1
3,2020-03-01,NA,NA
```

### LLM Output
#### `result.csv`


```csv
USUBJID,TRTSDT,TRTEDT,TRTDURD
01,2020-01-01,2020-01-10,10
02,2020-02-01,2020-02-01,1
03,2020-03-01,,
```
