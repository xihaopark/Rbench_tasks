# Case 134: pharmaverse/admiral/filter_exist

## Metadata
- Task ID: `pharmaverse/admiral/filter_exist`
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
Write an R script to perform filter exist using the admiral clinical task contract.

Input: dataset.tsv, required.tsv
Output: result.csv


Required columns for result.csv: USUBJID, PARAMCD, AVAL
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `dataset.tsv`


```text
USUBJID	PARAMCD	AVAL
01	ALT	10
01	AST	11
02	ALT	20
03	BILI	1.2
```

#### `required.tsv`


```text
USUBJID
01
02
```

## Code
### Ground Truth Code

```r
dataset <- read.delim(file.path("inputs", "dataset.tsv"), check.names = FALSE, stringsAsFactors = FALSE)
required <- read.delim(file.path("inputs", "required.tsv"), check.names = FALSE, stringsAsFactors = FALSE)
result_df <- dataset[dataset$USUBJID %in% required$USUBJID, , drop = FALSE]

dir.create("outputs", showWarnings = FALSE)
write.csv(result_df, file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
# Load required package
suppressPackageStartupMessages({
  library(readr)
  library(dplyr)
})

# Create outputs directory if it doesn't exist
if (!dir.exists("outputs")) {
  dir.create("outputs", recursive = TRUE)
}

# Read input files
dataset <- read_tsv(
  file = file.path("inputs", "dataset.tsv"),
  col_types = cols()
)

required <- read_tsv(
  file = file.path("inputs", "required.tsv"),
  col_types = cols()
)

# Filter based on required USUBJID
result <- dataset %>%
  semi_join(required, by = "USUBJID") %>%
  select(USUBJID, PARAMCD, AVAL)

# Write result to CSV
write_csv(result, file.path("outputs", "result.csv"))
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"USUBJID","PARAMCD","AVAL"
1,"ALT",10
1,"AST",11
2,"ALT",20
```

### LLM Output
#### `result.csv`


```csv
USUBJID,PARAMCD,AVAL
01,ALT,10
01,AST,11
02,ALT,20
```
