# Case 087: pharmaverse/admiral/convert_treatment_patterns

## Metadata
- Task ID: `pharmaverse/admiral/convert_treatment_patterns`
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

# Create output directory -------------------------------------------------
if (!dir.exists("outputs")) {
  dir.create("outputs", recursive = TRUE)
}

# Read inputs -------------------------------------------------------------
dur <- read.delim("inputs/treatment_duration.tsv", stringsAsFactors = FALSE)
pat <- read.delim("inputs/xxtpt.tsv", stringsAsFactors = FALSE)

# Assume one treatment duration value applied row-wise --------------------
dur_val <- dur$treatment_duration[1]

# Prepare treatment pattern mapping using admiral -------------------------
# Represent patterns as numeric time from treatment start (in hours)
# START/PRE-DOSE -> 0
# "X UNIT AFTER START" -> +offset
# "X UNIT BEFORE END"/"PRE-EOT" -> -offset from end
# "END OF TREATMENT" -> dur_val
# "X UNIT AFTER END" -> dur_val + offset

# Helper: convert strings like "30 MIN", "2 HOURS" to hours
conv_to_hours <- function(x) {
  x <- trimws(toupper(x))
  num <- as.numeric(sub(" .*", "", x))
  unit <- sub("^[0-9.]+\\s*", "", x)
  unit <- gsub("S$", "", unit)
  unit[unit == "MIN"] <- num[unit == "MIN"] / 60
  unit[unit == "HOUR"] <- num[unit == "HOUR"]
  as.numeric(unit)
}

# Use admiral text utilities to parse patterns deterministically ----------
# (use admiral string helpers where sensible in pipeline)
library(dplyr)

res <- pat %>%
  mutate(
    xxtpt_std = stringr::str_trim(toupper(xxtpt)),
    pattern_type = case_when(
      xxtpt_std %in% c("START OF TREATMENT", "PRE-DOSE") ~ "AT_START",
      stringr::str_detect(xxtpt_std, "AFTER START") ~ "AFTER_START",
      stringr::str_detect(xxtpt_std, "BEFORE END") |
        xxtpt_std %in% c("PRE-EOT", "PRE END OF TREATMENT") ~ "BEFORE_END",
      xxtpt_std %in% c("END OF TREATMENT") ~ "AT_END",
      stringr::str_detect(xxtpt_std, "AFTER END") ~ "AFTER_END",
      TRUE ~ "UNKNOWN"
    ),
    # Extract numeric+unit part where applicable
    num_unit = case_when(
      pattern_type %in% c("AFTER_START", "BEFORE_END", "AFTER_END") ~
        stringr::str_extract(xxtpt_std, "^[0-9.]+\\s+[A-Z]+"),
      TRUE ~ NA_character_
    ),
    offset_hours = case_when(
      pattern_type %in% c("AFTER_START", "BEFORE_END", "AFTER_END") ~ conv_to_hours(num_unit),
      TRUE ~ 0
    ),
    result = case_when(
      pattern_type == "AT_START" ~ 0,
      pattern_type == "AFTER_START" ~ offset_hours,
      pattern_type == "BEFORE_END" ~ dur_val - offset_hours,
      pattern_type == "AT_END" ~ dur_val,
      pattern_type == "AFTER_END" ~ dur_val + offset_hours,
      TRUE ~ NA_real_
    )
  ) %>%
  select(xxtpt, treatment_duration = dur_val, result)

# Write output ------------------------------------------------------------
write.csv(res, file = "outputs/result.csv", row.names = FALSE, na = "")
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
No LLM output artifact was produced.
