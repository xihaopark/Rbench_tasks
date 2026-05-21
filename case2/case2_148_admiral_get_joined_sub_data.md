# Case 148: pharmaverse/admiral/get_joined_sub_data

## Metadata
- Task ID: `pharmaverse/admiral/get_joined_sub_data`
- Package: `admiral`
- Model: `openai/gpt-5.1`
- Status: `PASS`
- Failure stage: `pass`

## Pass/Error
```text
PASS
```

## Prompt
```text
Write an R script for an R package task: perform get joined sub data using the admiral package. The function(s) you use to solve the task must come from the admiral package.

Output: result.csv

Computation: Use left.tsv as the primary dataset and left-join right.tsv by USUBJID, preserving unmatched left rows.


Required columns for result.csv: USUBJID, AVAL, TRT01A
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `left.tsv`

```text
USUBJID	AVAL
01	10
02	20
03	30
```

#### `right.tsv`

```text
USUBJID	TRT01A
01	Drug A
02	Placebo
```

## Code
### Ground Truth Code

```r
left <- read.delim(file.path("inputs", "left.tsv"), check.names = FALSE, stringsAsFactors = FALSE)
right <- read.delim(file.path("inputs", "right.tsv"), check.names = FALSE, stringsAsFactors = FALSE)
result_df <- merge(left, right, by = "USUBJID", all.x = TRUE, sort = FALSE)

dir.create("outputs", showWarnings = FALSE)
write.csv(result_df, file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

# Load required packages
library(admiral)
library(readr)
library(dplyr)

# Create outputs directory if it doesn't exist
if (!dir.exists("outputs")) {
  dir.create("outputs", recursive = TRUE)
}

# Read input data
left <- read_tsv(
  file = file.path("inputs", "left.tsv"),
  col_types = cols()
)

right <- read_tsv(
  file = file.path("inputs", "right.tsv"),
  col_types = cols()
)

# Perform left join using admiral
result <- derive_vars_merged(
  dataset = left,
  dataset_add = right,
  by_vars = exprs(USUBJID),
  new_vars = exprs(TRT01A),
  filter_add = TRUE,
  mode = "first"
)

# Keep only required columns
result <- result %>%
  select(USUBJID, AVAL, TRT01A)

# Write output
write_csv(result, file.path("outputs", "result.csv"), na = "")
```

## Output
### Ground Truth Output
#### `result.csv`

```csv
"USUBJID","AVAL","TRT01A"
1,10,"Drug A"
2,20,"Placebo"
3,30,NA
```

### LLM Output
#### `result.csv`

```csv
USUBJID,AVAL,TRT01A
01,10,Drug A
02,20,Placebo
03,30,
```
