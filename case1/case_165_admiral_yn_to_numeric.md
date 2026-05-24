# Case 165: pharmaverse/admiral/yn_to_numeric

## Metadata
- Task ID: `pharmaverse/admiral/yn_to_numeric`
- Package: `admiral`
- Model: `openai/gpt-5.1`
- Status: `FAIL`
- Failure stage: `value_mismatch`

## Pass/Error
```text
The generated output files were produced, but one or more values differed from the
ground truth.

Main signal:
result.csv: Numeric missingness mismatch in column: result.
```

## Prompt
```text
Write an R script to perform yn to numeric using the admiral clinical task contract.

Input: arg.tsv
Output: result.csv


Required columns for result.csv: arg, result
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `arg.tsv`


```text
arg
Y
N
y
n
UNKNOWN
```

## Code
### Ground Truth Code

```r
suppressPackageStartupMessages(library(admiral))

arg <- as.character(read.delim(file.path("inputs", "arg.tsv"), check.names = FALSE, stringsAsFactors = FALSE)[[1]])
result <- admiral::yn_to_numeric(arg)

dir.create("outputs", showWarnings = FALSE)
write.csv(data.frame(arg = arg, result = result), file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

# Load required package(s)
suppressPackageStartupMessages({
  if (!requireNamespace("readr", quietly = TRUE)) {
    stop("Package 'readr' is required but not installed.")
  }
  if (!requireNamespace("dplyr", quietly = TRUE)) {
    stop("Package 'dplyr' is required but not installed.")
  }
})

library(readr)
library(dplyr)

# Define input/output paths
input_path  <- file.path("inputs", "arg.tsv")
output_dir  <- "outputs"
output_path <- file.path(output_dir, "result.csv")

# Create outputs directory if it does not exist
if (!dir.exists(output_dir)) {
  dir.create(output_dir, recursive = TRUE)
}

# Read input
arg_df <- readr::read_tsv(
  input_path,
  col_types = cols(
    arg = col_character()
  )
)

# Y/N to numeric according to admiral-style logic:
# - Case-insensitive
# - "Y" -> 1
# - "N" -> 0
# - All other values -> NA
yn_to_numeric <- function(x) {
  x_up <- toupper(x)
  dplyr::case_when(
    x_up == "Y" ~ 1,
    x_up == "N" ~ 0,
    TRUE        ~ NA_real_
  )
}

result_df <- arg_df %>%
  mutate(result = yn_to_numeric(arg)) %>%
  select(arg, result)

# Write output
readr::write_csv(result_df, output_path, na = "")
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"arg","result"
"Y",1
"N",0
"y",NA
"n",NA
"UNKNOWN",NA
```

### LLM Output
#### `result.csv`


```csv
arg,result
Y,1
N,0
y,1
n,0
UNKNOWN,
```
