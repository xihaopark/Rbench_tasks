# Case 193: pharmaverse/sdtmchecks/fail

## Metadata
- Task ID: `pharmaverse/sdtmchecks/fail`
- Package: `sdtmchecks`
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
Write an R script to perform fail using the sdtmchecks clinical task contract.

Input: data.tsv, msg.tsv
Output: result.csv


Required columns for result.csv: result, msg, data_rows, data_cols
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `data.tsv`


```text
id	value	group	category
1	10.5	A	Type1
2	20.3	B	Type2
3	30.7	A	Type1
4	40.2	B	Type2
5	50.9	A	Type1
```

#### `msg.tsv`


```text
msg
Required SDTM variable is missing
```

## Code
### Ground Truth Code

```r
suppressPackageStartupMessages(library(sdtmchecks))

dir.create("outputs", showWarnings = FALSE, recursive = TRUE)

msg_df <- read.delim(
  file.path("inputs", "msg.tsv"),
  check.names = FALSE,
  stringsAsFactors = FALSE
)
data <- read.delim(
  file.path("inputs", "data.tsv"),
  check.names = FALSE,
  stringsAsFactors = FALSE
)

if (!"msg" %in% names(msg_df) || nrow(msg_df) < 1) {
  stop("inputs/msg.tsv must contain at least one row in a msg column")
}

msg <- as.character(msg_df$msg[[1]])
result <- sdtmchecks::fail(msg = msg, data = data)
result_data <- attr(result, "data", exact = TRUE)
if (!is.data.frame(result_data)) {
  result_data <- data
}

result_df <- data.frame(
  result = unname(as.logical(result)),
  msg = as.character(attr(result, "msg", exact = TRUE)),
  data_rows = nrow(result_data),
  data_cols = ncol(result_data),
  stringsAsFactors = FALSE
)

write.csv(result_df, file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

suppressPackageStartupMessages({
  library(readr)
  library(dplyr)
})

# Ensure outputs directory exists
if (!dir.exists("outputs")) {
  dir.create("outputs", recursive = TRUE)
}

# Read inputs
data_path <- file.path("inputs", "data.tsv")
msg_path  <- file.path("inputs", "msg.tsv")

data_df <- read_tsv(data_path, col_types = cols(.default = col_character()))
msg_df  <- read_tsv(msg_path,  col_types = cols(.default = col_character()))

# Prepare result
result <- tibble(
  result    = "FAIL",
  msg       = msg_df$msg[1],
  data_rows = nrow(data_df),
  data_cols = ncol(data_df)
)

# Write output
write_csv(result, file.path("outputs", "result.csv"), na = "")
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"result","msg","data_rows","data_cols"
FALSE,"Required SDTM variable is missing",5,4
```

### LLM Output
#### `result.csv`


```csv
result,msg,data_rows,data_cols
FAIL,Required SDTM variable is missing,5,4
```
