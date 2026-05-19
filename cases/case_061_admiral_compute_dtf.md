# Case 061: pharmaverse/admiral/compute_dtf

## Metadata
- Task ID: `pharmaverse/admiral/compute_dtf`
- Package: `admiral`
- Model: `openai/gpt-5.1`
- Status: `FAIL`
- Failure stage: `value_mismatch`

## Pass/Error
```text
The generated output files were produced, but one or more values differed from the
ground truth.

Main signal:
result.csv: Value mismatch in column: result.
```

## Prompt
```text
Write an R script to perform compute dtf using the admiral clinical task contract.

Input: dt.tsv, dtc.tsv
Output: result.csv


Required columns for result.csv: dtc, dt, result
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `dt.tsv`


```text
dt
2020-01-01
2020-06-01
2020-01-01
NA
```

#### `dtc.tsv`


```text
dtc
2020-01-01
2020-06
2020
NA
```

## Code
### Ground Truth Code

```r
suppressPackageStartupMessages(library(admiral))

dtc <- as.character(read.delim(file.path("inputs", "dtc.tsv"), check.names = FALSE, stringsAsFactors = FALSE, blank.lines.skip = FALSE)$dtc)
dt_raw <- read.delim(file.path("inputs", "dt.tsv"), check.names = FALSE, stringsAsFactors = FALSE, blank.lines.skip = FALSE)$dt
dt <- as.Date(dt_raw)
result <- admiral::compute_dtf(dtc = dtc, dt = dt)

dir.create("outputs", showWarnings = FALSE)
write.csv(data.frame(dtc = dtc, dt = dt, result = result), file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

suppressPackageStartupMessages({
  library(readr)
  library(dplyr)
  library(stringr)
  library(lubridate)
})

# Create outputs directory if it doesn't exist
if (!dir.exists("outputs")) {
  dir.create("outputs", recursive = TRUE)
}

# Read inputs
dt   <- read_tsv("inputs/dt.tsv", col_types = cols(.default = col_character()))
dtc  <- read_tsv("inputs/dtc.tsv", col_types = cols(.default = col_character()))

# Ensure same length
n <- max(nrow(dt), nrow(dtc))
dt  <- dt  %>% slice_head(n = n)
dtc <- dtc %>% slice_head(n = n)

# Parse dt as Date
dt_parsed <- ymd(dt$dt)

# Function to parse dtc as Date (ADaM-like partial date handling)
parse_dtc <- function(x) {
  x <- str_trim(x)
  ifelse(
    is.na(x) | x == "",
    NA_Date_,
    ifelse(
      str_detect(x, "^\\d{4}-\\d{2}-\\d{2}$"),
      ymd(x),
      ifelse(
        str_detect(x, "^\\d{4}-\\d{2}$"),
        ymd(paste0(x, "-01")),
        ifelse(
          str_detect(x, "^\\d{4}$"),
          ymd(paste0(x, "-01-01")),
          NA_Date_
        )
      )
    )
  )
}

dtc_parsed <- parse_dtc(dtc$dtc)

# Compute dtf: difference in days (dt - dtc) per ADaM-style
# If either is NA, result is NA
result <- ifelse(is.na(dt_parsed) | is.na(dtc_parsed),
                 NA_integer_,
                 as.integer(dt_parsed - dtc_parsed))

# Assemble result
out <- tibble(
  dtc    = dtc$dtc,
  dt     = dt$dt,
  result = result
)

# Write output
write_csv(out, "outputs/result.csv", na = "")
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"dtc","dt","result"
"2020-01-01",2020-01-01,NA
"2020-06",2020-06-01,"D"
"2020",2020-01-01,"M"
NA,NA,NA
```

### LLM Output
#### `result.csv`


```csv
dtc,dt,result
2020-01-01,2020-01-01,0
2020-06,2020-06-01,0
2020,2020-01-01,0
,,
```
