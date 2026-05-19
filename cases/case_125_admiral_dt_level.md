# Case 125: pharmaverse/admiral/dt_level

## Metadata
- Task ID: `pharmaverse/admiral/dt_level`
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
Write an R script to perform dt level using the admiral clinical task contract.

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
none
```

## Code
### Ground Truth Code

```r
level <- as.character(read.delim(file.path("inputs", "level.tsv"), check.names = FALSE, stringsAsFactors = FALSE)$level)
lookup <- c(none = 0L, year = 1L, month = 2L, day = 3L)
result <- unname(lookup[tolower(level)])
result_df <- data.frame(level = level, result = as.integer(result), stringsAsFactors = FALSE)

dir.create("outputs", showWarnings = FALSE)
write.csv(result_df, file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

suppressPackageStartupMessages({
  library(dplyr)
  library(readr)
})

# Create outputs directory if it does not exist
if (!dir.exists("outputs")) {
  dir.create("outputs", recursive = TRUE)
}

# Read input
level_df <- readr::read_tsv(
  file = file.path("inputs", "level.tsv"),
  col_types = cols(.default = col_character())
)

# Perform dt level derivation using admiral-like logic
# Here, we classify each level value into a simple result category
result_df <- level_df %>%
  mutate(
    result = dplyr::case_when(
      level %in% c("year", "month", "day") ~ "DATE_COMPONENT",
      level == "none" ~ "MISSING_COMPONENT",
      TRUE ~ "OTHER"
    )
  ) %>%
  select(level, result)

# Write output
readr::write_csv(result_df, file.path("outputs", "result.csv"))
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"level","result"
"year",1
"month",2
"day",3
"none",0
```

### LLM Output
#### `result.csv`


```csv
level,result
year,DATE_COMPONENT
month,DATE_COMPONENT
day,DATE_COMPONENT
none,MISSING_COMPONENT
```
