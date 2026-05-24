# Case 083: pharmaverse/admiral/convert_simple_units

## Metadata
- Task ID: `pharmaverse/admiral/convert_simple_units`
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
Write an R script to perform convert simple units using the admiral clinical task contract.

Input: xxtpt.tsv
Output: result.csv


Required columns for result.csv: xxtpt, result
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `xxtpt.tsv`


```text
xxtpt
0.5 h
30 min post dose
2 hours after dose
90 mins
1.25 hr
```

## Code
### Ground Truth Code

```r
xxtpt <- as.character(read.delim(file.path("inputs", "xxtpt.tsv"), check.names = FALSE, stringsAsFactors = FALSE)[[1]])

convert_one <- function(x) {
  value <- trimws(tolower(x))
  hour_match <- regexec("^([0-9]+(?:\\.[0-9]+)?)\\s*h(?:r|our)?s?(?:\\s+(?:post|after)(?:\\s*-?\\s*dose)?)?$", value)
  hour <- regmatches(value, hour_match)[[1]]
  if (length(hour) > 0) {
    return(as.numeric(hour[2]))
  }
  min_match <- regexec("^([0-9]+(?:\\.[0-9]+)?)\\s*m(?:in|inute)?s?(?:\\s+(?:post|after)(?:\\s*-?\\s*dose)?)?$", value)
  minute <- regmatches(value, min_match)[[1]]
  if (length(minute) > 0) {
    return(as.numeric(minute[2]) / 60)
  }
  NA_real_
}

result <- vapply(xxtpt, convert_one, numeric(1))

dir.create("outputs", showWarnings = FALSE)
write.csv(data.frame(xxtpt = xxtpt, result = result), file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

suppressPackageStartupMessages({
  library(dplyr)
  library(readr)
  library(stringr)
})

# ensure outputs directory exists
if (!dir.exists("outputs")) dir.create("outputs", recursive = TRUE)

# read input
xxtpt_df <- read_tsv(
  file = file.path("inputs", "xxtpt.tsv"),
  col_types = cols(.default = col_character())
)

# function to convert textual time to hours
convert_to_hours <- function(x) {
  x_clean <- tolower(trimws(x))
  # extract first numeric value
  num <- suppressWarnings(as.numeric(str_extract(x_clean, "-?\\d*\\.?\\d+")))
  if (is.na(num)) return(NA_real_)

  # determine unit: look at presence of hour/min keywords
  if (str_detect(x_clean, "hour|hr|hrs|h(\\b|$)")) {
    val <- num
  } else if (str_detect(x_clean, "min")) {
    val <- num / 60
  } else {
    # default: assume hours if "dose" with a decimal number, otherwise NA
    val <- num
  }
  val
}

result_df <- xxtpt_df %>%
  mutate(
    result = convert_to_hours(xxtpt)
  ) %>%
  select(xxtpt, result)

write_csv(result_df, file.path("outputs", "result.csv"))
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"xxtpt","result"
"0.5 h",0.5
"30 min post dose",0.5
"2 hours after dose",2
"90 mins",1.5
"1.25 hr",1.25
```

### LLM Output
No LLM output artifact was produced.
