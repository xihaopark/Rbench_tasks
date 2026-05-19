# Case 162: pharmaverse/admiral/restrict_imputed_dtc_dtm

## Metadata
- Task ID: `pharmaverse/admiral/restrict_imputed_dtc_dtm`
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
Write an R script to perform restrict imputed dtc dtm using the admiral clinical task contract.

Input: imputed_dtc.tsv
Output: result.csv


Required columns for result.csv: dtc, imputed_dtc, min_datetime, max_datetime, result
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `imputed_dtc.tsv`


```text
dtc	imputed_dtc	min_datetime	max_datetime
2020-01-15	2020-01-15T12:00:00	2020-01-15T13:00:00	NA
2020-02-15	2020-02-15T12:00:00	NA	2020-02-15T10:00:00
2020-03-15	2020-03-15T12:00:00	2020-03-15T00:00:00	2020-03-15T23:59:59
```

## Code
### Ground Truth Code

```r
data <- read.delim(file.path("inputs", "imputed_dtc.tsv"), check.names = FALSE, stringsAsFactors = FALSE)
to_time <- function(x) {
  x[x == "NA" | x == ""] <- NA_character_
  as.POSIXct(gsub("T", " ", x), tz = "UTC")
}
fmt <- function(x) {
  ifelse(is.na(x), NA_character_, format(x, "%Y-%m-%dT%H:%M:%S", tz = "UTC"))
}
imputed <- to_time(data$imputed_dtc)
min_datetime <- to_time(data$min_datetime)
max_datetime <- to_time(data$max_datetime)
result <- imputed
result[!is.na(min_datetime) & result < min_datetime] <- min_datetime[!is.na(min_datetime) & result < min_datetime]
result[!is.na(max_datetime) & result > max_datetime] <- max_datetime[!is.na(max_datetime) & result > max_datetime]
result_df <- data.frame(data, result = fmt(result), stringsAsFactors = FALSE)

dir.create("outputs", showWarnings = FALSE)
write.csv(result_df, file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

suppressPackageStartupMessages({
  library(dplyr)
  library(readr)
  library(lubridate)
})

# Create outputs directory if it does not exist
if (!dir.exists("outputs")) {
  dir.create("outputs", recursive = TRUE)
}

# Read input
imputed <- read_tsv(
  file = file.path("inputs", "imputed_dtc.tsv"),
  col_types = cols(
    dtc = col_character(),
    imputed_dtc = col_character(),
    min_datetime = col_character(),
    max_datetime = col_character()
  )
)

# Helper to safely parse datetimes (NA if parsing fails)
parse_dt <- function(x) {
  parse_date_time(x, orders = c("Ymd HMS", "Ymd HM", "Ymd H", "Ymd"),
                  tz = "UTC", quiet = TRUE)
}

result_df <- imputed %>%
  mutate(
    dt_min = parse_dt(min_datetime),
    dt_max = parse_dt(max_datetime),
    imp_dt = parse_dt(imputed_dtc),
    result = case_when(
      !is.na(dt_min) & imp_dt < dt_min ~ "OUTSIDE_RANGE",
      !is.na(dt_max) & imp_dt > dt_max ~ "OUTSIDE_RANGE",
      TRUE ~ "WITHIN_RANGE"
    )
  ) %>%
  select(dtc, imputed_dtc, min_datetime, max_datetime, result)

# Write output
write_csv(result_df, file.path("outputs", "result.csv"), na = "")
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"dtc","imputed_dtc","min_datetime","max_datetime","result"
"2020-01-15","2020-01-15T12:00:00","2020-01-15T13:00:00",NA,"2020-01-15T13:00:00"
"2020-02-15","2020-02-15T12:00:00",NA,"2020-02-15T10:00:00","2020-02-15T10:00:00"
"2020-03-15","2020-03-15T12:00:00","2020-03-15T00:00:00","2020-03-15T23:59:59","2020-03-15T12:00:00"
```

### LLM Output
#### `result.csv`


```csv
dtc,imputed_dtc,min_datetime,max_datetime,result
2020-01-15,2020-01-15T12:00:00,2020-01-15T13:00:00,,OUTSIDE_RANGE
2020-02-15,2020-02-15T12:00:00,,2020-02-15T10:00:00,OUTSIDE_RANGE
2020-03-15,2020-03-15T12:00:00,2020-03-15T00:00:00,2020-03-15T23:59:59,WITHIN_RANGE
```
