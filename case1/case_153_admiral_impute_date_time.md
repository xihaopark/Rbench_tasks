# Case 153: pharmaverse/admiral/impute_date_time

## Metadata
- Task ID: `pharmaverse/admiral/impute_date_time`
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
Write an R script to perform impute date time using the admiral clinical task contract.

Input: partial.tsv, target.tsv
Output: result.csv


Required columns for result.csv: partial, year, month, day, hour, minute, second, date_imputation, time_imputation
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `partial.tsv`


```text
partial
2020
2020-05
2020-05-20T14
```

#### `target.tsv`


```text
date_imputation	time_imputation
mid	first
mid	first
mid	first
```

## Code
### Ground Truth Code

```r
partial <- as.character(read.delim(file.path("inputs", "partial.tsv"), check.names = FALSE, stringsAsFactors = FALSE)$partial)
target <- read.delim(file.path("inputs", "target.tsv"), check.names = FALSE, stringsAsFactors = FALSE)
year <- substr(partial, 1, 4)
month <- ifelse(nchar(partial) >= 7, substr(partial, 6, 7), "06")
day <- ifelse(nchar(partial) >= 10, substr(partial, 9, 10), "15")
hour <- ifelse(nchar(partial) >= 13, substr(partial, 12, 13), "00")
minute <- ifelse(nchar(partial) >= 16, substr(partial, 15, 16), "00")
second <- ifelse(nchar(partial) >= 19, substr(partial, 18, 19), "00")
result_df <- data.frame(partial = partial, year = year, month = month, day = day, hour = hour, minute = minute, second = second, stringsAsFactors = FALSE)
result_df$date_imputation <- target$date_imputation
result_df$time_imputation <- target$time_imputation
result_df <- result_df[, c("partial", "year", "month", "day", "hour", "minute", "second", "date_imputation", "time_imputation")]

dir.create("outputs", showWarnings = FALSE)
write.csv(result_df, file.path("outputs", "result.csv"), row.names = FALSE)
saveRDS(result_df, file.path("outputs", "result.rds"))
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

suppressPackageStartupMessages({
  library(readr)
  library(dplyr)
  library(stringr)
})

dir.create("outputs", showWarnings = FALSE, recursive = TRUE)

partial_df <- read_tsv(
  file = file.path("inputs", "partial.tsv"),
  col_types = cols(
    partial = col_character()
  )
)

target_df <- read_tsv(
  file = file.path("inputs", "target.tsv"),
  col_types = cols(
    date_imputation = col_character(),
    time_imputation = col_character()
  )
)

df <- bind_cols(partial_df, target_df)

parse_partial <- function(x) {
  year <- NA_integer_
  month <- NA_integer_
  day <- NA_integer_
  hour <- NA_integer_
  minute <- NA_integer_
  second <- NA_integer_

  if (str_detect(x, "T")) {
    dt_parts <- str_split_fixed(x, "T", 2)
    d_part <- dt_parts[, 1]
    t_part <- dt_parts[, 2]
  } else {
    d_part <- x
    t_part <- NA_character_
  }

  d_fields <- str_split(d_part, "-", simplify = TRUE)
  if (ncol(d_fields) >= 1 && d_fields[1] != "") {
    year <- as.integer(d_fields[1])
  }
  if (ncol(d_fields) >= 2 && d_fields[2] != "") {
    month <- as.integer(d_fields[2])
  }
  if (ncol(d_fields) >= 3 && d_fields[3] != "") {
    day <- as.integer(d_fields[3])
  }

  if (!is.na(t_part)) {
    t_fields <- str_split(t_part, ":", simplify = TRUE)
    if (ncol(t_fields) >= 1 && t_fields[1] != "") {
      hour <- as.integer(t_fields[1])
    }
    if (ncol(t_fields) >= 2 && t_fields[2] != "") {
      minute <- as.integer(t_fields[2])
    }
    if (ncol(t_fields) >= 3 && t_fields[3] != "") {
      second <- as.integer(t_fields[3])
    }
  }

  tibble(
    year = year,
    month = month,
    day = day,
    hour = hour,
    minute = minute,
    second = second
  )
}

parsed <- bind_rows(lapply(df$partial, parse_partial))

apply_date_imputation <- function(year, month, day, date_imp) {
  if (is.na(year)) return(list(year = NA_integer_, month = NA_integer_, day = NA_integer_))

  if (is.na(month)) {
    if (identical(date_imp, "mid")) {
      month <- 7L
    }
  }

  if (is.na(day)) {
    if (identical(date_imp, "mid")) {
      if (!is.na(month)) {
        days_in_month <- as.integer(format(as.Date(sprintf("%04d-%02d-01", year, month)) + months(1) - 1, "%d"))
        day <- as.integer(ceiling(days_in_month / 2))
      }
    }
  }

  list(year = year, month = month, day = day)
}

apply_time_imputation <- function(hour, minute, second, time_imp) {
  if (identical(time_imp, "first")) {
    if (is.na(hour)) hour <- 0L
    if (is.na(minute)) minute <- 0L
    if (is.na(second)) second <- 0L
  }
  list(hour = hour, minute = minute, second = second)
}

imputed <- pmap_dfr(
  list(
    parsed$year,
    parsed$month,
    parsed$day,
    parsed$hour,
    parsed$minute,
    parsed$second,
    df$date_imputation,
    df$time_imputation
  ),
  function(year, month, day, hour, minute, second, date_imp, time_imp) {
    d <- apply_date_imputation(year, month, day, date_imp)
    t <- apply_time_imputation(hour, minute, second, time_imp)
    tibble(
      year = d$year,
      month = d$month,
      day = d$day,
      hour = t$hour,
      minute = t$minute,
      second = t$second
    )
  }
)

result <- bind_cols(
  df %>% select(partial, date_imputation, time_imputation),
  imputed %>% select(year, month, day, hour, minute, second)
) %>%
  select(partial, year, month, day, hour, minute, second, date_imputation, time_imputation)

write_csv(result, file.path("outputs", "result.csv"), na = "")
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"partial","year","month","day","hour","minute","second","date_imputation","time_imputation"
"2020","2020","06","15","00","00","00","mid","first"
"2020-05","2020","05","15","00","00","00","mid","first"
"2020-05-20T14","2020","05","20","14","00","00","mid","first"
```

### LLM Output
No LLM output artifact was produced.
