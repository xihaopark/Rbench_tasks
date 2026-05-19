# Case 170: pharmaverse/admiraldev/is_order_vars

## Metadata
- Task ID: `pharmaverse/admiraldev/is_order_vars`
- Package: `admiraldev`
- Model: `openai/gpt-5.1`
- Status: `FAIL`
- Failure stage: `schema_mismatch`

## Pass/Error
```text
The generated output file schema did not match the ground truth schema.

Main signal:
result.csv: Shape mismatch: ref=(1, 1) vs llm=(3, 1).
```

## Prompt
```text
Write an R script to perform is order vars using the admiraldev clinical task contract.

Input: arg.tsv
Output: result.csv


Required columns for result.csv: is_order_vars
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `arg.tsv`


```text
var
USUBJID
AVISITN
ADT
```

## Code
### Ground Truth Code

```r
suppressPackageStartupMessages(library(admiraldev))
suppressPackageStartupMessages(library(rlang))

vars <- read.delim(file.path("inputs", "arg.tsv"), check.names = FALSE, stringsAsFactors = FALSE)$var
arg <- as.list(lapply(vars, sym))
class(arg) <- c("quosures", "list")
result <- data.frame(is_order_vars = admiraldev::is_order_vars(arg))
dir.create("outputs", showWarnings = FALSE)
write.csv(result, file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

suppressPackageStartupMessages({
  library(readr)
  library(dplyr)
  library(stringr)
})

# Create outputs directory if it doesn't exist
if (!dir.exists("outputs")) {
  dir.create("outputs", recursive = TRUE)
}

# Read input
arg <- read_tsv(
  file = file.path("inputs", "arg.tsv"),
  col_types = cols(.default = col_character())
)

vars <- arg$var

# Define is_order_vars according to admiraldev clinical conventions
is_order_vars <- vars %in% c("USUBJID", "STUDYID", "SUBJID", "SITEID") |
  str_detect(vars, "^TRTSTDT$|^TRTEDT$|^TRT[A-Z]*STDT$|^TRT[A-Z]*EDT$") |
  str_detect(vars, "DT$|DTM$|TM$|DY$|N$")

# Assemble result
result <- tibble(is_order_vars = is_order_vars)

# Write output
write_csv(result, file.path("outputs", "result.csv"))
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"is_order_vars"
TRUE
```

### LLM Output
#### `result.csv`


```csv
is_order_vars
TRUE
TRUE
TRUE
```
