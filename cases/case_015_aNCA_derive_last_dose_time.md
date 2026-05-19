# Case 015: pharmaverse/aNCA/derive_last_dose_time

## Metadata
- Task ID: `pharmaverse/aNCA/derive_last_dose_time`
- Package: `aNCA`
- Model: `openai/gpt-5.1`
- Status: `PASS`
- Failure stage: `pass`

## Pass/Error
```text
pass
```

## Prompt
```text
Write an R script to perform derive last dose time using the aNCA clinical task contract.

Input: conc_time_col.tsv, pknca_data.tsv
Output: result.csv


Required columns for result.csv: subject, time, conc
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `conc_time_col.tsv`


```text
conc_time_col
time
```

#### `pknca_data.tsv`


```text
subject	time	conc
101	0	0.0
101	1	10.5
101	2	8.2
102	0	0.0
102	1	12.1
```

## Code
### Ground Truth Code

```r
read_tsv <- function(name) {
  path <- file.path("inputs", name)
  if (!file.exists(path)) {
    stop(name, " is required")
  }
  read.delim(path, check.names = FALSE, stringsAsFactors = FALSE)
}

first_column <- function(df) {
  if (ncol(df) == 0) {
    return(character())
  }
  df[[1]]
}

scalar_numeric <- function(df) {
  values <- suppressWarnings(as.numeric(first_column(df)))
  values <- values[!is.na(values)]
  if (length(values) == 0) NA_real_ else values[[1]]
}

write_result <- function(result) {
  dir.create("outputs", showWarnings = FALSE, recursive = TRUE)
  write.csv(result, file.path("outputs", "result.csv"), row.names = FALSE, na = "NA")
}

pknca_data <- read_tsv("pknca_data.tsv")
write_result(pknca_data)
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

suppressPackageStartupMessages({
  library(dplyr)
  library(readr)
})

# Ensure outputs directory exists
if (!dir.exists("outputs")) {
  dir.create("outputs", recursive = TRUE)
}

# Read inputs
conc_time_col <- read_tsv(
  file = file.path("inputs", "conc_time_col.tsv"),
  col_types = cols()
)

pknca_data <- read_tsv(
  file = file.path("inputs", "pknca_data.tsv"),
  col_types = cols()
)

# Identify concentration time column name
time_col <- conc_time_col$conc_time_col[1]

# Rename columns in pknca_data to required output names
# subject, time, conc
result <- pknca_data %>%
  rename(
    subject = !!sym("subject"),
    time    = !!sym(time_col),
    conc    = !!sym("conc")
  ) %>%
  select(subject, time, conc)

# Write output
write_csv(result, file.path("outputs", "result.csv"), na = "")
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"subject","time","conc"
101,0,0
101,1,10.5
101,2,8.2
102,0,0
102,1,12.1
```

### LLM Output
#### `result.csv`


```csv
subject,time,conc
101,0,0
101,1,10.5
101,2,8.2
102,0,0
102,1,12.1
```
