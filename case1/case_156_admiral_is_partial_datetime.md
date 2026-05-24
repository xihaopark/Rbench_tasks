# Case 156: pharmaverse/admiral/is_partial_datetime

## Metadata
- Task ID: `pharmaverse/admiral/is_partial_datetime`
- Package: `admiral`
- Model: `openai/gpt-5.1`
- Status: `FAIL`
- Failure stage: `value_mismatch`

## Pass/Error
```text
The generated output files were produced, but one or more values differed from the
ground truth.

Main signal:
result.csv: Numeric missingness mismatch in column: result.
```

## Prompt
```text
Write an R script to perform is partial datetime using the admiral clinical task contract.

Input: dtc.tsv
Output: result.csv


Required columns for result.csv: dtc, result
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `dtc.tsv`


```text
dtc
2020
2020-06
2020-06-15
2020-06-15T12:30
2020-06-15T12:30:00
NA
```

## Code
### Ground Truth Code

```r
dtc <- as.character(read.delim(file.path("inputs", "dtc.tsv"), check.names = FALSE, stringsAsFactors = FALSE)$dtc)
is_missing <- is.na(dtc) | dtc == "" | toupper(dtc) == "NA"
complete_date <- grepl("^[0-9]{4}-[0-9]{2}-[0-9]{2}$", dtc)
complete_dtm <- grepl("^[0-9]{4}-[0-9]{2}-[0-9]{2}T[0-9]{2}:[0-9]{2}:[0-9]{2}$", dtc)
partial_date <- grepl("^[0-9]{4}(-[0-9]{2})?$", dtc)
partial_dtm <- grepl("^[0-9]{4}-[0-9]{2}-[0-9]{2}T[0-9]{2}(:[0-9]{2})?$", dtc)
result <- !is_missing & (partial_date | partial_dtm) & !(complete_date | complete_dtm)

dir.create("outputs", showWarnings = FALSE)
write.csv(data.frame(dtc = dtc, result = result), file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

# Load required packages
suppressPackageStartupMessages({
  library(readr)
  library(dplyr)
  library(stringr)
})

# Create outputs directory if it doesn't exist
if (!dir.exists("outputs")) dir.create("outputs", recursive = TRUE)

# Read input
dtc_df <- read_tsv(
  file = "inputs/dtc.tsv",
  col_types = cols(
    dtc = col_character()
  )
)

# Function to determine if partial datetime according to ADaM/ADmiral rules
is_partial_dtc <- function(x) {
  x <- str_trim(x)

  # Missing or blank is not a datetime (return NA)
  is_blank <- x == "" | is.na(x)
  ifelse(
    is_blank, NA,
    # Year only (YYYY)
    ifelse(str_detect(x, "^\\d{4}$"), TRUE,
    # Year-month only (YYYY-MM)
    ifelse(str_detect(x, "^\\d{4}-\\d{2}$"), TRUE,
    # Year-month-day only (YYYY-MM-DD)
    ifelse(str_detect(x, "^\\d{4}-\\d{2}-\\d{2}$"), TRUE,
    # Date + hour:minute (YYYY-MM-DDThh:mm)
    ifelse(str_detect(x, "^\\d{4}-\\d{2}-\\d{2}T\\d{2}:\\d{2}$"), TRUE,
    # Date + hour:minute:second (full datetime, not partial)
    ifelse(str_detect(x, "^\\d{4}-\\d{2}-\\d{2}T\\d{2}:\\d{2}:\\d{2}$"), FALSE,
    # Any other format treated as NA
    NA
    )))))
  )
}

# Apply function
result_df <- dtc_df %>%
  mutate(result = is_partial_dtc(dtc)) %>%
  select(dtc, result)

# Write output
write_csv(result_df, "outputs/result.csv", na = "")
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"dtc","result"
"2020",TRUE
"2020-06",TRUE
"2020-06-15",FALSE
"2020-06-15T12:30",TRUE
"2020-06-15T12:30:00",FALSE
NA,FALSE
```

### LLM Output
#### `result.csv`


```csv
dtc,result
2020,TRUE
2020-06,TRUE
2020-06-15,TRUE
2020-06-15T12:30,TRUE
2020-06-15T12:30:00,FALSE
,
```
