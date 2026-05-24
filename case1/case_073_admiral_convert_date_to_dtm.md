# Case 073: pharmaverse/admiral/convert_date_to_dtm

## Metadata
- Task ID: `pharmaverse/admiral/convert_date_to_dtm`
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
Write an R script to perform convert date to dtm using the admiral clinical task contract.

Input: dt.tsv
Output: result.csv


Required columns for result.csv: dt, result
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `dt.tsv`


```text
dt
2020-01-01
2020-06-15
NA
```

## Code
### Ground Truth Code

```r
suppressPackageStartupMessages(library(admiral))

dt <- as.Date(read.delim(file.path("inputs", "dt.tsv"), check.names = FALSE, stringsAsFactors = FALSE, blank.lines.skip = FALSE)$dt)
result <- as.POSIXct(dt, tz = "UTC")

dir.create("outputs", showWarnings = FALSE)
write.csv(data.frame(dt = dt, result = result), file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

library(dplyr)
library(readr)
library(stringr)
library(lubridate)
library(admiral)

# Create outputs directory if it does not exist
if (!dir.exists("outputs")) dir.create("outputs", recursive = TRUE)

# Read input
dt <- read_tsv(
  file = file.path("inputs", "dt.tsv"),
  col_types = cols(
    dt = col_character()
  )
)

# Convert dt to datetime (DTM) per ADaM conventions (assume ISO date, set 00:00 time)
result <- dt %>%
  mutate(
    result = case_when(
      is.na(dt) ~ NA_character_,
      dt == "" ~ NA_character_,
      TRUE ~ format(as_datetime(ymd(dt)), "%Y-%m-%dT%H:%M:%S")
    )
  ) %>%
  select(dt, result)

# Write output
write_csv(result, file.path("outputs", "result.csv"), na = "")
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"dt","result"
2020-01-01,2020-01-01
2020-06-15,2020-06-15
NA,NA
```

### LLM Output
#### `result.csv`


```csv
dt,result
2020-01-01,2020-01-01T00:00:00
2020-06-15,2020-06-15T00:00:00
,
```
