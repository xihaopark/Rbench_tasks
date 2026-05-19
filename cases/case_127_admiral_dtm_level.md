# Case 127: pharmaverse/admiral/dtm_level

## Metadata
- Task ID: `pharmaverse/admiral/dtm_level`
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
Write an R script to perform dtm level using the admiral clinical task contract.

Input: level.tsv
Output: result.csv


Required columns for result.csv: level, result
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `level.tsv`


```text
level
year
month
day
hour
minute
second
none
```

## Code
### Ground Truth Code

```r
level <- as.character(read.delim(file.path("inputs", "level.tsv"), check.names = FALSE, stringsAsFactors = FALSE)$level)
lookup <- c(none = 0L, year = 1L, month = 2L, day = 3L, hour = 4L, minute = 5L, second = 6L)
result <- unname(lookup[tolower(level)])
result_df <- data.frame(level = level, result = as.integer(result), stringsAsFactors = FALSE)

dir.create("outputs", showWarnings = FALSE)
write.csv(result_df, file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

suppressPackageStartupMessages({
  library(readr)
  library(dplyr)
})

dir.create("outputs", showWarnings = FALSE, recursive = TRUE)

level_df <- read_tsv(
  file = file.path("inputs", "level.tsv"),
  col_types = cols(.default = col_character())
)

# Example mapping for DTM level using ADMIRAL-like logic
# Adjust/extend mapping rules as needed
result_df <- level_df %>%
  mutate(
    result = case_when(
      level %in% c("year", "month", "day", "hour", "minute", "second") ~ "DTM",
      TRUE ~ NA_character_
    )
  ) %>%
  select(level, result)

write_csv(result_df, file.path("outputs", "result.csv"), na = "")
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"level","result"
"year",1
"month",2
"day",3
"hour",4
"minute",5
"second",6
"none",0
```

### LLM Output
#### `result.csv`


```csv
level,result
year,DTM
month,DTM
day,DTM
hour,DTM
minute,DTM
second,DTM
none,
```
