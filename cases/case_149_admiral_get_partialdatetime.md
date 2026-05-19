# Case 149: pharmaverse/admiral/get_partialdatetime

## Metadata
- Task ID: `pharmaverse/admiral/get_partialdatetime`
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
Write an R script to perform get partialdatetime using the admiral clinical task contract.

Input: dtc.tsv
Output: result.csv


Required columns for result.csv: dtc, year, month, day, hour, minute, second
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
missing <- is.na(dtc) | toupper(dtc) == "NA" | dtc == ""
year <- ifelse(missing, NA_integer_, as.integer(substr(dtc, 1, 4)))
month <- ifelse(nchar(dtc) >= 7 & !missing, as.integer(substr(dtc, 6, 7)), NA_integer_)
day <- ifelse(nchar(dtc) >= 10 & !missing, as.integer(substr(dtc, 9, 10)), NA_integer_)
hour <- ifelse(nchar(dtc) >= 13 & !missing, as.integer(substr(dtc, 12, 13)), NA_integer_)
minute <- ifelse(nchar(dtc) >= 16 & !missing, as.integer(substr(dtc, 15, 16)), NA_integer_)
second <- ifelse(nchar(dtc) >= 19 & !missing, as.integer(substr(dtc, 18, 19)), NA_integer_)
result_df <- data.frame(dtc = dtc, year = year, month = month, day = day, hour = hour, minute = minute, second = second, stringsAsFactors = FALSE)

dir.create("outputs", showWarnings = FALSE)
write.csv(result_df, file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

suppressPackageStartupMessages({
  library(dplyr)
  library(readr)
  library(stringr)
})

# create outputs dir if needed
if (!dir.exists("outputs")) dir.create("outputs", recursive = TRUE)

# read input
dt <- read_tsv(
  file = file.path("inputs", "dtc.tsv"),
  col_types = cols(.default = col_character())
)

# helper to split ISO 8601 partial datetime (YYYY, YYYY-MM, YYYY-MM-DD, etc.)
parse_partial_dtc <- function(dtc) {
  # separate date and time parts
  date_part  <- ifelse(str_detect(dtc, "T"), str_extract(dtc, "^[^T]+"), dtc)
  time_part  <- ifelse(str_detect(dtc, "T"), str_extract(dtc, "(?<=T).*$"), NA_character_)

  # split date
  ymd <- str_split_fixed(date_part, "-", 3)
  year  <- ifelse(ymd[, 1] == "", NA_character_, ymd[, 1])
  month <- ifelse(ymd[, 2] == "", NA_character_, ymd[, 2])
  day   <- ifelse(ymd[, 3] == "", NA_character_, ymd[, 3])

  # split time
  hms <- ifelse(
    is.na(time_part),
    NA_character_,
    time_part
  )
  hms_split <- str_split_fixed(coalesce(hms, ""), ":", 3)
  hour   <- ifelse(hms_split[, 1] == "" | is.na(hms), NA_character_, hms_split[, 1])
  minute <- ifelse(hms_split[, 2] == "" | is.na(hms), NA_character_, hms_split[, 2])
  second <- ifelse(hms_split[, 3] == "" | is.na(hms), NA_character_, hms_split[, 3])

  tibble(
    year   = year,
    month  = month,
    day    = day,
    hour   = hour,
    minute = minute,
    second = second
  )
}

result <- dt %>%
  mutate(parsed = parse_partial_dtc(dtc)) %>%
  bind_cols(select(., dtc), .$parsed) %>%
  select(dtc, year, month, day, hour, minute, second)

write_csv(result, file.path("outputs", "result.csv"), na = "")
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"dtc","year","month","day","hour","minute","second"
"2020",2020,NA,NA,NA,NA,NA
"2020-06",2020,6,NA,NA,NA,NA
"2020-06-15",2020,6,15,NA,NA,NA
"2020-06-15T12:30",2020,6,15,12,30,NA
"2020-06-15T12:30:00",2020,6,15,12,30,0
NA,NA,NA,NA,NA,NA,NA
```

### LLM Output
No LLM output artifact was produced.
