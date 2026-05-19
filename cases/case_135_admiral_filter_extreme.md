# Case 135: pharmaverse/admiral/filter_extreme

## Metadata
- Task ID: `pharmaverse/admiral/filter_extreme`
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
Write an R script to perform filter extreme using the admiral clinical task contract.

Input: dataset.tsv, mode.tsv
Output: result.csv


Required columns for result.csv: USUBJID, PARAMCD, AVISITN, AVAL
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `dataset.tsv`


```text
USUBJID	PARAMCD	AVISITN	AVAL
01	ALT	1	22
01	ALT	2	40
01	ALT	3	35
02	ALT	1	18
02	ALT	2	29
```

#### `mode.tsv`


```text
mode
last
```

## Code
### Ground Truth Code

```r
dataset <- read.delim(file.path("inputs", "dataset.tsv"), check.names = FALSE, stringsAsFactors = FALSE)
mode <- as.character(read.delim(file.path("inputs", "mode.tsv"), check.names = FALSE, stringsAsFactors = FALSE)$mode[1])
dataset <- dataset[order(dataset$USUBJID, dataset$PARAMCD, dataset$AVISITN), , drop = FALSE]
groups <- interaction(dataset$USUBJID, dataset$PARAMCD, drop = TRUE)
idx <- if (mode == "first") {
  !duplicated(groups)
} else {
  !duplicated(groups, fromLast = TRUE)
}
result_df <- dataset[idx, , drop = FALSE]

dir.create("outputs", showWarnings = FALSE)
write.csv(result_df, file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

suppressPackageStartupMessages({
  library(readr)
  library(dplyr)
})

# Create outputs directory if it doesn't exist
if (!dir.exists("outputs")) dir.create("outputs", recursive = TRUE)

# Read inputs
dataset <- read_tsv(
  file = file.path("inputs", "dataset.tsv"),
  col_types = cols(
    USUBJID = col_character(),
    PARAMCD = col_character(),
    AVISITN = col_double(),
    AVAL = col_double()
  )
)

mode_df <- read_tsv(
  file = file.path("inputs", "mode.tsv"),
  col_types = cols(mode = col_character())
)

mode_val <- tolower(mode_df$mode[1])

# Filter extreme per admiral clinical task contract (mode: first/last)
result <- dataset %>%
  group_by(USUBJID, PARAMCD) %>%
  {
    if (mode_val == "first") {
      slice_min(., order_by = AVISITN, with_ties = FALSE)
    } else if (mode_val == "last") {
      slice_max(., order_by = AVISITN, with_ties = FALSE)
    } else {
      stop("Unsupported mode in mode.tsv. Expected 'first' or 'last'.")
    }
  } %>%
  ungroup() %>%
  select(USUBJID, PARAMCD, AVISITN, AVAL)

# Write output
write_csv(result, file.path("outputs", "result.csv"), na = "")
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"USUBJID","PARAMCD","AVISITN","AVAL"
1,"ALT",3,35
2,"ALT",2,29
```

### LLM Output
#### `result.csv`


```csv
USUBJID,PARAMCD,AVISITN,AVAL
01,ALT,3,35
02,ALT,2,29
```
