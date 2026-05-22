# Case 087: pharmaverse/admiral/convert_treatment_patterns

## Metadata
- Task ID: `pharmaverse/admiral/convert_treatment_patterns`
- Package: `admiral`
- Model: `openai/gpt-5.5`
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
suppressPackageStartupMessages({
  library(admiral)
  library(readr)
  library(dplyr)
  library(stringr)
})

dir.create("outputs", showWarnings = FALSE, recursive = TRUE)

trtdur <- read_tsv("inputs/treatment_duration.tsv", show_col_types = FALSE)
xxtpt <- read_tsv("inputs/xxtpt.tsv", show_col_types = FALSE)

dat <- bind_cols(
  xxtpt = xxtpt$xxtpt,
  treatment_duration = trtdur$treatment_duration
)

dat <- admiral::convert_blanks_to_na(dat)

parse_offset <- function(txt) {
  txt <- str_to_upper(str_squish(txt))

  num <- str_match(txt, "(\\d+(?:\\.\\d+)?)")[, 2]
  num <- suppressWarnings(as.numeric(num))

  multiplier <- case_when(
    str_detect(txt, "\\bMIN\\b|\\bMINS\\b|\\bMINUTE\\b|\\bMINUTES\\b") ~ 1 / 60,
    str_detect(txt, "\\bH\\b|\\bHR\\b|\\bHRS\\b|\\bHOUR\\b|\\bHOURS\\b") ~ 1,
    str_detect(txt, "\\bDAY\\b|\\bDAYS\\b") ~ 24,
    str_detect(txt, "\\bWEEK\\b|\\bWEEKS\\b") ~ 24 * 7,
    TRUE ~ 0
  )

  ifelse(is.na(num), 0, num * multiplier)
}

convert_treatment_pattern <- function(xxtpt, treatment_duration) {
  txt <- str_to_upper(str_squish(xxtpt))
  offset <- parse_offset(txt)

  case_when(
    is.na(txt) | is.na(treatment_duration) ~ NA_real_,

    str_detect(txt, "PRE[- ]?DOSE|START OF TREATMENT|TREATMENT START|^START$|ZERO|TIME 0|TIME ZERO") ~ 0,

    str_detect(txt, "AFTER START|POST[- ]?START|FROM START") ~ offset,

    str_detect(txt, "BEFORE END|PRIOR TO END|PRE[- ]?EOT|PRE[- ]?END|BEFORE EOT|PRIOR TO EOT") ~
      treatment_duration - offset,

    str_detect(txt, "AFTER END|POST[- ]?END|AFTER EOT|POST[- ]?EOT") ~
      treatment_duration + offset,

    str_detect(txt, "END OF TREATMENT|TREATMENT END|^END$|\\bEOT\\b") ~ treatment_duration,

    TRUE ~ NA_real_
  )
}

result <- dat %>%
  mutate(
    treatment_duration = as.numeric(treatment_duration),
    result = convert_treatment_pattern(xxtpt, treatment_duration)
  ) %>%
  select(xxtpt, treatment_duration, result)

write_csv(result, "outputs/result.csv")
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
4 HOURS BEFORE END,72,68
6 HOURS AFTER END,72,78
```
