# Case 148: pharmaverse/admiral/get_joined_sub_data

## Metadata
- Task ID: `pharmaverse/admiral/get_joined_sub_data`
- Package: `admiral`
- Model: `openai/gpt-5.5`
- Status: `PASS`
- Failure stage: `pass`

## Pass/Error
```text
PASS
```

## Prompt
```text
Write an R script for an R package task: perform get joined sub data using the admiral package. Use functions from the admiral package when suitable.

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

library(admiral)
library(dplyr)
library(readr)

dir.create("outputs", showWarnings = FALSE, recursive = TRUE)

left <- read_tsv(
  "inputs/left.tsv",
  col_types = cols(
    USUBJID = col_character(),
    AVAL = col_double()
  )
)

right <- read_tsv(
  "inputs/right.tsv",
  col_types = cols(
    USUBJID = col_character(),
    TRT01A = col_character()
  )
)

result <- derive_vars_merged(
  dataset = left,
  dataset_add = right,
  by_vars = rlang::exprs(USUBJID),
  new_vars = rlang::exprs(TRT01A)
) %>%
  select(USUBJID, AVAL, TRT01A)

write_csv(result, "outputs/result.csv")
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
03,30,NA
```
