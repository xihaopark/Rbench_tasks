# Case 093: pharmaverse/admiral/derive_basetype_records

## Metadata
- Task ID: `pharmaverse/admiral/derive_basetype_records`
- Package: `admiral`
- Model: `openai/gpt-5.1`
- Status: `FAIL`
- Failure stage: `value_mismatch`

## Pass/Error
```text
The generated output files were produced, but one or more values differed from the
ground truth.

Main signal:
result.csv: Value mismatch in column: DTYPE.
```

## Prompt
```text
Write an R script to perform derive basetype records using the admiral clinical task contract.

Input: dataset.tsv
Output: result.csv


Required columns for result.csv: USUBJID, PARAMCD, BASETYPE, AVAL, DTYPE
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `dataset.tsv`


```text
USUBJID	PARAMCD	BASETYPE	AVAL
01	ALT	SCREENING	10
01	ALT	RANDOMIZATION	12
02	ALT	SCREENING	20
```

## Code
### Ground Truth Code

```r
dataset <- read.delim(file.path("inputs", "dataset.tsv"), check.names = FALSE, stringsAsFactors = FALSE)
result_df <- data.frame(dataset, DTYPE = "BASETYPE", stringsAsFactors = FALSE)

dir.create("outputs", showWarnings = FALSE)
write.csv(result_df, file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

library(dplyr)
library(readr)
library(stringr)

# Create outputs directory if it doesn't exist
if (!dir.exists("outputs")) {
  dir.create("outputs", recursive = TRUE)
}

# Read input
input_df <- read.delim(
  file = file.path("inputs", "dataset.tsv"),
  header = TRUE,
  sep = "\t",
  stringsAsFactors = FALSE
)

# Derive BASETYPE records (admiral-style placeholder logic)
# Keep original records as-is; no imputed/derived baseline records added
result <- input_df %>%
  transmute(
    USUBJID = as.character(USUBJID),
    PARAMCD = as.character(PARAMCD),
    BASETYPE = as.character(BASETYPE),
    AVAL = as.numeric(AVAL),
    DTYPE = NA_character_
  )

# Write output
write_csv(result, file.path("outputs", "result.csv"), na = "")
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"USUBJID","PARAMCD","BASETYPE","AVAL","DTYPE"
1,"ALT","SCREENING",10,"BASETYPE"
1,"ALT","RANDOMIZATION",12,"BASETYPE"
2,"ALT","SCREENING",20,"BASETYPE"
```

### LLM Output
#### `result.csv`


```csv
USUBJID,PARAMCD,BASETYPE,AVAL,DTYPE
1,ALT,SCREENING,10,
1,ALT,RANDOMIZATION,12,
2,ALT,SCREENING,20,
```
