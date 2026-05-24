# Case 140: pharmaverse/admiral/get_admiral_option

## Metadata
- Task ID: `pharmaverse/admiral/get_admiral_option`
- Package: `admiral`
- Model: `openai/gpt-5.1`
- Status: `FAIL`
- Failure stage: `schema_mismatch`

## Pass/Error
```text
The generated output file schema did not match the ground truth schema.

Main signal:
result.csv: Shape mismatch: ref=(2, 2) vs llm=(1, 2).
```

## Prompt
```text
Write an R script to perform get admiral option using the admiral clinical task contract.

Input: option.tsv
Output: result.csv


Required columns for result.csv: option, result
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `option.tsv`


```text
option
"subject_keys"
```

## Code
### Ground Truth Code

```r
suppressPackageStartupMessages(library(admiral))

option_df <- read.delim(file.path("inputs", "option.tsv"), check.names = FALSE, stringsAsFactors = FALSE)
option <- as.character(option_df[[ncol(option_df)]][1])

valid_opts <- c("subject_keys", "signif_digits", "max_n_char")
if (!option %in% valid_opts) option <- valid_opts[1]

result <- admiral::get_admiral_option(option)

dir.create("outputs", showWarnings = FALSE)
write.csv(data.frame(option = option, result = as.character(result)), file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

suppressPackageStartupMessages({
  library(readr)
  library(dplyr)
  library(admiral)
})

# Ensure outputs directory exists
if (!dir.exists("outputs")) {
  dir.create("outputs", recursive = TRUE)
}

# Read options
opt_df <- read_tsv(
  file = file.path("inputs", "option.tsv"),
  show_col_types = FALSE
)

# Helper to safely get admiral option value as character
get_admiral_option_value <- function(opt_name) {
  val <- get_admiral_option(opt_name)
  if (is.null(val)) {
    return(NA_character_)
  }
  if (is.atomic(val) && length(val) == 1) {
    return(as.character(val))
  }
  # For non-scalar options, serialize to JSON-like string
  paste(capture.output(str(val, give.attr = FALSE)), collapse = " ")
}

result <- opt_df %>%
  mutate(
    result = vapply(option, get_admiral_option_value, character(1))
  ) %>%
  select(option, result)

write_csv(result, file.path("outputs", "result.csv"), na = "")
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"option","result"
"subject_keys","STUDYID"
"subject_keys","USUBJID"
```

### LLM Output
#### `result.csv`


```csv
option,result
subject_keys,List of 2  $ : symbol STUDYID  $ : symbol USUBJID
```
