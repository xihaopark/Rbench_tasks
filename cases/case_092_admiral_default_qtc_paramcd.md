# Case 092: pharmaverse/admiral/default_qtc_paramcd

## Metadata
- Task ID: `pharmaverse/admiral/default_qtc_paramcd`
- Package: `admiral`
- Model: `openai/gpt-5.1`
- Status: `FAIL`
- Failure stage: `value_mismatch`

## Pass/Error
```text
The generated output files were produced, but one or more values differed from the
ground truth.

Main signal:
result.csv: Value mismatch in column: PARAMCD.
```

## Prompt
```text
Write an R script to perform default qtc paramcd using the admiral clinical task contract.

Input: method.tsv
Output: result.csv


Required columns for result.csv: method, PARAMCD
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `method.tsv`


```text
method
Bazett
```

## Code
### Ground Truth Code

```r
suppressPackageStartupMessages(library(admiral))

method <- as.character(read.delim(file.path("inputs", "method.tsv"), check.names = FALSE, stringsAsFactors = FALSE)$method[[1]])
result <- admiral::default_qtc_paramcd(method)

dir.create("outputs", showWarnings = FALSE)
write.csv(data.frame(method = method, PARAMCD = result$PARAMCD), file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

library(readr)
library(dplyr)
library(stringr)

# Create outputs directory if it doesn't exist
if (!dir.exists("outputs")) {
  dir.create("outputs", recursive = TRUE)
}

# Read input method file
method_path <- file.path("inputs", "method.tsv")
method_df <- read_tsv(method_path, show_col_types = FALSE)

# Default mapping of QTC methods to PARAMCD per admiral conventions
# Extend this mapping as needed
qtc_paramcd_map <- tibble(
  method = c("Bazett", "Fridericia", "Framingham", "Hodges"),
  PARAMCD = c("QTCB", "QTCF", "QTCFRM", "QTCH")
)

# Join and derive PARAMCD
result <- method_df %>%
  left_join(qtc_paramcd_map, by = "method") %>%
  mutate(PARAMCD = if_else(is.na(PARAMCD), NA_character_, PARAMCD)) %>%
  select(method, PARAMCD)

# Write output
output_path <- file.path("outputs", "result.csv")
write_csv(result, output_path)
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"method","PARAMCD"
"Bazett","QTCBR"
```

### LLM Output
#### `result.csv`


```csv
method,PARAMCD
Bazett,QTCB
```
