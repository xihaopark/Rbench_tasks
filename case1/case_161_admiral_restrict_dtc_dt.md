# Case 161: pharmaverse/admiral/restrict_imputed_dtc_dt

## Metadata
- Task ID: `pharmaverse/admiral/restrict_imputed_dtc_dt`
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
Write an R script to perform restrict imputed dtc dt using the admiral clinical task contract.

Input: imputed_dtc.tsv
Output: result.csv


Required columns for result.csv: dtc, imputed_dtc, min_date, max_date, result
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `imputed_dtc.tsv`


```text
dtc	imputed_dtc	min_date	max_date
2020-01	2020-01-15	2020-01-20	NA
2020-02	2020-02-15	NA	2020-02-10
2020-03	2020-03-15	2020-03-01	2020-03-31
```

## Code
### Ground Truth Code

```r
data <- read.delim(file.path("inputs", "imputed_dtc.tsv"), check.names = FALSE, stringsAsFactors = FALSE)
to_date <- function(x) {
  x[x == "NA" | x == ""] <- NA_character_
  as.Date(x)
}
imputed <- to_date(data$imputed_dtc)
min_date <- to_date(data$min_date)
max_date <- to_date(data$max_date)
result <- imputed
result[!is.na(min_date) & result < min_date] <- min_date[!is.na(min_date) & result < min_date]
result[!is.na(max_date) & result > max_date] <- max_date[!is.na(max_date) & result > max_date]
result_df <- data.frame(data, result = as.character(result), stringsAsFactors = FALSE)

dir.create("outputs", showWarnings = FALSE)
write.csv(result_df, file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

library(dplyr)
library(readr)
library(lubridate)

# Create output directory if it does not exist
if (!dir.exists("outputs")) {
  dir.create("outputs", recursive = TRUE)
}

# Read input
imputed <- read_tsv(
  file = file.path("inputs", "imputed_dtc.tsv"),
  col_types = cols(
    dtc = col_character(),
    imputed_dtc = col_character(),
    min_date = col_character(),
    max_date = col_character()
  )
)

# Helper to safely parse dates, returning NA if parsing fails
parse_safe <- function(x) {
  ifelse(is.na(x) | x == "" | x == "NA",
         NA_Date_,
         ymd(x, quiet = TRUE)
  )
}

result <- imputed %>%
  mutate(
    imputed_dtc_date = parse_safe(imputed_dtc),
    min_date_date    = parse_safe(min_date),
    max_date_date    = parse_safe(max_date),
    result = case_when(
      # both bounds available: check inclusive within range
      !is.na(min_date_date) & !is.na(max_date_date) &
        !is.na(imputed_dtc_date) &
        imputed_dtc_date >= min_date_date &
        imputed_dtc_date <= max_date_date ~ "Y",

      # only min bound: imputed must be on/after min
      !is.na(min_date_date) & is.na(max_date_date) &
        !is.na(imputed_dtc_date) &
        imputed_dtc_date >= min_date_date ~ "Y",

      # only max bound: imputed must be on/before max
      is.na(min_date_date) & !is.na(max_date_date) &
        !is.na(imputed_dtc_date) &
        imputed_dtc_date <= max_date_date ~ "Y",

      TRUE ~ "N"
    )
  ) %>%
  select(dtc, imputed_dtc, min_date, max_date, result)

# Write output
write_csv(result, file.path("outputs", "result.csv"), na = "")
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"dtc","imputed_dtc","min_date","max_date","result"
"2020-01","2020-01-15","2020-01-20",NA,"2020-01-20"
"2020-02","2020-02-15",NA,"2020-02-10","2020-02-10"
"2020-03","2020-03-15","2020-03-01","2020-03-31","2020-03-15"
```

### LLM Output
#### `result.csv`


```csv
dtc,imputed_dtc,min_date,max_date,result
2020-01,2020-01-15,2020-01-20,,N
2020-02,2020-02-15,,2020-02-10,N
2020-03,2020-03-15,2020-03-01,2020-03-31,Y
```
