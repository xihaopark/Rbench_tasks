# Case 139: pharmaverse/admiral/format_imputed_dtc

## Metadata
- Task ID: `pharmaverse/admiral/format_imputed_dtc`
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
Write an R script to perform format imputed dtc using the admiral clinical task contract.

Input: imputed.tsv
Output: result.csv


Required columns for result.csv: imputed, result
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `imputed.tsv`


```text
imputed
2020-06-15
2020-06-01
2020-01-01
NA
```

## Code
### Ground Truth Code

```r
imputed <- as.character(read.delim(file.path("inputs", "imputed.tsv"), check.names = FALSE, stringsAsFactors = FALSE)$imputed)
dates <- as.Date(ifelse(toupper(imputed) == "NA", NA_character_, imputed))
result <- ifelse(is.na(dates), NA_character_, format(dates, "%Y-%m-%d"))

dir.create("outputs", showWarnings = FALSE)
write.csv(data.frame(imputed = imputed, result = result), file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

library(admiral)
library(dplyr)
library(readr)
library(stringr)

# Create outputs directory if it does not exist
if (!dir.exists("outputs")) {
  dir.create("outputs", recursive = TRUE)
}

# Read input
imputed_df <- read_tsv(
  file = file.path("inputs", "imputed.tsv"),
  col_types = cols(imputed = col_character())
)

# Assume imputed dates are already imputed and in ISO format (YYYY-MM-DD)
# Use admiral's format_imputed_dtc to ensure proper formatting behavior
result_df <- imputed_df %>%
  mutate(
    result = if_else(
      is.na(imputed) | imputed == "" | imputed == "NA",
      NA_character_,
      format_imputed_dtc(
        dtc = imputed,
        impute_year = "first",
        impute_month = "first",
        impute_day = "first",
        flag_imputation = FALSE
      )
    )
  ) %>%
  select(imputed, result)

# Write output
write_csv(result_df, file.path("outputs", "result.csv"), na = "")
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"imputed","result"
"2020-06-15","2020-06-15"
"2020-06-01","2020-06-01"
"2020-01-01","2020-01-01"
NA,NA
```

### LLM Output
No LLM output artifact was produced.
