# Case 074: pharmaverse/admiral/convert_dtc_to_dt

## Metadata
- Task ID: `pharmaverse/admiral/convert_dtc_to_dt`
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
Write an R script to perform convert dtc to dt using the admiral clinical task contract.

Input: dtc.tsv
Output: result.csv


Required columns for result.csv: dtc, result
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `dtc.tsv`


```text
dtc
2020-01-01
2020-06
2020
NA
```

## Code
### Ground Truth Code

```r
suppressPackageStartupMessages(library(admiral))

dtc <- as.character(read.delim(file.path("inputs", "dtc.tsv"), check.names = FALSE, stringsAsFactors = FALSE, blank.lines.skip = FALSE)$dtc)
complete_date <- grepl("^[0-9]{4}-[0-9]{2}-[0-9]{2}$", dtc)
result <- as.Date(ifelse(complete_date, dtc, NA_character_))

dir.create("outputs", showWarnings = FALSE)
write.csv(data.frame(dtc = dtc, result = result), file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

suppressPackageStartupMessages({
  library(dplyr)
  library(readr)
  library(stringr)
})

# Create outputs directory if it doesn't exist
if (!dir.exists("outputs")) dir.create("outputs", recursive = TRUE)

# Read input
dtc_df <- read_tsv(
  file = file.path("inputs", "dtc.tsv"),
  col_types = cols(.default = col_character())
)

# Function to convert DTC to DT (YYYY-MM-DD) using CDISC/ADaM-like rules
convert_dtc_to_dt <- function(dtc) {
  if (is.na(dtc) || str_trim(dtc) == "") return(NA_character_)

  # Full date
  if (str_detect(dtc, "^\\d{4}-\\d{2}-\\d{2}$")) {
    return(dtc)
  }

  # Year-month only (YYYY-MM) -> assume day = 01
  if (str_detect(dtc, "^\\d{4}-\\d{2}$")) {
    return(paste0(dtc, "-01"))
  }

  # Year only (YYYY) -> assume month = 01, day = 01
  if (str_detect(dtc, "^\\d{4}$")) {
    return(paste0(dtc, "-01-01"))
  }

  # If pattern not recognized, return NA
  return(NA_character_)
}

result_df <- dtc_df %>%
  mutate(result = vapply(dtc, convert_dtc_to_dt, character(1))) %>%
  select(dtc, result)

# Write output
write_csv(result_df, file.path("outputs", "result.csv"), na = "")
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"dtc","result"
"2020-01-01",2020-01-01
"2020-06",NA
"2020",NA
NA,NA
```

### LLM Output
#### `result.csv`


```csv
dtc,result
2020-01-01,2020-01-01
2020-06,2020-06-01
2020,2020-01-01
,
```
