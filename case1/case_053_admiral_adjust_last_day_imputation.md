# Case 053: pharmaverse/admiral/adjust_last_day_imputation

## Metadata
- Task ID: `pharmaverse/admiral/adjust_last_day_imputation`
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
Write an R script to perform adjust last day imputation using the admiral clinical task contract.

Input: imputed_dtc.tsv, partial.tsv
Output: result.csv


Required columns for result.csv: imputed_dtc, year, month, day, result
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `imputed_dtc.tsv`


```text
imputed_dtc
2020-02-01
2020-02-15
2021-04-01
```

#### `partial.tsv`


```text
year	month	day
2020	02	NA
2020	02	15
2021	04	NA
```

## Code
### Ground Truth Code

```r
suppressPackageStartupMessages(library(admiral))

imputed_dtc <- as.character(read.delim(file.path("inputs", "imputed_dtc.tsv"), check.names = FALSE, stringsAsFactors = FALSE)[[1]])
partial <- read.delim(file.path("inputs", "partial.tsv"), check.names = FALSE, stringsAsFactors = FALSE, na.strings = c("", "NA"))
last_day_one <- function(year, month) {
  first <- as.Date(sprintf("%04d-%02d-01", as.integer(year), as.integer(month)))
  seq(first, by = "month", length.out = 2)[2] - 1
}
last_days <- as.Date(mapply(last_day_one, partial$year, partial$month), origin = "1970-01-01")
result <- ifelse(is.na(partial$day), as.character(last_days), imputed_dtc)

dir.create("outputs", showWarnings = FALSE)
write.csv(data.frame(imputed_dtc = imputed_dtc, year = partial$year, month = partial$month, day = partial$day, result = result), file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

suppressPackageStartupMessages({
  library(readr)
  library(dplyr)
  library(lubridate)
})

# Create outputs directory if it doesn't exist
if (!dir.exists("outputs")) dir.create("outputs", recursive = TRUE)

# Read inputs
imputed_dtc <- read_tsv(
  file = file.path("inputs", "imputed_dtc.tsv"),
  col_types = cols(
    imputed_dtc = col_character()
  )
)

partial <- read_tsv(
  file = file.path("inputs", "partial.tsv"),
  col_types = cols(
    year  = col_integer(),
    month = col_integer(),
    day   = col_integer()
  )
)

# Combine inputs row-wise (assumes aligned rows)
df <- bind_cols(imputed_dtc, partial)

# Perform last day imputation:
# If day is NA, set it to last day of month; otherwise keep as is.
df_imputed <- df %>%
  mutate(
    imputed_day = if_else(
      is.na(day),
      day(ceiling_date(ymd(sprintf("%04d-%02d-01", year, month)), "month") - days(1)),
      day
    ),
    result = if_else(
      is.na(day),
      format(ymd(sprintf("%04d-%02d-%02d", year, month, imputed_day)), "%Y-%m-%d"),
      as.character(ymd(imputed_dtc))
    )
  ) %>%
  transmute(
    imputed_dtc,
    year,
    month,
    day,
    result
  )

# Write output
write_csv(df_imputed, file.path("outputs", "result.csv"), na = "")
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"imputed_dtc","year","month","day","result"
"2020-02-01",2020,2,NA,"2020-02-29"
"2020-02-15",2020,2,15,"2020-02-15"
"2021-04-01",2021,4,NA,"2021-04-30"
```

### LLM Output
#### `result.csv`


```csv
imputed_dtc,year,month,day,result
2020-02-01,2020,2,,2020-02-29
2020-02-15,2020,2,15,2020-02-15
2021-04-01,2021,4,,2021-04-30
```
