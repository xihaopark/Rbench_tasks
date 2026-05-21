# Case 087: pharmaverse/admiral/convert_treatment_patterns

## Metadata
- Task ID: `pharmaverse/admiral/convert_treatment_patterns`
- Package: `admiral`
- Model: `openai/gpt-5.1`
- Status: `FAIL`
- Failure stage: `value_mismatch`

## Pass/Error
```text
The generated output files were produced, but one or more values differed from the ground truth.

Main signal:
result.csv: Numeric mismatch in column: result
```

## Prompt
```text
Write an R script for an R package task: perform convert treatment patterns using the admiral package. Use functions from the admiral package when suitable.

Output: result.csv

Computation: Map treatment-relative text patterns deterministically: treatment start/zero/pre-dose to 0, after-start to positive offsets, before-end/pre-EOT to negative offsets from end, and end/after-end patterns to the treatment end plus any offset.


Required columns for result.csv: xxtpt, treatment_duration, result
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `treatment_duration.tsv`

```text
treatment_duration
72
72
72
72
72
72
72
```

#### `xxtpt.tsv`

```text
xxtpt
START OF TREATMENT
PRE-DOSE
30 MIN AFTER START
2 HOURS AFTER START
END OF TREATMENT
4 HOURS BEFORE END
6 HOURS AFTER END
```

## Code
### Ground Truth Code

```r
xxtpt <- as.character(read.delim(file.path("inputs", "xxtpt.tsv"), check.names = FALSE, stringsAsFactors = FALSE)$xxtpt)
treatment_duration <- as.numeric(read.delim(file.path("inputs", "treatment_duration.tsv"), check.names = FALSE, stringsAsFactors = FALSE)$treatment_duration)

parse_hours <- function(label, duration) {
  text <- toupper(trimws(label))
  number <- suppressWarnings(as.numeric(sub(".*?([0-9]+(?:\\.[0-9]+)?).*", "\\1", text)))
  if (text == "START OF TREATMENT") {
    0
  } else if (text == "PRE-DOSE") {
    -1
  } else if (grepl("MIN AFTER START", text)) {
    number / 60
  } else if (grepl("HOURS? AFTER START", text)) {
    number
  } else if (text == "END OF TREATMENT") {
    duration
  } else if (grepl("HOURS? BEFORE END", text)) {
    duration - number
  } else if (grepl("HOURS? AFTER END", text)) {
    duration + number
  } else {
    NA_real_
  }
}

result <- mapply(parse_hours, xxtpt, treatment_duration)
result_df <- data.frame(xxtpt = xxtpt, treatment_duration = treatment_duration, result = as.numeric(result), stringsAsFactors = FALSE)

dir.create("outputs", showWarnings = FALSE)
write.csv(result_df, file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

library(admiral)
library(dplyr)
library(readr)
library(stringr)

# Create outputs directory if it doesn't exist
if (!dir.exists("outputs")) {
  dir.create("outputs", recursive = TRUE)
}

# Read inputs
treat_dur <- read_tsv(
  file = file.path("inputs", "treatment_duration.tsv"),
  show_col_types = FALSE
)

xxtpt <- read_tsv(
  file = file.path("inputs", "xxtpt.tsv"),
  show_col_types = FALSE
)

# Ensure same number of rows; recycle if needed
if (nrow(treat_dur) != nrow(xxtpt)) {
  n_rep <- ceiling(nrow(xxtpt) / nrow(treat_dur))
  treat_dur <- treat_dur[rep(seq_len(nrow(treat_dur)), length.out = nrow(xxtpt)), , drop = FALSE]
}

# Helper to convert time unit text to hours
parse_offset_to_hours <- function(text) {
  text <- str_trim(str_to_upper(text))
  num <- as.numeric(str_extract(text, "\\d+\\.?\\d*"))
  if (is.na(num)) num <- 0

  if (str_detect(text, "HOUR")) {
    num
  } else if (str_detect(text, "MIN")) {
    num / 60
  } else if (str_detect(text, "DAY")) {
    num * 24
  } else {
    0
  }
}

# Derive numeric treatment-relative pattern result
result_df <- xxtpt %>%
  mutate(
    treatment_duration = treat_dur$treatment_duration,
    xxtpt_up = str_to_upper(str_trim(xxtpt))
  ) %>%
  rowwise() %>%
  mutate(
    result = case_when(
      # Start / zero / pre-dose -> 0
      xxtpt_up %in% c("START OF TREATMENT", "PRE-DOSE", "TREATMENT START", "ZERO") ~ 0,

      # Patterns AFTER START -> positive offsets from start
      str_detect(xxtpt_up, "AFTER START") ~ parse_offset_to_hours(xxtpt_up),

      # BEFORE END / PRE-EOT -> negative offsets from end
      str_detect(xxtpt_up, "BEFORE END") | str_detect(xxtpt_up, "PRE-EOT") ~ {
        -parse_offset_to_hours(xxtpt_up)
      },

      # END / AFTER END -> end plus any offset
      xxtpt_up %in% c("END OF TREATMENT", "TREATMENT END") ~ as.numeric(treatment_duration),
      str_detect(xxtpt_up, "AFTER END") ~ {
        as.numeric(treatment_duration) + parse_offset_to_hours(xxtpt_up)
      },

      TRUE ~ NA_real_
    )
  ) %>%
  ungroup() %>%
  select(xxtpt, treatment_duration, result)

# Write output
write_csv(result_df, file.path("outputs", "result.csv"))
```

## Output
### Ground Truth Output
#### `result.csv`

```csv
"xxtpt","treatment_duration","result"
"START OF TREATMENT",72,0
"PRE-DOSE",72,-1
"30 MIN AFTER START",72,0.5
"2 HOURS AFTER START",72,2
"END OF TREATMENT",72,72
"4 HOURS BEFORE END",72,68
"6 HOURS AFTER END",72,78
```

### LLM Output
#### `result.csv`

```csv
xxtpt,treatment_duration,result
START OF TREATMENT,72,0
PRE-DOSE,72,0
30 MIN AFTER START,72,0.5
2 HOURS AFTER START,72,2
END OF TREATMENT,72,72
4 HOURS BEFORE END,72,-4
6 HOURS AFTER END,72,78
```
