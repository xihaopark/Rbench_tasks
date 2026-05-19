# Case 136: pharmaverse/admiral/filter_not_exist

## Metadata
- Task ID: `pharmaverse/admiral/filter_not_exist`
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
Write an R script to perform filter not exist using the admiral clinical task contract.

Input: dataset.tsv, dataset_add.tsv
Output: result.csv


Required columns for result.csv: USUBJID, PARAMCD, AVAL
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `dataset.tsv`


```text
USUBJID	PARAMCD	AVAL
01	ALT	20
02	ALT	35
03	AST	18
```

#### `dataset_add.tsv`


```text
USUBJID	PARAMCD	AVAL
01	ALT	20
02	ALT	35
03	AST	18
04	ALT	42
05	AST	22
```

## Code
### Ground Truth Code

```r
dataset <- read.delim(file.path("inputs", "dataset.tsv"), check.names = FALSE, stringsAsFactors = FALSE)
dataset_add <- read.delim(file.path("inputs", "dataset_add.tsv"), check.names = FALSE, stringsAsFactors = FALSE)
key <- function(data) paste(data$USUBJID, data$PARAMCD, sep = "\r")
result_df <- dataset_add[!(key(dataset_add) %in% key(dataset)), , drop = FALSE]

dir.create("outputs", showWarnings = FALSE)
write.csv(result_df, file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

suppressPackageStartupMessages({
  library(dplyr)
  library(readr)
})

# Create outputs directory if it does not exist
if (!dir.exists("outputs")) dir.create("outputs", recursive = TRUE)

# Read input datasets
dataset <- read.delim(
  file = "inputs/dataset.tsv",
  header = TRUE,
  sep = "\t",
  stringsAsFactors = FALSE
)

dataset_add <- read.delim(
  file = "inputs/dataset_add.tsv",
  header = TRUE,
  sep = "\t",
  stringsAsFactors = FALSE
)

# Filter not exist: records in dataset_add that are not in dataset
result <- dataset_add %>%
  anti_join(dataset, by = c("USUBJID", "PARAMCD", "AVAL")) %>%
  select(USUBJID, PARAMCD, AVAL)

# Write result
write.csv(result, file = "outputs/result.csv", row.names = FALSE, na = "")
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"USUBJID","PARAMCD","AVAL"
4,"ALT",42
5,"AST",22
```

### LLM Output
#### `result.csv`


```csv
"USUBJID","PARAMCD","AVAL"
4,"ALT",42
5,"AST",22
```
