# Case 075: pharmaverse/admiral/convert_dtc_to_dtm

## Metadata
- Task ID: `pharmaverse/admiral/convert_dtc_to_dtm`
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
Write an R script to perform convert dtc to dtm using the admiral clinical task contract.

Input: dtc.tsv
Output: result.csv


Required columns for result.csv: dtc, result
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `dtc.tsv`


```text
dtc
2020-01-01T12:30:00
2020-06
2020
NA
```

## Code
### Ground Truth Code

```r
suppressPackageStartupMessages(library(admiral))

dtc <- as.character(read.delim(file.path("inputs", "dtc.tsv"), check.names = FALSE, stringsAsFactors = FALSE, blank.lines.skip = FALSE)$dtc)
complete_dtm <- grepl("^[0-9]{4}-[0-9]{2}-[0-9]{2}T[0-9]{2}:[0-9]{2}:[0-9]{2}$", dtc)
result <- as.POSIXct(ifelse(complete_dtm, sub("T", " ", dtc), NA_character_), tz = "UTC")

dir.create("outputs", showWarnings = FALSE)
write.csv(data.frame(dtc = dtc, result = result), file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

library(readr)
library(dplyr)
library(stringr)

# Create outputs directory if it doesn't exist
if (!dir.exists("outputs")) {
  dir.create("outputs", recursive = TRUE)
}

# Read input
dtc <- read_tsv(
  file = file.path("inputs", "dtc.tsv"),
  col_types = cols(
    dtc = col_character()
  )
)

# Function to convert DTC -> DTM per CDISC/ADaM rules
convert_dtc_to_dtm <- function(x) {
  if (is.na(x) || str_trim(x) == "") return(NA_character_)

  # Keep original
  orig <- x

  # Split date/time
  parts <- str_split_fixed(orig, "T", 2)
  date_part <- parts[, 1]
  time_part <- ifelse(parts[, 2] == "", NA_character_, parts[, 2])

  # Derive full date (YYYY-MM-DD)
  date_len <- nchar(date_part)
  if (date_len >= 10) {
    full_date <- substr(date_part, 1, 10)
  } else if (date_len == 7) {
    # YYYY-MM
    full_date <- paste0(date_part, "-01")
  } else if (date_len == 4) {
    # YYYY
    full_date <- paste0(date_part, "-01-01")
  } else {
    return(NA_character_)
  }

  # Derive full time (HH:MM:SS)
  if (is.na(time_part) || time_part == "") {
    full_time <- "00:00:00"
  } else {
    # Handle time that might be HH, HH:MM, HH:MM:SS
    t_split <- str_split_fixed(time_part, ":", 3)
    h <- t_split[, 1]
    m <- ifelse(t_split[, 2] == "", "00", t_split[, 2])
    s <- ifelse(t_split[, 3] == "", "00", t_split[, 3])

    # Truncate to two digits each
    h <- str_sub(h, 1, 2)
    m <- str_sub(m, 1, 2)
    s <- str_sub(s, 1, 2)

    full_time <- paste(h, m, s, sep = ":")
  }

  paste(full_date, full_time)
}

result <- dtc %>%
  mutate(
    result = vapply(dtc, convert_dtc_to_dtm, character(1))
  ) %>%
  select(dtc, result)

# Write output
write_csv(result, file.path("outputs", "result.csv"), na = "")
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"dtc","result"
"2020-01-01T12:30:00",2020-01-01 12:30:00
"2020-06",NA
"2020",NA
NA,NA
```

### LLM Output
#### `result.csv`


```csv
dtc,result
2020-01-01T12:30:00,2020-01-01 12:30:00
2020-06,2020-06-01 00:00:00
2020,2020-01-01 00:00:00
,
```
