# Case 079: pharmaverse/admiral/convert_post_end_patterns

## Metadata
- Task ID: `pharmaverse/admiral/convert_post_end_patterns`
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
Write an R script to perform convert post end patterns using the admiral clinical task contract.

Input: na_idx.tsv, result.tsv, treatment_duration.tsv, xxtpt.tsv
Output: result.csv


Required columns for result.csv: xxtpt, treatment_duration, result
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `na_idx.tsv`


```text
na_idx
FALSE
FALSE
FALSE
FALSE
```

#### `result.tsv`


```text
result
NA
NA
NA
4
```

#### `treatment_duration.tsv`


```text
treatment_duration
2
2
10
4
```

#### `xxtpt.tsv`


```text
xxtpt
30 min post EOI
1 hr after end of infusion
2 hours after end of treatment
unmatched
```

## Code
### Ground Truth Code

```r
suppressPackageStartupMessages(library(stringr))

xxtpt <- read.delim(file.path("inputs", "xxtpt.tsv"), check.names = FALSE, stringsAsFactors = FALSE)$xxtpt
result <- read.delim(file.path("inputs", "result.tsv"), check.names = FALSE, stringsAsFactors = FALSE)$result
result <- suppressWarnings(as.numeric(result))
na_idx <- read.delim(file.path("inputs", "na_idx.tsv"), check.names = FALSE, stringsAsFactors = FALSE)$na_idx
na_idx <- as.logical(na_idx)
treatment_duration <- read.delim(file.path("inputs", "treatment_duration.tsv"), check.names = FALSE, stringsAsFactors = FALSE)$treatment_duration
treatment_duration <- as.numeric(treatment_duration)

pattern <- regex("^(?<value>\\d+(?:\\.\\d+)?)\\s*(?<unit>m(?:in|inute)?|h(?:r|our)?)s?\\s+(?:post|after)\\s*-?\\s*(?:eo[it]|inf(?:usion)?|end\\s+of\\s+(?:infusion|treatment))$", ignore_case = TRUE)
matches <- str_match(xxtpt, pattern)
idx <- !is.na(matches[, 1]) & is.na(result) & !na_idx
if (any(idx)) {
  value <- as.numeric(matches[idx, "value"])
  unit <- tolower(matches[idx, "unit"])
  offset <- ifelse(substr(unit, 1, 1) == "m", value / 60, value)
  result[idx] <- treatment_duration[idx] + offset
}

dir.create("outputs", showWarnings = FALSE)
write.csv(data.frame(xxtpt = xxtpt, treatment_duration = treatment_duration, result = result), file.path("outputs", "result.csv"), row.names = FALSE)
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
if (!dir.exists("outputs")) dir.create("outputs", recursive = TRUE)

# Read inputs
na_idx <- read_tsv(
  file = file.path("inputs", "na_idx.tsv"),
  col_types = cols(.default = col_logical())
)

result_df <- read_tsv(
  file = file.path("inputs", "result.tsv"),
  col_types = cols(.default = col_double())
)

treat_dur <- read_tsv(
  file = file.path("inputs", "treatment_duration.tsv"),
  col_types = cols(.default = col_double())
)

xxtpt_df <- read_tsv(
  file = file.path("inputs", "xxtpt.tsv"),
  col_types = cols(.default = col_character())
)

# Combine inputs by row position
dat <- tibble(
  na_idx = na_idx$na_idx,
  result = result_df$result,
  treatment_duration = treat_dur$treatment_duration,
  xxtpt = xxtpt_df$xxtpt
)

# Helper: extract numeric value from pattern like "30 min", "1 hr", "2 hours"
extract_time_value <- function(x) {
  as.numeric(str_extract(x, "\\d+\\.*\\d*"))
}

# Convert xxtpt post-end patterns per admiral-like logic
dat_converted <- dat %>%
  mutate(
    # Standardize text
    xxtpt_std = str_to_lower(xxtpt),
    xxtpt_std = str_squish(xxtpt_std),

    # Identify post end/treatment/infusion patterns
    is_post_eoi = str_detect(xxtpt_std, "post eoi") |
      str_detect(xxtpt_std, "after end of infusion") |
      str_detect(xxtpt_std, "after end of treatment"),

    # Units: assume minutes if "min", hours if "hr", "hour", "hours"
    unit_hours = str_detect(xxtpt_std, "hr") |
      str_detect(xxtpt_std, "hour"),
    time_val = extract_time_value(xxtpt_std),

    # Convert to hours
    post_time_hr = case_when(
      is_post_eoi & unit_hours  ~ time_val,
      is_post_eoi & !unit_hours ~ time_val / 60,
      TRUE                      ~ NA_real_
    ),

    # New result according to post-end pattern:
    # If xxtpt describes "time after end" and na_idx is FALSE and result is NA,
    # impute as treatment_duration + post_time_hr (when post_time_hr available)
    result_new = case_when(
      !na_idx & is.na(result) & !is.na(post_time_hr) ~ treatment_duration + post_time_hr,
      TRUE                                           ~ result
    )
  )

# Prepare final output
out <- dat_converted %>%
  transmute(
    xxtpt,
    treatment_duration,
    result = result_new
  )

# Write output
write_csv(out, file.path("outputs", "result.csv"), na = "")
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"xxtpt","treatment_duration","result"
"30 min post EOI",2,2.5
"1 hr after end of infusion",2,3
"2 hours after end of treatment",10,12
"unmatched",4,4
```

### LLM Output
#### `result.csv`


```csv
xxtpt,treatment_duration,result
30 min post EOI,2,2.5
1 hr after end of infusion,2,3
2 hours after end of treatment,10,12
unmatched,4,4
```
