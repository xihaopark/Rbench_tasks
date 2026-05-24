# Case 090: pharmaverse/admiral/create_period_dataset

## Metadata
- Task ID: `pharmaverse/admiral/create_period_dataset`
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
Write an R script to perform create period dataset using the admiral clinical task contract.

Input: periods.tsv
Output: result.csv


Required columns for result.csv: USUBJID, APERIOD, APERSDT, APEREDT
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `periods.tsv`


```text
USUBJID	APERIOD	APERSDT	APEREDT
01	1	2020-01-01	2020-01-31
01	2	2020-02-01	2020-02-28
02	1	2020-01-05	2020-01-20
```

## Code
### Ground Truth Code

```r
periods <- read.delim(file.path("inputs", "periods.tsv"), check.names = FALSE, stringsAsFactors = FALSE)
result_df <- data.frame(
  USUBJID = periods$USUBJID,
  APERIOD = as.integer(periods$APERIOD),
  APERSDT = as.Date(periods$APERSDT),
  APEREDT = as.Date(periods$APEREDT),
  stringsAsFactors = FALSE
)

dir.create("outputs", showWarnings = FALSE)
write.csv(result_df, file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

library(readr)
library(dplyr)

# Create outputs directory if it does not exist
if (!dir.exists("outputs")) {
  dir.create("outputs", recursive = TRUE)
}

# Read input
periods <- read_tsv(
  file = file.path("inputs", "periods.tsv"),
  col_types = cols(
    USUBJID = col_character(),
    APERIOD = col_integer(),
    APERSDT = col_date(format = ""),
    APEREDT = col_date(format = "")
  )
)

# Select required columns (and ensure order)
result <- periods %>%
  select(USUBJID, APERIOD, APERSDT, APEREDT)

# Write output
write_csv(result, file.path("outputs", "result.csv"), na = "")
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"USUBJID","APERIOD","APERSDT","APEREDT"
1,1,2020-01-01,2020-01-31
1,2,2020-02-01,2020-02-28
2,1,2020-01-05,2020-01-20
```

### LLM Output
#### `result.csv`


```csv
USUBJID,APERIOD,APERSDT,APEREDT
01,1,2020-01-01,2020-01-31
01,2,2020-02-01,2020-02-28
02,1,2020-01-05,2020-01-20
```
