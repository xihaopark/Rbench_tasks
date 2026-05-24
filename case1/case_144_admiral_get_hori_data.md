# Case 144: pharmaverse/admiral/get_hori_data

## Metadata
- Task ID: `pharmaverse/admiral/get_hori_data`
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
Write an R script to perform get hori data using the admiral clinical task contract.

Input: dataset.tsv
Output: result.csv


Required columns for result.csv: USUBJID, ALT, AST, BILI
Read input files from inputs/ using relative paths. Write only the required output file(s) under outputs/. Create outputs/ if needed. Do not write alternative filenames.
```

## Input
#### `dataset.tsv`


```text
USUBJID	PARAMCD	AVAL
01	ALT	20
01	AST	18
01	BILI	0.7
02	ALT	40
02	AST	35
02	BILI	1.1
```

## Code
### Ground Truth Code

```r
dataset <- read.delim(file.path("inputs", "dataset.tsv"), check.names = FALSE, stringsAsFactors = FALSE)
subjects <- sort(unique(dataset$USUBJID))
params <- sort(unique(dataset$PARAMCD))
result_df <- data.frame(USUBJID = subjects, stringsAsFactors = FALSE)
for (param in params) {
  result_df[[param]] <- NA_real_
}
for (i in seq_len(nrow(dataset))) {
  row <- match(dataset$USUBJID[i], result_df$USUBJID)
  result_df[[dataset$PARAMCD[i]]][row] <- dataset$AVAL[i]
}

dir.create("outputs", showWarnings = FALSE)
write.csv(result_df, file.path("outputs", "result.csv"), row.names = FALSE)
```

### LLM Generated Code
```r
#!/usr/bin/env Rscript

library(dplyr)
library(readr)
library(tidyr)

# Create outputs directory if it does not exist
if (!dir.exists("outputs")) {
  dir.create("outputs", recursive = TRUE)
}

# Read input
infile <- file.path("inputs", "dataset.tsv")
dat <- read_tsv(infile, show_col_types = FALSE)

# Pivot to get horizontal data
res <- dat %>%
  mutate(PARAMCD = toupper(PARAMCD)) %>%
  filter(PARAMCD %in% c("ALT", "AST", "BILI")) %>%
  pivot_wider(
    id_cols = USUBJID,
    names_from = PARAMCD,
    values_from = AVAL
  ) %>%
  select(USUBJID, ALT, AST, BILI)

# Write output
outfile <- file.path("outputs", "result.csv")
write_csv(res, outfile)
```

## Output
### Ground Truth Output
#### `result.csv`


```csv
"USUBJID","ALT","AST","BILI"
1,20,18,0.7
2,40,35,1.1
```

### LLM Output
#### `result.csv`


```csv
USUBJID,ALT,AST,BILI
01,20,18,0.7
02,40,35,1.1
```
