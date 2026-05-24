# Case 154: pharmaverse/admiral/impute_dtc_dt

## Metadata
- Task ID: `pharmaverse/admiral/impute_dtc_dt`
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
Write an R script to perform impute dtc dt using the admiral clinical task contract.

Input: date_imputation.tsv, dtc.tsv
Output: result.csv


Required columns for result.csv: dtc, result
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `date_imputation.tsv`


```text
date_imputation
first
```

#### `dtc.tsv`


```text
dtc
2020-06-15
2020-06
2020
NA
```

## Code
### Ground Truth Code

```r
dtc <- as.character(read.delim(file.path("inputs", "dtc.tsv"), check.names = FALSE, stringsAsFactors = FALSE)$dtc)
missing <- is.na(dtc) | toupper(dtc) == "NA" | dtc == ""
result_chr <- ifelse(grepl("^[0-9]{4}$", dtc), paste0(dtc, "-01-01"),
  ifelse(grepl("^[0-9]{4}-[0-9]{2}$", dtc), paste0(dtc, "-01"),
    ifelse(grepl("^[0-9]{4}-[0-9]{2}-[0-9]{2}$", dtc), dtc, NA_character_)))
result <- as.Date(ifelse(missing, NA_character_, result_chr))

dir.create("outputs", showWarnings = FALSE)
write.csv(data.frame(dtc = dtc, result = result), file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

suppressPackageStartupMessages({
  library(readr)
  library(dplyr)
  library(stringr)
})

# Create outputs directory if it does not exist
if (!dir.exists("outputs")) {
  dir.create("outputs", recursive = TRUE)
}

# Read inputs
date_imp <- read_tsv(
  file = file.path("inputs", "date_imputation.tsv"),
  show_col_types = FALSE
)

dtc_df <- read_tsv(
  file = file.path("inputs", "dtc.tsv"),
  show_col_types = FALSE
)

# Determine imputation rule from date_imputation
# Assume "first" -> impute missing month/day with "01"
imp_type <- tolower(date_imp$date_imputation[1])

impute_dtc <- function(dtc, imp_type = "first") {
  if (is.na(dtc) || dtc == "") return(NA_character_)

  # If already full date (YYYY-MM-DD), return as is
  if (str_detect(dtc, "^\\d{4}-\\d{2}-\\d{2}$")) {
    return(dtc)
  }

  # Year-month (YYYY-MM)
  if (str_detect(dtc, "^\\d{4}-\\d{2}$")) {
    if (imp_type == "first") {
      return(paste0(dtc, "-01"))
    } else {
      return(NA_character_)
    }
  }

  # Year only (YYYY)
  if (str_detect(dtc, "^\\d{4}$")) {
    if (imp_type == "first") {
      return(paste0(dtc, "-01-01"))
    } else {
      return(NA_character_)
    }
  }

  NA_character_
}

result_df <- dtc_df %>%
  mutate(
    result = vapply(dtc, impute_dtc, character(1), imp_type = imp_type)
  ) %>%
  select(dtc, result)

write_csv(result_df, file.path("outputs", "result.csv"), na = "")
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"dtc","result"
"2020-06-15",2020-06-15
"2020-06",2020-06-01
"2020",2020-01-01
NA,NA
```

### LLM Output
#### `result.csv`


```csv
dtc,result
2020-06-15,2020-06-15
2020-06,2020-06-01
2020,2020-01-01
,
```
