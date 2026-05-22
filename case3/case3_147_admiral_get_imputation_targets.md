# Case 147: pharmaverse/admiral/get_imputation_targets

## Metadata
- Task ID: `pharmaverse/admiral/get_imputation_targets`
- Package: `admiral`
- Model: `openai/gpt-5.5`
- Status: `NO_OUTPUT`
- Failure stage: `execution_failure`
- Failure pattern: `package_api_hallucination`
- Failure pattern note: unused arguments (highest_imputation = highest_imputation, date_imputation = date_imputation)

## Pass/Error
```text
The generated code failed while running before valid outputs could be evaluated.

Main signal:
missing artifacts: result.csv.
```

## Prompt
```text
Write an R script for an R package task: perform get imputation targets using the admiral package. Use functions from the admiral package when suitable.

Output: result.csv

Computation: For partial date/time targets, year-only dates use month 06 and day 15; missing time components use 00.


Required columns for result.csv: partial, year, month, day, hour, minute, second
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `partial.tsv`

```text
partial
2020
2020-02
2020-02-20T14
```

## Code
### Ground Truth Code

```r
partial <- as.character(read.delim(file.path("inputs", "partial.tsv"), check.names = FALSE, stringsAsFactors = FALSE)$partial)
year <- substr(partial, 1, 4)
month <- ifelse(nchar(partial) >= 7, substr(partial, 6, 7), "06")
day <- ifelse(nchar(partial) >= 10, substr(partial, 9, 10), "15")
hour <- ifelse(nchar(partial) >= 13, substr(partial, 12, 13), "00")
minute <- ifelse(nchar(partial) >= 16, substr(partial, 15, 16), "00")
second <- ifelse(nchar(partial) >= 19, substr(partial, 18, 19), "00")
result_df <- data.frame(partial = partial, year = year, month = month, day = day, hour = hour, minute = minute, second = second, stringsAsFactors = FALSE)

dir.create("outputs", showWarnings = FALSE)
write.csv(result_df, file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
dir.create("outputs", showWarnings = FALSE, recursive = TRUE)

input <- read.delim(
  file = file.path("inputs", "partial.tsv"),
  stringsAsFactors = FALSE,
  na.strings = c("", "NA")
)

imputed <- admiral::impute_dtc_dtm(
  dtc = input$partial,
  highest_imputation = "M",
  date_imputation = "mid",
  time_imputation = "first"
)

result <- data.frame(
  partial = input$partial,
  year = format(imputed, "%Y"),
  month = format(imputed, "%m"),
  day = format(imputed, "%d"),
  hour = format(imputed, "%H"),
  minute = format(imputed, "%M"),
  second = format(imputed, "%S"),
  stringsAsFactors = FALSE
)

write.csv(
  result,
  file = file.path("outputs", "result.csv"),
  row.names = FALSE,
  quote = FALSE
)
```

## Output
### Ground Truth Output
#### `result.csv`

```csv
"partial","year","month","day","hour","minute","second"
"2020","2020","06","15","00","00","00"
"2020-02","2020","02","15","00","00","00"
"2020-02-20T14","2020","02","20","14","00","00"
```

### LLM Output
No LLM output artifact was produced.
