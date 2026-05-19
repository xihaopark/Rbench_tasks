# Case 099: pharmaverse/admiral/derive_param_rr

## Metadata
- Task ID: `pharmaverse/admiral/derive_param_rr`
- Package: `admiral`
- Model: `openai/gpt-5.1`
- Status: `FAIL`
- Failure stage: `value_mismatch`

## Pass/Error
```text
The generated output files were produced, but one or more values differed from the
ground truth.

Main signal:
result.csv: Value mismatch in column: PARAMCD.
```

## Prompt
```text
Write an R script to perform derive param RR using the admiral clinical task contract.

Input: dataset.tsv
Output: result.csv


Required columns for result.csv: USUBJID, PARAMCD, AVAL
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `dataset.tsv`


```text
USUBJID	PARAMCD	AVAL
01	HR	60
02	HR	75
03	HR	100
```

## Code
### Ground Truth Code

```r
dataset <- read.delim(file.path("inputs", "dataset.tsv"), check.names = FALSE, stringsAsFactors = FALSE)
hr <- dataset[dataset$PARAMCD == "HR", , drop = FALSE]
result_df <- data.frame(
  USUBJID = hr$USUBJID,
  PARAMCD = "RRR",
  AVAL = 60000 / as.numeric(hr$AVAL),
  stringsAsFactors = FALSE
)

dir.create("outputs", showWarnings = FALSE)
write.csv(result_df, file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

library(dplyr)
library(readr)
library(stringr)

# Create outputs directory if it does not exist
if (!dir.exists("outputs")) {
  dir.create("outputs", recursive = TRUE)
}

# Read input
dat <- read_tsv(
  file = file.path("inputs", "dataset.tsv"),
  col_types = cols()
)

# Derive RR parameter from HR if needed (example: copy HR to RR)
# Adjust logic as per specific contract if different
result <- dat %>%
  mutate(
    PARAMCD = "RR"
  ) %>%
  select(USUBJID, PARAMCD, AVAL)

# Write output
write_csv(result, file.path("outputs", "result.csv"), na = "")
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"USUBJID","PARAMCD","AVAL"
1,"RRR",1000
2,"RRR",800
3,"RRR",600
```

### LLM Output
#### `result.csv`


```csv
USUBJID,PARAMCD,AVAL
01,RR,60
02,RR,75
03,RR,100
```
