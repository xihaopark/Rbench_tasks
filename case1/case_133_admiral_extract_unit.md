# Case 133: pharmaverse/admiral/extract_unit

## Metadata
- Task ID: `pharmaverse/admiral/extract_unit`
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
Write an R script to perform extract unit using the admiral clinical task contract.

Input: x.tsv
Output: result.csv


Required columns for result.csv: x, result
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `x.tsv`


```text
x
"Systolic blood pressure (mmHg)"
"Dose normalized AUC (h*ng/mL)"
"Body weight (kg)"
"QT interval (msec)"
"Heart rate"
```

## Code
### Ground Truth Code

```r
suppressPackageStartupMessages(library(admiral))
suppressPackageStartupMessages(library(stringr))

x_df <- read.delim(file.path("inputs", "x.tsv"), check.names = FALSE, stringsAsFactors = FALSE)
x <- as.character(x_df[[ncol(x_df)]])
if (length(x) == 0) x <- "10 (mg)"

result <- admiral::extract_unit(x)

dir.create("outputs", showWarnings = FALSE)
write.csv(data.frame(x = x, result = result), file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

library(readr)
library(dplyr)
library(stringr)
library(fs)

# Create outputs directory if it doesn't exist
if (!dir_exists("outputs")) {
  dir_create("outputs")
}

# Read input
x <- read_tsv("inputs/x.tsv", show_col_types = FALSE)

# Simple unit extraction using patterns common in clinical text
extract_unit <- function(text) {
  # Look for parentheses content
  unit_paren <- str_match(text, "\\(([^()]*)\\)")[, 2]
  unit_paren <- str_trim(unit_paren)

  # If parentheses exist and are not empty, use them
  if (!is.na(unit_paren) && unit_paren != "") {
    return(unit_paren)
  }

  # Otherwise return blank
  return(NA_character_)
}

result <- x %>%
  mutate(result = vapply(x, extract_unit, character(1))) %>%
  select(x, result)

# Write output
write_csv(result, "outputs/result.csv", na = "")
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"x","result"
"Systolic blood pressure (mmHg)","mmHg"
"Dose normalized AUC (h*ng/mL)","h*ng/mL"
"Body weight (kg)","kg"
"QT interval (msec)","msec"
"Heart rate",NA
```

### LLM Output
#### `result.csv`


```csv
x,result
Systolic blood pressure (mmHg),mmHg
Dose normalized AUC (h*ng/mL),h*ng/mL
Body weight (kg),kg
QT interval (msec),msec
Heart rate,
```
