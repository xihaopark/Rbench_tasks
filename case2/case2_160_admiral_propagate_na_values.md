# Case 160: pharmaverse/admiral/propagate_na_values

## Metadata
- Task ID: `pharmaverse/admiral/propagate_na_values`
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
Write an R script for an R package task: perform propagate na values using the admiral package. Use functions from the admiral package when suitable.

Output: result.csv

Computation: For ordered datetime components, keep components before the first missing component and blank all later components.


Required columns for result.csv: year, month, day, hour, minute, second
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `partial.tsv`

```text
year	month	day	hour	minute	second
2020	05	NA	10	30	45
2021	NA	12	09	15	00
2022	07	04	NA	20	30
```

## Code
### Ground Truth Code

```r
result_df <- read.delim(file.path("inputs", "partial.tsv"), check.names = FALSE, stringsAsFactors = FALSE, na.strings = c("NA", ""))
cols <- c("year", "month", "day", "hour", "minute", "second")
for (i in seq_len(nrow(result_df))) {
  missing_seen <- FALSE
  for (col in cols) {
    if (missing_seen || is.na(result_df[[col]][i]) || result_df[[col]][i] == "") {
      result_df[[col]][i] <- NA_character_
      missing_seen <- TRUE
    }
  }
}

dir.create("outputs", showWarnings = FALSE)
write.csv(result_df, file.path("outputs", "result.csv"), row.names = FALSE, na = "")
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

suppressPackageStartupMessages({
  library(admiral)
  library(dplyr)
  library(readr)
  library(stringr)
})

# Ensure outputs directory exists
if (!dir.exists("outputs")) {
  dir.create("outputs", recursive = TRUE)
}

# Read input
input_path <- file.path("inputs", "partial.tsv")
df <- read_tsv(
  input_path,
  show_col_types = FALSE,
  col_types = cols(
    year   = col_character(),
    month  = col_character(),
    day    = col_character(),
    hour   = col_character(),
    minute = col_character(),
    second = col_character()
  )
)

# Convert literal "NA" strings to NA and trim whitespace
df <- df %>%
  mutate(across(
    c(year, month, day, hour, minute, second),
    ~ na_if(str_trim(.x), "NA")
  ))

# Create a partial datetime character and propagate missing components
df_res <- df %>%
  derive_vars_dt(
    new_vars_prefix = "DT",
    dtc = paste(year, month, day, hour, minute, second, sep = "-")
  ) %>%
  mutate(
    dtc_propagated = convert_dtc_to_dt(dtc = DT, highest_imputation = "s"),
    # Split back into components
    year   = substr(dtc_propagated, 1, 4),
    month  = substr(dtc_propagated, 6, 7),
    day    = substr(dtc_propagated, 9, 10),
    hour   = substr(dtc_propagated, 12, 13),
    minute = substr(dtc_propagated, 15, 16),
    second = substr(dtc_propagated, 18, 19)
  ) %>%
  # Convert fully imputed components beyond first missing to blank
  rowwise() %>%
  mutate(
    comps = c(year, month, day, hour, minute, second),
    first_na_pos = match(TRUE, is.na(c_across(c(year, month, day, hour, minute, second)))),
    first_na_pos = ifelse(is.na(first_na_pos), length(comps) + 1, first_na_pos),
    comps = ifelse(seq_along(comps) >= first_na_pos, "", comps)
  ) %>%
  mutate(
    year   = comps[1],
    month  = comps[2],
    day    = comps[3],
    hour   = comps[4],
    minute = comps[5],
    second = comps[6]
  ) %>%
  ungroup() %>%
  select(year, month, day, hour, minute, second)

# Write output
output_path <- file.path("outputs", "result.csv")
write_csv(df_res, output_path, na = "")
```

## Output
### Ground Truth Output
#### `result.csv`

```csv
"year","month","day","hour","minute","second"
2020,"5",,,,
2021,,,,,
2022,"7","4",,,
```

### LLM Output
No LLM output artifact was produced.
