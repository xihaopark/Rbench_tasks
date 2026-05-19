# Case 112: pharmaverse/admiral/derive_vars_dt

## Metadata
- Task ID: `pharmaverse/admiral/derive_vars_dt`
- Package: `admiral`
- Model: `openai/gpt-5.1`
- Status: `FAIL`
- Failure stage: `value_mismatch`

## Pass/Error
```text
The generated output files were produced, but one or more values differed from the
ground truth.

Main signal:
result.csv: Value mismatch in column: ADT.
```

## Prompt
```text
Write an R script to perform derive vars dt using the admiral clinical task contract.

Input: dataset.tsv
Output: result.csv


Required columns for result.csv: USUBJID, DTC, ADT
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `dataset.tsv`


```text
USUBJID	DTC
01	2020-01-15
02	2020-02
03	2020
04	NA
```

## Code
### Ground Truth Code

```r
dataset <- read.delim(file.path("inputs", "dataset.tsv"), check.names = FALSE, stringsAsFactors = FALSE)
dtc <- as.character(dataset$DTC)
complete_date <- grepl("^[0-9]{4}-[0-9]{2}-[0-9]{2}$", dtc)
adt <- as.Date(ifelse(complete_date, dtc, NA_character_))
result_df <- data.frame(USUBJID = dataset$USUBJID, DTC = dtc, ADT = adt, stringsAsFactors = FALSE)

dir.create("outputs", showWarnings = FALSE)
write.csv(result_df, file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

# Load required package
suppressPackageStartupMessages({
  library(readr)
  library(dplyr)
  library(stringr)
  library(lubridate)
})

# Ensure outputs directory exists
if (!dir.exists("outputs")) {
  dir.create("outputs", recursive = TRUE)
}

# Read input
infile <- file.path("inputs", "dataset.tsv")
df <- read_tsv(infile, show_col_types = FALSE)

# Derive ADT from DTC according to partial date rules
df_out <- df %>%
  mutate(
    DTC = na_if(DTC, "NA"),
    ADT = case_when(
      is.na(DTC) ~ as.Date(NA),
      str_detect(DTC, "^\\d{4}-\\d{2}-\\d{2}$") ~ ymd(DTC),
      str_detect(DTC, "^\\d{4}-\\d{2}$") ~ ymd(paste0(DTC, "-01")),
      str_detect(DTC, "^\\d{4}$") ~ ymd(paste0(DTC, "-01-01")),
      TRUE ~ as.Date(NA)
    )
  ) %>%
  select(USUBJID, DTC, ADT)

# Write output
outfile <- file.path("outputs", "result.csv")
write_csv(df_out, outfile, na = "")
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"USUBJID","DTC","ADT"
1,"2020-01-15",2020-01-15
2,"2020-02",NA
3,"2020",NA
4,NA,NA
```

### LLM Output
#### `result.csv`


```csv
USUBJID,DTC,ADT
01,2020-01-15,2020-01-15
02,2020-02,2020-02-01
03,2020,2020-01-01
04,,
```
