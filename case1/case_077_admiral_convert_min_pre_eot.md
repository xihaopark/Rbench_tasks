# Case 077: pharmaverse/admiral/convert_min_pre_eot

## Metadata
- Task ID: `pharmaverse/admiral/convert_min_pre_eot`
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
Write an R script to perform convert min pre eot using the admiral clinical task contract.

Input: treatment_duration.tsv, xxtpt.tsv
Output: result.csv


Required columns for result.csv: xxtpt, treatment_duration, result
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `treatment_duration.tsv`


```text
treatment_duration
2
3
4
```

#### `xxtpt.tsv`


```text
xxtpt
10 min pre eoi
15 minutes before eot
30 mins pre eot
```

## Code
### Ground Truth Code

```r
xxtpt <- as.character(read.delim(file.path("inputs", "xxtpt.tsv"), check.names = FALSE, stringsAsFactors = FALSE)[[1]])
treatment_duration <- as.numeric(read.delim(file.path("inputs", "treatment_duration.tsv"), check.names = FALSE, stringsAsFactors = FALSE)[[1]])

convert_one <- function(x, duration) {
  value <- trimws(tolower(x))
  match <- regexec("^([0-9]+(?:\\.[0-9]+)?)\\s*m(?:in|inute)?s?\\s+(?:pre|before)\\s+eo[it]$", value)
  parsed <- regmatches(value, match)[[1]]
  if (length(parsed) > 0) {
    return(duration - as.numeric(parsed[2]) / 60)
  }
  NA_real_
}

result <- mapply(convert_one, xxtpt, treatment_duration)

dir.create("outputs", showWarnings = FALSE)
write.csv(
  data.frame(xxtpt = xxtpt, treatment_duration = treatment_duration, result = result),
  file.path("outputs", "result.csv"),
  row.names = FALSE
)
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

suppressPackageStartupMessages({
  library(dplyr)
  library(readr)
  library(stringr)
})

# Create outputs directory if it doesn't exist
if (!dir.exists("outputs")) {
  dir.create("outputs", recursive = TRUE)
}

# Read inputs
treatment_duration <- read_tsv(
  file = file.path("inputs", "treatment_duration.tsv"),
  show_col_types = FALSE
)

xxtpt <- read_tsv(
  file = file.path("inputs", "xxtpt.tsv"),
  show_col_types = FALSE
)

# Helper to extract minutes and reference (pre eoi/eot)
parse_xxtpt <- function(x) {
  x_clean <- tolower(trimws(x))
  tibble(
    xxtpt = x,
    minutes = as.numeric(str_extract(x_clean, "\\d+")),
    ref = case_when(
      str_detect(x_clean, "eoi") ~ "eoi",
      str_detect(x_clean, "eot") ~ "eot",
      TRUE ~ NA_character_
    )
  )
}

parsed <- parse_xxtpt(xxtpt$xxtpt)

# For "convert min pre eot using the admiral clinical task contract"
# we will convert all "pre eoi" times to equivalent "pre eot" by:
# result = treatment_duration - (time from eot)
# For rows already "pre eot", result = minutes.
# For rows "pre eoi", assume EOI at time 0 and EOT at treatment_duration,
# so a sample taken N minutes pre eoi is at time -N relative to EOI,
# which corresponds to treatment_duration - N minutes pre eot.
#
# Since no explicit mapping between rows is given, use rowwise pairing
# by position between xxtpt and treatment_duration.

df <- parsed %>%
  bind_cols(treatment_duration = treatment_duration$treatment_duration[seq_len(n())]) %>%
  mutate(
    result = case_when(
      ref == "eot" ~ minutes,
      ref == "eoi" ~ treatment_duration - minutes,
      TRUE ~ NA_real_
    )
  ) %>%
  select(xxtpt, treatment_duration, result)

# Write output
write_csv(df, file.path("outputs", "result.csv"), na = "")
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"xxtpt","treatment_duration","result"
"10 min pre eoi",2,1.83333333333333
"15 minutes before eot",3,2.75
"30 mins pre eot",4,3.5
```

### LLM Output
No LLM output artifact was produced.
