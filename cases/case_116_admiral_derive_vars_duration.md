# Case 116: pharmaverse/admiral/derive_vars_duration

## Metadata
- Task ID: `pharmaverse/admiral/derive_vars_duration`
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
Write an R script to perform derive vars duration using the admiral clinical task contract.

Input: dataset.tsv
Output: result.csv


Required columns for result.csv: USUBJID, STARTDT, ENDDT, DURD
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `dataset.tsv`


```text
USUBJID	STARTDT	ENDDT
01	2020-01-01	2020-01-10
02	2020-02-01	2020-02-01
03	2020-03-01	NA
```

## Code
### Ground Truth Code

```r
dataset <- read.delim(file.path("inputs", "dataset.tsv"), check.names = FALSE, stringsAsFactors = FALSE)
startdt <- as.Date(dataset$STARTDT)
enddt <- as.Date(ifelse(toupper(dataset$ENDDT) == "NA", NA_character_, dataset$ENDDT))
durd <- ifelse(is.na(startdt) | is.na(enddt), NA_integer_, as.integer(enddt - startdt) + 1L)
result_df <- data.frame(USUBJID = dataset$USUBJID, STARTDT = startdt, ENDDT = enddt, DURD = durd, stringsAsFactors = FALSE)

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
if (!dir.exists("outputs")) dir.create("outputs", recursive = TRUE)

# Read input
infile <- file.path("inputs", "dataset.tsv")
dat <- read_tsv(infile, show_col_types = FALSE, na = c("NA", ""))

# Derive duration in days (inclusive) per admiral-like approach
# DURD = ENDDT - STARTDT + 1, NA if either date is missing
result <- dat %>%
  mutate(
    STARTDT = ymd(STARTDT),
    ENDDT   = ymd(ENDDT),
    DURD    = if_else(
      !is.na(STARTDT) & !is.na(ENDDT),
      as.integer(ENDDT - STARTDT + 1L),
      NA_integer_
    )
  ) %>%
  select(USUBJID, STARTDT, ENDDT, DURD)

# Write output
outfile <- file.path("outputs", "result.csv")
write_csv(result, outfile, na = "")
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"USUBJID","STARTDT","ENDDT","DURD"
1,2020-01-01,2020-01-10,10
2,2020-02-01,2020-02-01,1
3,2020-03-01,NA,NA
```

### LLM Output
#### `result.csv`


```csv
USUBJID,STARTDT,ENDDT,DURD
01,2020-01-01,2020-01-10,10
02,2020-02-01,2020-02-01,1
03,2020-03-01,,
```
