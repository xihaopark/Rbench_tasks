# Case 132: pharmaverse/admiral/extract_duplicate_records

## Metadata
- Task ID: `pharmaverse/admiral/extract_duplicate_records`
- Package: `admiral`
- Model: `openai/gpt-5.1`
- Status: `FAIL`
- Failure stage: `schema_mismatch`

## Pass/Error
```text
The generated output file schema did not match the ground truth schema.

Main signal:
result.csv: Shape mismatch: ref=(4, 4) vs llm=(2, 4).
```

## Prompt
```text
Write an R script to perform extract duplicate records using the admiral clinical task contract.

Input: dataset.tsv
Output: result.csv


Required columns for result.csv: USUBJID, PARAMCD, AVISIT, AVAL
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `dataset.tsv`


```text
USUBJID	PARAMCD	AVISIT	AVAL
01	ALT	Week 1	10
01	ALT	Week 1	10
02	ALT	Week 1	20
03	AST	Week 1	30
03	AST	Week 1	31
```

## Code
### Ground Truth Code

```r
dataset <- read.delim(file.path("inputs", "dataset.tsv"), check.names = FALSE, stringsAsFactors = FALSE)
key <- paste(dataset$USUBJID, dataset$PARAMCD, dataset$AVISIT, sep = "\r")
result_df <- dataset[duplicated(key) | duplicated(key, fromLast = TRUE), , drop = FALSE]

dir.create("outputs", showWarnings = FALSE)
write.csv(result_df, file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

# Load required packages
suppressPackageStartupMessages({
  library(dplyr)
  library(readr)
})

# Ensure outputs directory exists
if (!dir.exists("outputs")) {
  dir.create("outputs", recursive = TRUE)
}

# Read input data
input_path <- file.path("inputs", "dataset.tsv")
dat <- read_tsv(input_path, col_types = cols())

# Find duplicated records based on all required columns
dup_flag <- duplicated(dat[, c("USUBJID", "PARAMCD", "AVISIT", "AVAL")]) |
  duplicated(dat[, c("USUBJID", "PARAMCD", "AVISIT", "AVAL")], fromLast = TRUE)

result <- dat[dup_flag, c("USUBJID", "PARAMCD", "AVISIT", "AVAL")]

# Write output
output_path <- file.path("outputs", "result.csv")
write_csv(result, output_path)
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"USUBJID","PARAMCD","AVISIT","AVAL"
1,"ALT","Week 1",10
1,"ALT","Week 1",10
3,"AST","Week 1",30
3,"AST","Week 1",31
```

### LLM Output
#### `result.csv`


```csv
USUBJID,PARAMCD,AVISIT,AVAL
01,ALT,Week 1,10
01,ALT,Week 1,10
```
