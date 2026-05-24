# Case 011: pharmaverse/aNCA/convert_to_iso8601_duration

## Metadata
- Task ID: `pharmaverse/aNCA/convert_to_iso8601_duration`
- Package: `aNCA`
- Model: `openai/gpt-5.1`
- Status: `FAIL`
- Failure stage: `value_mismatch`

## Pass/Error
```text
The generated output files were produced, but one or more values differed from the
ground truth.

Main signal:
result.csv: Numeric missingness mismatch in column: value.
```

## Prompt
```text
Write an R script to perform convert to ISO 8601 duration using the aNCA clinical task contract.

Input: unit.tsv, value.tsv
Output: result.csv


Required columns for result.csv: value, unit, result
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `unit.tsv`


```text
time_unit
minutes
```

#### `value.tsv`


```text
set_values_to
exprs(AVAL = mean(AVAL, na.rm=TRUE))
```

## Code
### Ground Truth Code

```r
read_table <- function(name) {
  read.delim(file.path("inputs", name), check.names = FALSE, stringsAsFactors = FALSE)
}

first_value <- function(df, default = "") {
  if (nrow(df) == 0 || ncol(df) == 0 || is.na(df[[1]][[1]])) return(default)
  as.character(df[[1]][[1]])
}

normalize_unit <- function(unit) {
  key <- tolower(trimws(unit))
  if (key %in% c("y", "yr", "year", "years")) return("y")
  if (key %in% c("mo", "mon", "month", "months")) return("mo")
  if (key %in% c("w", "week", "weeks")) return("w")
  if (key %in% c("h", "hr", "hour", "hours")) return("h")
  if (key %in% c("m", "min", "minute", "minutes")) return("m")
  if (key %in% c("s", "sec", "second", "seconds")) return("s")
  "d"
}

format_duration <- function(value, unit) {
  suffix <- switch(unit, y = "Y", mo = "M", w = "W", d = "D", h = "H", m = "M", s = "S")
  if (unit %in% c("h", "m", "s")) paste0("PT", value, suffix) else paste0("P", value, suffix)
}

value <- suppressWarnings(as.numeric(first_value(read_table("value.tsv"), "0")))
if (is.na(value)) value <- 0
unit <- normalize_unit(first_value(read_table("unit.tsv"), "d"))
result <- data.frame(value = value, unit = ifelse(unit == "mo", "mo", unit), result = format_duration(value, unit), stringsAsFactors = FALSE)

dir.create("outputs", showWarnings = FALSE, recursive = TRUE)
write.csv(result, file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

suppressPackageStartupMessages({
  library(readr)
  library(dplyr)
})

dir.create("outputs", showWarnings = FALSE, recursive = TRUE)

unit_path  <- file.path("inputs", "unit.tsv")
value_path <- file.path("inputs", "value.tsv")

unit_df <- read_tsv(unit_path, show_col_types = FALSE)
value_df <- read_tsv(value_path, show_col_types = FALSE)

unit_raw  <- unit_df$time_unit[1]
value_raw <- value_df$set_values_to[1]

unit_map <- list(
  seconds = "PT1S",
  second  = "PT1S",
  sec     = "PT1S",
  s       = "PT1S",
  minutes = "PT1M",
  minute  = "PT1M",
  min     = "PT1M",
  hours   = "PT1H",
  hour    = "PT1H",
  h       = "PT1H",
  days    = "P1D",
  day     = "P1D",
  d       = "P1D",
  weeks   = "P1W",
  week    = "P1W",
  w       = "P1W"
)

key <- tolower(trimws(unit_raw))
iso_duration <- if (!is.null(unit_map[[key]])) unit_map[[key]] else NA_character_

result_df <- tibble(
  value  = value_raw,
  unit   = unit_raw,
  result = iso_duration
)

write_csv(result_df, file.path("outputs", "result.csv"), na = "")
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"value","unit","result"
0,"m","PT0M"
```

### LLM Output
#### `result.csv`


```csv
value,unit,result
"exprs(AVAL = mean(AVAL, na.rm=TRUE))",minutes,PT1M
```
